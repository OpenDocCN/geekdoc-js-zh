# 第五章：解析器组合器

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/05-parser-combinators`](https://keleshev.com/compiling-to-assembly-from-scratch/05-parser-combinators)

从零开始编译汇编

由 Vladimir Keleshev 提供

解析器是一个函数，它将某些文本源转换为结构化数据。在我们的情况下，它将程序的源代码转换为相应的 AST。

有无数种解析技术和方法。选择哪一种通常取决于源语言及其句法特征。

我们将使用的技术称为**解析器组合器**。我们还将使用一些**正则表达式**，因为它们在 JavaScript 中非常方便。

解析器组合器的想法是创建少量**原始**解析器，这些解析器可以**组合**成更复杂的解析器。每个原始解析器都非常简单，单独使用时几乎没有任何有用的功能，但通过组合它们，我们可以解析越来越复杂的语言。

解析器组合器可以用来实现几乎任何解析算法。它们可以立即解析，或者生成表示语法的用于后续的数据结构。它们甚至可以用于代码生成。在实践中（以及历史上），许多解析器组合器是无扫描器（无标记）、**贪婪**、**回溯**和具有**优先选择**的。我们将在稍后解释这些术语的含义。我们的解析器组合器也不例外。

当我需要手动解析某些内容时，解析器组合器是我的首选技术：它们易于实现且功能强大。在实现几个原始组合器函数之后，你将得到一个类似于完整解析器生成器的功能。

## 词法分析、扫描或标记化

当谈论解析时，词法分析、扫描和标记化都是同义词。它们指的是将源程序转换为称为**标记流**的中间表示。标记流结构由我们程序的各个“单词”组成，也称为**标记**、**词素**或**词法元素**。这个术语起源于语言学，该领域是语法和解析的先驱。

标记流是一种线性表示，与 AST 的树状表示不同。

词法分析器会将像`x + y`这样的源代码转换为如下线性标记流：

```js
[
 new TokenId("x"),
 new TokenOperator("+"),
 new TokenId("y"),
]
```

只有在那时，解析器才会将其转换为像`new Addition(new Id("x"), new Id("y"))`这样的 AST。

我们使用数组来表示它，但更常见的是某种懒加载或流式集合。

这就是说，我们不会使用词法分析器：我们的解析器将是**无扫描器**的。无扫描器解析的思考方式是将每个字符视为一个标记。然而，当我们谈论解析单个逻辑词法元素时，我们仍然会使用“标记”这个词。我们也不会深入探讨词法分析或无扫描器解析的优缺点。在这里，我们选择无扫描器是因为其实施简单。

## 语法

当创建解析器时，我们希望讨论这些解析器可以识别的语言结构。为此，我们将使用*语法*。描述语法的符号有几种。你可能听说过*扩展巴科斯-诺尔范式*（EBNF）。在我们的情况下，我们将使用*解析表达式语法*（PEG）符号。PEG 符号非常适合我们，因为它是为*无扫描器*、*贪婪*、*回溯*解析器和表达*优先选择*而设计的。它借鉴了 EBNF 和正则表达式的符号。

当我们介绍每个解析器组合器时，我们也会展示相应的语法。你会发现我们的解析器组合器被设计成模仿语法符号。

## 接口

假设解析器是一个具有`parse`方法的对象，该方法接受称为`Source`的东西，并返回称为`ParseResult`的东西或`null`。

```js
interface Parser<T> {
 parse(s: Source): ParseResult<T> | null;
}
```

TypeScript 允许我们明确表示`parse`方法可能返回`null`，并且它还会在其*严格空检查*模式下强制执行此操作。

`Source`和`ParseResult`是什么？为什么我们不能使用`string`作为源？

在解析时，我们需要跟踪我们解析的字符串以及我们当前匹配的字符串位置。因此，`Source`是一个由以下组成的对：

+   我们解析的字符串，以及

+   当前解析的字符串中的索引，指向我们当前解析的位置。

```js
class Source {
 constructor(public string: string,
 public index: number) {}
 …
}
```

此外，`Source`允许我们避免与`string`类型紧密耦合。例如，我们可以在以后实现`Source`的另一个版本，该版本可以从文件中懒加载。

对于我们的用例，`Source`将非常高效，因为大多数`Source`对象在解析期间将共享相同的`string`。

关于`ParseResult`，它是一个简单的数据对象，可以保存解析器生成的某些值以及带有更新索引位置的源。`value`通常是我们案例中的`AST`。`source`表示解析器停止的位置，因此另一个解析器可以从那里继续。

```js
class ParseResult<T> {
 constructor(public value: T,
 public source: Source) {}
}
```

然而，解析器不仅可以返回`ParseResult`，还可以返回`null`。当返回`null`时，解析器表示它没有匹配任何内容。这不是错误：在单次遍历中，许多规则不会匹配，但其他规则会匹配。

目前，`Source`没有定义任何操作。有许多选择。其中之一是公开一个`match`方法，它与`string.match`非常相似，它接受一个正则表达式。与`string.match`的不同之处在于，我们需要从特定的`source.index`位置进行匹配。这可以通过所谓的“粘性”正则表达式实现。

在 JavaScript 中，粘性正则表达式使用标志“`y`”指定，例如：`/hello/y`。它们特别之处在于它们有一个`lastIndex`属性。通过将此属性设置为某个索引，我们可以控制正则表达式将在何处匹配。这是一个奇特的设计，但它对我们有效。其他编程语言有不同的方式从特定索引匹配正则表达式。

让我们编写我们的`match`方法：

```js
class Source {
 constructor(public string: string,
 public index: number) {}

