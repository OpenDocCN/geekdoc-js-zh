# 第六章：第一次遍历：解析器

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/06-first-pass-the-parser`](https://keleshev.com/compiling-to-assembly-from-scratch/06-first-pass-the-parser)

从头开始编译汇编

由 Vladimir Keleshev 提供

现在我们已经拥有了足够的解析机制，我们可以实现从源代码生成抽象语法树（AST）的解析器。

## 空白和注释

首先，让我们定义空白和注释的解析器，我们将它们统称为 *ignored*。（当一个解析器被分割成词法分析和解析器时，空白和注释通常在词法分析级别完成。）

```js
let whitespace = regexp(/[ \n\r\t]+/y);
let comments =
 regexp(/[/][/].*/y).or(regexp(/[/][*].*[*][/]/sy));
let ignored = zeroOrMore(whitespace.or(comments));
```

我们允许单行注释（`// ...`）和多行注释（`/* ... */`）。在 JavaScript 正则表达式中，点号匹配任何字符，*除了换行符*。为了实现多行注释，我们需要匹配换行符。通过传递“dot-all”标志“`s`”和“sticky”标志“`y`”，我们可以改变点号正则表达式的含义，使其匹配任何字符*包括换行符*：

```js
/[/][*].*[*][/]/sy
```

我们使用单个字符如 `[*]` 的字符类作为在正则表达式中有特殊意义的字符的可读转义方式。否则，它们很快就会看起来像一把损坏的锯子：

```js
/\/\*.*\*\//sy
```

## 标记

尽管我们的解析器是无扫描器（或无标记），但仍然有用，因为它可以帮助我们区分标记作为我们的句法构建块。想法是从简单的字符解析器构建标记类似的解析器，然后仅在此基础上构建完整的解析器。

按照惯例，我们将使用标记的命名约定（在语法规则和解析器中）：它们将是大写。

标记为我们提供了比单个字符更高层次的源代码视图。它们允许我们不必处理像空白和注释这样的细节。这正是我们将如何使用它们的。我们将定义一个 `token` 解析器组合器（或更精确地说，构造器），它允许我们忽略词法单元（或标记）周围的空白和注释。

```js
let token = (pattern) =>
 regexp(pattern).bind((value) =>
 ignored.and(constant(value)));
```

它接受一个正则表达式模式，并返回一个解析器，该解析器是那个模式的序列和一个我们之前定义的 `ignored` 规则，用于处理空白和注释。它产生与模式匹配的字符串值，但忽略 `ignored` 规则的值。我们只在右侧使用 `ignored` 进行“填充”，而不是在模式周围。在除了第一个标记之外的所有情况下，这将是多余的，我们可以特别处理它。这是在无扫描器解析器中处理空白和注释的常见方式。

我们将从定义我们语言中的关键字标记开始，如 `function`、`if`、`else`、`return`、`var` 和 `while`：

```js
let FUNCTION = token(/function\b/y);
let IF = token(/if\b/y);
let ELSE = token(/else\b/y);
let RETURN = token(/return\b/y);
let VAR = token(/var\b/y);
let WHILE = token(/while\b/y);
```

我们使用了一个单词分隔转义序列 `\b` 来确保我们不将 `functional` 识别为关键字 `function` 后跟标识符 `al`。

然后，定义标点符号的标记：逗号、括号等：

```js
let COMMA = token(/[,]/y);
let SEMICOLON = token(/;/y);
let LEFT_PAREN = token(/[(]/y);
let RIGHT_PAREN = token(/[)]/y);
let LEFT_BRACE = token(/[{]/y);
let RIGHT_BRACE = token(/[}]/y);
```

我们的基本编译器将只处理十进制整数数字（基数 10）：

```js
let NUMBER =
 token(/[0-9]+/y).map((digits) =>
 new Number(parseInt(digits, 10)));
```

我们将`[0-9]+`标记映射到使用`parseInt` JavaScript 内置函数将数字转换为 JavaScript 数字，该函数接受一个字符串和一个基数（十进制为 10）。然后我们从这个字符串构造一个`Number` AST 节点。

标识符以字母或下划线开头，后跟一定数量的字母、数字和下划线。换句话说，它们不能以数字开头。

```js
let ID = token(/[a-zA-Z_][a-zA-Z0-9_]*/y);
```

在某些情况下，仅为了解析标识符的字符串值，对我们来说是有用的，但有时更方便的是生成 AST 节点。这就是为什么我们创建了一个别名`id`，它与`ID`相同，但不是产生一个字符串，而是产生一个 AST 节点`Id`。

```js
let id = ID.map((x) => new Id(x));
```

现在，操作符标记：

```js
let NOT = token(/!/y).map((_) => Not);
let EQUAL = token(/==/y).map((_) => Equal);
let NOT_EQUAL = token(/!=/y).map((_) => NotEqual);
let PLUS = token(/[+]/y).map((_) => Add);
let MINUS = token(/[-]/y).map((_) => Subtract);
let STAR = token(/[*]/y).map((_) => Multiply);
let SLASH = token(/[/]/y).map((_) => Divide);
let ASSIGN = token(/=/y).map((_) => Assign);
```

我们不能仅从单个运算符构造一个完整的 AST 节点，但返回 AST 节点的类别将会很方便。

## 语法

我们的基础编译器的语法分为两部分：*表达式*和*语句*。JavaScript（以及 TypeScript）对区分这两者相对宽松。一般来说，表达式是产生值的构造，例如`1 + 2`，而语句是具有影响但不会产生值的操作，例如`while`循环。

## 表达式解析器

我们将首先构建一个用于单个表达式的解析器。以下是基线语言中表达式的语法。

```js
args <- (expression (COMMA expression)*)?
call <- ID LEFT_PAREN args RIGHT_PAREN
atom <- call / ID / NUMBER
      / LEFT_PAREN expression RIGHT_PAREN
unary <- NOT? atom
product <- unary ((STAR / SLASH) unary)*
sum <- product ((PLUS / MINUS) product)*
comparison <- sum ((EQUAL / NOT_EQUAL) sum)*
expression <- comparison
```

它包括如`==`、`!=`、`+`、`-`、`*`、`/`、一元否定`!x`、标识符、整数数字和函数调用等中缀操作。

表达式规则被分成几个层次，以便处理运算符优先级。`expression`本身只是一个`comparison`的别名。`comparison`规则表示具有最低优先级的运算符：`==`和`!=`。`comparison`本身是由具有更高优先级的`sum`构建的，其中运算符有`+`和`-`。`sum`反过来又是由具有最高优先级的`product`规则构建的，这些规则是中缀运算符中的最高优先级：`*`和`/`。这些又是由表示一元运算符的`unary`构建的，其中我们只有否定：`!`。否定在我们的所有运算符中具有最高的优先级。它是建立在称为`atom`的规则之上的。`atom`规则表示那些“原子”的规则，或者由于它们的结构不需要优先级。例如，`ID`和`NUMBER`是单个词素，所以不适用优先级，而`call`和括号表达式有明确的分隔符，所以也不适用优先级。`args`规则被提取出来以简化`call`规则。

这种通过增加优先级来构建解析器的过程有时被比作给绳子添加珠子。

你可能注意到，最低级别的规则，如`atom`和`call`（通过`args`）都回指`expression`。这意味着我们的语法是**递归的**。这为构建这个语法的解析器创造了一个小问题。虽然 JavaScript 允许递归函数，但不允许递归值。其他语言允许所谓的`let-rec`绑定，但 JavaScript 不允许。我们处理这个问题的方法是首先将`expression`定义为`error`解析器：

```js
let expression: Parser<AST> =
 Parser.error(
 "Expression parser used before definition");
```

现在，我们可以在构建我们的解析器时自由使用它。然而，我们必须记住在定义`comparison`之后和在使用它之前，就地更改这个解析器：

```js
expression.parse = comparison.parse;
```

## 调用解析器

我们需要通过首先实现`args`来实现`call`。

```js
args <- (expression (COMMA expression)*)?
call <- ID LEFT_PAREN args RIGHT_PAREN
```

将`args`机械地转换为解析器，我们得到：

```js
// args <- (expression (COMMA expression)*)?
let args =
 maybe(
 expression.and(zeroOrMore(COMMA.and(expression))))
```

就像在`Call` AST 节点的情况一样，我们将其称为`args`而不是`arguments`，因为`arguments`是一个特殊的 JavaScript 对象，我们不希望与之冲突。