 match(regexp: RegExp): (ParseResult<string> | null) {
 console.assert(regexp.sticky);
 regexp.lastIndex = this.index;
 let match = this.string.match(regexp);
 if (match) {
 let value = match[0];
 let newIndex = this.index + value.length;
 let source = new Source(this.string, newIndex);
 return new ParseResult(value, source);
 }
 return null;
 }
}
```

首先，我们断言我们得到的正则表达式是粘性的；否则，这不会起作用。然后我们将正则表达式上的`lastIndex`设置为从源索引开始匹配。然后我们委托给`string.match`方法进行匹配。如果匹配对象是`null`，我们返回`null`：我们无法匹配正则表达式。否则，我们得到了一个正则表达式“match”对象。它是一个类似数组的对象，其中第一个项目是与正则表达式匹配的子串。我们以两种方式使用那个子串：首先，它是我们使用的`ParseResult`的值。其次，我们计算匹配字符串的长度，以前进我们返回的新`Source`索引，作为`ParseResult`的第二个组件。

如您所见，`Source`几乎已经满足我们的`Parser`接口。

在 TypeScript 中实现满足`Parser`接口的解析器组合器的一种自然方式是为每个解析器创建一个类，并具有不同的`parser`方法。然而，由于`Parser`接口只有一个方法，并且我们的大多数解析器组合器将具有单行实现，让我们使用一种更轻量级的方法。相反，我们将有一个单一的`Parser`类，它接受`parse`方法（作为一个箭头函数）作为参数：

```js
class Parser<T> {
 constructor(
 public parse: (s: Source) => (ParseResult<T> | null)
 ) {}

 …
}
```

## 原始组合器

现在，让我们开始定义*原始*解析器组合器。其中一些将是`Parser`实例方法；一些将是静态方法。我们纯粹基于符号来选择一个或另一个：对于每个组合器，`f(x)`或`x.f()`哪个更易读。在做出这个选择时，我们将尝试模仿相应的语法。

## 正则表达式组合器

我们第一个名为`regexp`的组合器创建了一个匹配正则表达式的解析器。它只委托给刚刚定义的`source.match`方法。

```js
class Parser<T> {
 …

 static regexp(regexp: RegExp): Parser<string> {
 return new Parser(source => source.match(regexp));
 }
}
```

当使用这个组合器时，我们必须记住传递带有“`y`”标志的“粘性”正则表达式。否则，我们之前定义的断言会提醒我们。

```js
let hello = Parser.regexp(/hello[0-9]/y);
```

对应的 PEG 语法是：

```js
hello <- "hello" [0-9]
```

箭头定义了一个名为`hello`的语法规则。如您所见，PEG 符号借鉴了正则表达式符号。双引号中的字符串匹配字面意义，而`[0-9]`是一个字符类，这是一个从正则表达式借用的概念。在引号和字符类之外，空白字符不重要。

让我们尝试使用我们的`hello`解析器来解析一个字符串：

```js
let source = new Source("hello1 bye2", 0);
let result = Parser.regexp(/hello[0-9]/y).parse(source);

console.assert(result.value === "hello1");
console.assert(result.source.index === 6);
```

在这里，我们从一个字符串`"hello1 bye2"`创建一个新的源。我们希望它从开始解析，因此我们将源索引设置为零。我们用构造的源调用`parse`，并返回一个`ParseResult`对象。然后我们断言正在解析的值是`"hello1"`，并且结果源索引已经前进到第六列，那里是字符串的其余部分：“`bye2`”。

## 常量组合器

下一个组合器可能看起来有点愚蠢：它是一个总是成功的解析器，返回一个常量值，不消耗任何输入（因此，不前进源）。我们称它为 `constant`。它有什么好处呢？很快你就会看到。现在，让我们说它允许，当与其他解析器组合时，改变解析器的返回值（和类型）。

```js
class Parser<T> {
 …

 static constant<U>(value: U): Parser<U> {
 return new Parser(source =>
 new ParseResult(value, source));
 }
}
```

在 PEG 中，它对应于一个空字符串：

```js
empty <- ""
```

该表示法不关心产生什么值，只关心识别什么字符串。

## 错误组合器

接下来是 `error`：一个仅抛出异常的解析器。这个异常并不打算被处理。它预期将终止程序。

```js
class Parser<T> {
 …

 static error<U>(message: string): Parser<U> {
 return new Parser(source => {
 throw Error(message);
 });
 }
}
```

它与 PEG 表示法没有对应关系。

> **探索**
> 
> `error` 组合器的更好实现会检查源，将源索引转换为行-列对，并将其与有问题的行和一些上下文一起显示。

我们可以使用完全限定的名称使用这些组合器，例如 `Parser.regexp`，或者我们可以将它们“导入”到当前命名空间中：

```js
let {regexp, constant, error} = Parser;
let hello = regexp(/hello[0-9]/y);
```

从现在起，我们将假设我们定义的解析器组合器的名称已导入，我们不需要限定它们。

## 选择运算符

下一个原始解析器组合器是 *选择运算符*，我们将其定义为解析器的 `or` 实例方法。它允许我们在两个（或更多）替代解析器选择之间进行选择。与之前的原始组合器不同，这是一个实例方法，而不是静态方法。我们在这里使用实例方法纯粹是出于语法原因。我们希望写 `x.or(y).or(z)`，而不是 `or(or(x, y), z)`，因此它模仿相应的 PEG 语法：`x / y / z`。

类似于 `left.or(right)` 的解析器首先尝试使用 `left` 解析器进行解析。如果成功，则返回结果。如果不成功，则尝试 `right` 解析器，并返回其结果。

```js
class Parser<T> {
 …

 or(parser: Parser<T>): Parser<T> {
 return new Parser((source) => {
 let result = this.parse(source);
 if (result)
 return result;
 else
 return parser.parse(source);
 });
 }
}
```

这种选择运算符被称为 *优先选择* 运算符。与 *无序选择* 运算符相比，它从左到右按（优先级）顺序尝试替代解析器。

PEG 表示法通过使用正斜杠 (`/`) 作为优先选择运算符，而不是通常用水平线 (`|`) 表示的无序选择运算符，帮助我们精确地表达我们使用优先选择的事实。

无序选择也可以以不同的方式处理：解析器可以同时探索几个选择，或者如果通过查看一个（或 *n* 个字符）无法确定选择是否可解析，则可以提前直接拒绝该选择，在任何解析之前。

选择运算符是解析中的核心主题之一，其选择往往是解析器性能和识别能力的决定性因素。

给定一个如 `left.or(right)` 的解析器，我们称它为**回溯**，因为它将尝试解析器 `left`，如果它不成功，将**回溯**（或返回）到相同的源索引并尝试 `right` 解析器。我们说我们的解析器有**无限前瞻**，因为它将尝试完全解析 `left`，无论匹配有多长，换句话说，没有任何**限制**。其他技术前瞻一个（或 *n* 个字符，或标记）来决定选择哪个。

从理论上讲，我们实现选择运算符的方式不是很高效，可以通过缓存来改进。然而，如果我们小心地组合我们的解析器，它可能不是问题。关键是不要组合可以解析相同长前缀的解析器，而是组合替代解析器，这样它们可以快速识别它们不匹配并移动到下一个。

考虑由以下两个解析器构建的解析器：一个匹配连续一百个字符“a”后跟一个单独的“b”，另一个匹配连续一百个字符“a”后跟一个单独的“c”。给定一个匹配后者的输入，它将不得不扫描一百个字符**两次**：

```js
regexp(/a{100}b/y).or(regexp(/a{100}c/y))
```

另一方面，如果我们有一个解析字母的解析器，另一个解析数字的解析器，我们可以用 `or` 组合它们，并且只需要一个字符来检查第一个解析器是否匹配，然后再移动到下一个解析器。

```js
let letterOrDigit =
 regexp(/[a-z]/y).or(regexp(/[0-9]/y));