我们希望这个解析器返回一些有用的东西，比如 AST 节点的数组。`maybe`组合器在不匹配时返回`null`，但我们需要一个空数组，所以让我们用`.or(constant([]))`来替换它。我们需要绑定第一个表达式，然后绑定其余的表达式，然后将它们连接起来。通过这样做，我们最终得到以下内容：

```js
// args <- (expression (COMMA expression)*)?
let args: Parser<Array<AST>> =
 expression.bind((arg) =>
 zeroOrMore(COMMA.and(expression)).bind((args) =>
 constant([arg, ...args]))).or(constant([]))
```

表达式`zeroOrMore(COMMA.and(expression))`方便地忽略了逗号，并产生一个 AST 节点的数组。

现在，从语法中机械地实现`call`，我们得到：

```js
// call <- ID LEFT_PAREN args RIGHT_PAREN
let call =
 ID.and(LEFT_PAREN.and(args.and(RIGHT_PAREN)))
```

我们将`ID`（表示被调用者的名称）和`args`绑定来构建一个`Call`节点：

```js
// call <- ID LEFT_PAREN args RIGHT_PAREN
let call: Parser<AST> =
 ID.bind((callee) =>
 LEFT_PAREN.and(args.bind((args) =>
 RIGHT_PAREN.and(
 constant(new Call(callee, args))))));
```

## 原子

下一个解析器是`atom`。为了确保它产生 AST，我们需要使用`id`规则而不是`ID`。我们还绑定`expression`以提取其值。

```js
// atom <- call / ID / NUMBER
//       / LEFT_PAREN expression RIGHT_PAREN
let atom: Parser<AST> =
 call.or(id).or(NUMBER).or(
 LEFT_PAREN.and(expression).bind((e) =>
 RIGHT_PAREN.and(constant(e))));
```

选择项的顺序很重要。如果规则从`ID / call`开始，那么`call`将永远不会匹配，因为`call`本身以`ID`开头。

## 一元运算符

对于一元解析器，我们绑定`NOT?`和`atom`。如果存在`NOT`运算符，我们构建`Not` AST 节点，否则，我们产生底层节点。

```js
// unary <- NOT? atom
let unary: Parser<AST> =
 maybe(NOT).bind((not) =>
 atom.map((term) => not ? new Not(term) : term));
```

我们已经将`atom`的值绑定到名称`term`上，当绑定表达式或语句时，我们将使用它作为通用的短语。

## 中缀运算符

中缀运算符都是类似地构建的，低优先级规则建立在高优先级规则之上。

```js
product <- unary ((STAR / SLASH) unary)*
sum <- product ((PLUS / MINUS) product)*
comparison <- sum ((EQUAL / NOT_EQUAL) sum)*
```

你可能记得我们确保了运算符规则解析器产生相应的 AST 节点的类。例如，`EQUAL`标记产生`Equal`类。我们将使用这个属性。

我们将首先从`product`解析器开始。直观的机械翻译给我们：

```js
// product <- unary ((STAR / SLASH) unary)*
let product =
 unary.and(zeroOrMore(STAR.or(SLASH).and(unary)));
```

绑定第一个`一元`操作符很简单。然而，从所有这些中构建嵌套的 AST 结构并不容易。

我们可以将`(STAR / SLASH) unary`映射到一个中间数据结构`{operator, term}`，这是一个由`STAR / SLASH`和`unary`值构成的配对。这样，我们得到一个操作符-术语对的数组。我们绑定`unary`并调用它的值`first`。

例如，如果我们正在解析字符串`"x * y / z"`，那么`first`将是`new Id("x")`，而`operatorTerms`将是一个如下的数组：

```js
[
 {operator: Multiply, term: new Id("y")},
 {operator: Divide,   term: new Id("z")},
]
```

我们如何将这些减少为像以下这样的 AST 节点？

```js
new Divide(
 new Multiply(new Id("x"), new Id("y")),
 new Id("z"))
```

结果表明`array.reduce`正是可以完成这个任务的方法！以下是相应的代码：

```js
// product <- unary ((STAR / SLASH) unary)*
let product =
 unary.bind((first) =>
 zeroOrMore(STAR.or(SLASH).bind((operator) =>
 unary.bind((term) =>
 constant({operator, term})))).map((operatorTerms) =>
 operatorTerms.reduce((left, {operator, term}) =>
 new operator(left, term), first)));
```

真的很复杂！幸运的是，这是我们将会遇到的最复杂的规则。更好的是，我们可以将这个重复的模式提取到另一个解析器组合器中，并再次使用它。我们将这个组合器称为`infix`：

```js
let infix = (operatorParser, termParser) =>
 termParser.bind((term) =>
 zeroOrMore(operatorParser.bind((operator) =>
 termParser.bind((term) =>
 constant({operator, term})))).map((operatorTerms) =>
 operatorTerms.reduce((left, {operator, term}) =>
 new operator(left, term), term)));
```

我们将`product`规则改为一个接受两个参数的函数：`operatorParser`和`termParser`。我们将`unary`替换为`termParser`，将`STAR.or(SLASH)`替换为`operatorParser`。现在我们有一个可重用的组合器，我们可以不仅用于`product`，还可以用于`sum`和`comparison`。

```js
// product <- unary ((STAR / SLASH) unary)*
let product = infix(STAR.or(SLASH), unary);

// sum <- product ((PLUS / MINUS) product)*
let sum = infix(PLUS.or(MINUS), product);

// comparison <- sum ((EQUAL / NOT_EQUAL) sum)*
let comparison = infix(EQUAL.or(NOT_EQUAL), sum);
```

## 结合性

当我们解析`"x * y / z"`时，我们希望它被解释为`"(x * y) / z"`并产生相应的 AST：

```js
new Divide(
 new Multiply(new Id("x"), new Id("y")),
 new Id("z"))
```

因此，我们说这些运算符是左结合的。目前我们没有任何右结合的运算符，但如果有，我们将使用相同的语法规则，但我们必须使用`array.reduceBack`而不是`array.reduce`来构建 AST，并进行一些调整。

## 关闭循环：表达式

现在我们已经定义了`comparison`，我们可以最终定义`expression`，它迄今为止只有一个伪实现。我们通过覆盖`expression`的`parse`方法来实现这一点：

```js
// expression <- comparison
expression.parse = comparison.parse;
```

## 语句

接下来是解析语句。这是语法：

```js
returnStatement <- RETURN expression SEMICOLON

expressionStatement <- expression SEMICOLON

ifStatement <-
  IF LEFT_PAREN expression RIGHT_PAREN
    statement
  ELSE
    statement

whileStatement <-
  WHILE LEFT_PAREN expression RIGHT_PAREN statement

varStatement <- VAR ID ASSIGN expression SEMICOLON

assignmentStatement <- ID ASSIGN EXPRESSION SEMICOLON

blockStatement <- LEFT_BRACE statement* RIGHT_BRACE

parameters <- (ID (COMMA ID)*)?

functionStatement <-
  FUNCTION ID LEFT_PAREN parameters RIGHT_PAREN
  blockStatement

statement <- returnStatement
           / ifStatement
           / whileStatement
           / varStatement
           / assignmentStatemnt
           / blockStatement
           / functionStatement
           / expressionStatement
```

高级结构与表达式的情况非常相似。它由几个规则组成，其中`statement`是递归定义的。再次，我们从伪实现开始：

```js
let statement: Parser<AST> =
 Parser.error(
 "Statement parser used before definition");
```

第一种语句类型是`returnStatement`，它产生`Return` AST 节点：

```js
// returnStatement <- RETURN expression SEMICOLON
let returnStatement: Parser<AST> =
 RETURN.and(expression).bind((term) =>
 SEMICOLON.and(constant(new Return(term))));
```

`expressionStatement`是一个用分号分隔的表达式：

```js
// expressionStatement <- expression SEMICOLON
let expressionStatement: Parser<AST> =
 expression.bind((term) =>
 SEMICOLON.and(constant(term)));
```

`ifStatement`产生`If`节点：

```js
// ifStatement <-
//   IF LEFT_PAREN expression RIGHT_PAREN
//     statement
//   ELSE
//     statement
let ifStatement: Parser<AST> =
 IF.and(LEFT_PAREN).and(expression).bind(
 (conditional) =>
 RIGHT_PAREN.and(statement).bind((consequence) =>
 ELSE.and(statement).bind((alternative) =>
 constant(new If(conditional,
 consequence,
 alternative)))));
```

`whileStatement`在语法上与`ifStatement`相似：