```

这个解析器可以用以下语法来描述：

```js
letterOrDigit <- [a-z] / [0-9]
```

## 重复：零次或多次

下一个原始解析器组合器是 `zeroOrMore`，它处理重复。给定一个返回某些结果的解析器，它将返回一个新的解析器，该解析器返回一个结果数组。其实现如下。

```js
class Parser<T> {
 …

 static zeroOrMore<U>(
 parser: Parser<U>,
 ): Parser<Array<U>> {
 return new Parser(source => {
 let results = [];
 let item = null;
 while (item = parser.parse(source)) {
 source = item.source;
 results.push(item.value);
 }
 return new ParseResult(results, source);
 });
 }
}
```

我们在 `while` 循环中多次应用相同的解析器。每次我们前进 `source` 并将结果值推入数组。一旦其中一个解析器不匹配并返回 `null`，我们就返回该数组作为新 `ParseResult` 对象的结果，该对象捆绑了最后的 `source`。

例如，这个解析器将匹配零个或多个字母或数字，给定我们刚刚定义的 `letterOrDigit`。

```js
let someLettersOrDigits = zeroOrMore(letterOrDigit);
```

这个组合函数对应于 PEG 中的星号 (`*`) 运算符，类似于正则表达式表示法。

```js
someLettersOrDigits -> letterOrDigit*
```

我们还可以内联 `letterOrDigit` 并将其重写为：

```js
someLettersOrDigits -> ([a-z] / [0-9])*
```

我们称我们的 `zeroOrMore` 或“星号”运算符为**贪婪**。它符合 PEG 语义，但与正则表达式不同。正则表达式如 `a*a` 将匹配一个或多个字母 `a`。然而，PEG（以及我们的解析器组合器）中的 `a*` 部分将永远不会匹配任何内容：表达式中的 `a*` 部分会“贪婪”地消费输入，只要它能匹配，而不考虑其后的内容。

## 绑定

下一个解析器组合器非常有趣。它是一个关键的组合器，与选择操作符相当。正如其名所暗示的，它*绑定*一个正在解析的值到一个名称。通过将值绑定到名称，我们可以操作和构建解析器生成的值。

```js
class Parser<T> {
 …

 bind<U>(
 callback: (value: T) => Parser<U>,
 ): Parser<U> {
 return new Parser((source) => {
 let result = this.parse(source);
 if (result) {
 let value = result.value;
 let source = result.source;
 return callback(value).parse(source);
 } else {
 return null;
 }
 });
 }
}
```

它接受一个回调，这通常将是一个箭头函数。回调函数将解析结果作为参数，并返回一个应该继续解析的新解析器。这样，我们可以组合几个解析器并将它们的值绑定以构建新的返回值。

例如，这里我们正在构建一个解析逗号分隔的数字对的解析器，例如`"12,34"`。

```js
let pair =
 regexp(/[0-9]+/y).bind((first) =>
 regexp(/,/y).bind((_) =>
 regexp(/[0-9]+/y).bind((second) =>
 constant([first, second]))));
```

我们将第一个数值解析器绑定到`first`参数。我们忽略逗号正则表达式的结果（通过将其绑定到下划线），然后将第二个数值正则表达式绑定到`second`参数。从这些中，我们构建一个数组，并通过构建一个常量解析器“返回”这个数组。

相应的语法是：

```js
pair <- [0-9]+ "," [0-9]+
```

像往常一样，语法忽略了解析器构造的值；它只关心它能识别的语言构造。

## 非原始解析器

在使用`bind`的第一个例子中，我们已经看到了在实际上使用`bind`时反复出现的两种模式。这就是为什么我们现在将它们作为非原始解析器引入。

## 并且和 map

第一个重复的模式是使用`bind`，但忽略值。它有效地创建了一系列解析器。我们称这个非原始组合器为`and`。它允许像`bind`一样按顺序排列解析器，但不需要将值绑定到名称：

```js
class Parser<T> {
 …