```js
// whileStatement <-
//   WHILE LEFT_PAREN expression RIGHT_PAREN statement
let whileStatement: Parser<AST> =
 WHILE.and(LEFT_PAREN).and(expression).bind(
 (conditional) =>
 RIGHT_PAREN.and(statement).bind((body) =>
 constant(new While(conditional, body))));
```

`varStatement`和`assignmentStatement`也非常相似：

```js
// varStatement <-
//   VAR ID ASSIGN expression SEMICOLON
let varStatement: Parser<AST> =
 VAR.and(ID).bind((name) =>
 ASSIGN.and(expression).bind((value) =>
 SEMICOLON.and(constant(new Var(name, value)))));
```

```js
// assignmentStatement <- ID ASSIGN EXPRESSION SEMICOLON
let assignmentStatement: Parser<AST> =
 ID.bind((name) =>
 ASSIGN.and(expression).bind((value) =>
 SEMICOLON.and(constant(new Assign(name, value)))));
```

从技术上来说，在 JavaScript 中，赋值是一个表达式，但为了简单起见，我们将其定义为语句。这也是它被大多数时候使用的方式。

块语句是一系列用大括号分隔的语句。它产生一个`Block`节点：

```js
// blockStatement <- LEFT_BRACE statement* RIGHT_BRACE
let blockStatement: Parser<AST> =
 LEFT_BRACE.and(zeroOrMore(statement)).bind(
 (statements) =>
 RIGHT_BRACE.and(constant(new Block(statements))));
```

函数参数规则与我们之前定义的`args`规则非常相似（就解析器结构而言），但解析器产生的是`Array<string>`而不是`Array<AST>`：

```js
// parameters <- (ID (COMMA ID)*)?
let parameters: Parser<Array<string>> =
 ID.bind((param) =>
 zeroOrMore(COMMA.and(ID)).bind((params) =>
 constant([param, ...params]))).or(constant([]))
```

函数定义（我们也将它称为语句）基于`parameters`和`blockStatement`来产生一个`Function`节点：

```js
// functionStatement <-
//   FUNCTION ID LEFT_PAREN parameters RIGHT_PAREN
//   blockStatement
let functionStatement: Parser<AST> =
 FUNCTION.and(ID).bind((name) =>
 LEFT_PAREN.and(parameters).bind((parameters) =>
 RIGHT_PAREN.and(blockStatement).bind((block) =>
 constant(
 new Function(name, parameters, block)))));
```

我们将`statement`定义为上述语句之一，并通过修改原始的`statement.parser`来解开递归的结：

```js
// statement <- returnStatement
//            / ifStatement
//            / whileStatement
//            / varStatement
//            / assignmentStatement
//            / blockStatement
//            / functionStatement
//            / expressionStatement
```

```js
let statementParser: Parser<AST> =
 returnStatement
 .or(functionStatement)
 .or(ifStatement)
 .or(whileStatement)
 .or(varStatement)
 .or(assignmentStatement)
 .or(blockStatement)
 .or(expressionStatement);

statement.parse = statementParser.parse;
```

由于我们的语言应该接受多个语句，我们需要一个最后的修饰来完成我们的解析器：

```js
let parser: Parser<AST> =
 ignored.and(zeroOrMore(statement)).map((statements) =>
 new Block(statements));
```

它从允许“忽略”的空白和注释（在第一个逻辑标记之前需要显式忽略）开始，然后是零个或多个我们将其转换为`Block`节点的语句。

## 测试

在每一步测试解析器是至关重要的。本章*抽象语法树*中的示例提供了一个良好的单元测试来源。以下是一个可能的集成测试示例：

```js
let source = `
 function factorial(n) {
 var result = 1;
 while (n != 1) {
 result = result * n;
 n = n - 1;
 }
 return result;
 }
`;
```

```js
let expected = new Block([
 new Function("factorial", ["n"], new Block([
 new Var("result", new Number(1)),
 new While(new NotEqual(new Id("n"),
 new Number(1)), new Block([
 new Assign("result", new Multiply(new Id("result"),
 new Id("n"))),
 new Assign("n", new Subtract(new Id("n"),
 new Number(1))),
 ])),
 new Return(new Id("result")),
 ])),
]);

let result = parser.parseStringToCompletion(source);

console.assert(result.equals(expected));
```

* * *

解析器现在已经完成，我们的编译器的第一遍也完成了。

下一章：第七章\. ARM 汇编编程

* * *