 and<U>(parser: Parser<U>): Parser<U> {
 return this.bind((_) => parser);
 }
}
```

我们接下来会看到很多的是只绑定名称以立即返回一个常量解析器的模式。我们称之为`map`。

```js
class Parser<T> {
 …

 map<U>(callback: (t: T) => U): Parser<U> {
 return this.bind((value) =>
 constant(callback(value)));
 }
}
```

值得注意的是，这个`map`方法在某种程度上类似于`array.map`方法。

* * *

现在，让我们尝试使用新方法重写我们的配对解析器：

```js
let pair =
 regexp(/[0-9]+/y).bind((first) =>
 regexp(/,/y).and(
 regexp(/[0-9]+/y)).map((second) =>
 [first, second]));
```

注意，`left.and(right)`中左解析器产生的值被忽略了。如果你不想忽略它，你需要`bind`它。

以这种方式编写代码并不像编写涉及 JavaScript promises 的代码。有时可能会很棘手，但幸运的是，给定一个语法，我们可以几乎机械地制作相应的解析器。

让我们从另一个角度来看它。假设我们心中有一个语法，比如我们的配对语法：

```js
pair <- [0-9]+ "," [0-9]+
```

假设我们想要生成一个解析器。我们使用`let`来定义命名规则，并使用`and`方法对所有序列（以及`or`方法对任何替代）进行操作：

```js
let pair =
 regexp(/[0-9]+/y).and(
 regexp(/,/y).and(
 regexp(/[0-9]+/y)));
```

现在，我们应该考虑我们想要提取哪些值。我们想要提取两个数值，所以我们将`and`替换为`bind`：

```js
let pair =
 regexp(/[0-9]+/y).bind((first) =>
 regexp(/,/y).and(
 regexp(/[0-9]+/y).bind((second) => …)));
```

在最后一个绑定中，我们使用`constant`解析器构建我们想要的值：

```js
let pair =
 regexp(/[0-9]+/y).bind((first) =>
 regexp(/,/y).and(
 regexp(/[0-9]+/y).bind((second) =>
 constant([first, second])));
```

我们完成了，但如果愿意，我们可以将第二个`bind`替换为`map`，因为它返回一个常量解析器：

```js
let pair =
 regexp(/[0-9]+/y).bind((first) =>
 regexp(/,/y).and(
 regexp(/[0-9]+/y).map((second) =>
 [first, second])));
```

## 也许

`maybe`组合器允许我们解析某些内容，或者可选地返回`null`。我们可以通过将`or`操作符与`constant(null)`解析器组合来实现这一点：

```js
class Parser<T> {
 …

 static maybe<U>(
 parser: Parser<U | null>,
 ): Parser<U | null> {
 return parser.or(constant(null));
 }
}
```

在这里，我们可选地解析一个字母或一个数字：

```js
let maybeLetterOrDigit = maybe(letterOrDigit);
```

这个组合器对应于 PEG 中的“`?`”运算符，也类似于正则表达式的表示法：

```js
maybeLetterOrDigit <- letterOrDigit?
```

就像在 PEG 中一样，与正则表达式不同，我们的 `maybe` 组合器是 *贪婪的*。

有时使用一个不同的默认值而不是 `null`，例如一个空数组，是非常有用的。在这些情况下，我们可以说 `parser.or(constant([]))`。

## 解析字符串

虽然每个解析器都有一个 `parse` 方法，但这个方法在组合解析器时比在实际使用中更方便。所以让我们定义一个稍微不同的辅助方法，称为 `parseStringToCompletion`。

```js
class Parser<T> {
 …

 parseStringToCompletion(string: string): T {
 let source = new Source(string, 0);

 let result = this.parse(source);
 if (!result)
 throw Error("Parse error at index 0");

 let index = result.source.index;
 if (index != result.source.string.length)
 throw Error("Parse error at index " + index);

 return result.value;
 }
}
```

正如其名所示，这个方法不是接受一个 `Source` 对象，而是接受一个字符串。这使得测试和使用生成的解析器更加方便。这个方法会为你创建一个 `Source` 对象并调用 `parse` 方法。然而，与 `parse` 方法不同，如果结果无法解析，或者解析没有消耗掉整个输入字符串，它会抛出一个异常。它还会解包 `ParseResult`，只给我们结果值。

* * *

现在我们已经学会了如何构建解析器，让我们让解析器通过我们的基线编译器！

下一章：第六章. 第一次遍历：解析器

* * *
