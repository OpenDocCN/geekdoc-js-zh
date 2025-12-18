# 第十三章：静态类型检查和推断

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/13-static-type-checking-and-inference`](https://keleshev.com/compiling-to-assembly-from-scratch/13-static-type-checking-and-inference)

从头开始编译汇编

by 弗拉基米尔·凯列舍夫

在本章中，我们将为我们的语言实现一个带有 *局部类型推断* 的类型检查器。局部类型推断意味着局部变量不需要类型注解，因此你可以编写如下代码：

```js
let a = [1, 2, 3];
```

`a` 的类型将被推断为 `Array<number>`。

我们的类型检查器将涵盖 `number`、`boolean`、`void` 和 `Array<T>` 等类型，其中 `T` 可以是任何其他类型，例如 `Array<Array<boolean>>`。在类型检查函数调用时，我们还将处理函数类型，如 `(x: boolean, y: number) => number`。

首先，我们需要引用不同的类型来操作它们。我们将以类似 AST 节点的方式表示它们，使用一个名为 `Type` 的接口，并为每种类型实现一个类。

```js
interface Type {
 equals(other: Type): boolean;
 toString(): string;
}
```

我们将需要相等性和一个 `toString` 方法来能够显示类型错误。

类型构造函数签名摘要及示例

| 类型构造函数签名 | 示例 |
| --- | --- |



```js
BooleanType()
```



```js
boolean
```



|  |  |
| --- | --- |



```js
NumberType()
```



```js
number
```



|  |  |
| --- | --- |



```js
VoidType()
```



```js
void
```



|  |  |
| --- | --- |



```js
ArrayType(element: Type)
```



```js
Array<number>
```



|  |  |
| --- | --- |



```js
FunctionType(
  parameters: Map<string, Type>,
  returnType: Type,
)
```



```js
(x: boolean,
 y: number)
  => number
```



为这些数据类实现 `toString` 和 `equals` 是直接的。然而，`FunctionType` 需要一些注释：

+   首先，JavaScript 中的 `Map` 保留了元素的顺序，这对于在调用点匹配参数位置非常重要。

+   当比较两个 `FunctionType` 的实例时，忽略参数名称是有意义的，因为 `(x: boolean, y: number) => number` 与 `(p1: boolean, p2: number) => number` 是相同的类型。

我们的语言将改变以允许函数进行类型注解。我们首先将我们的 `Function` AST 节点更改为存储其签名作为 `FunctionType`，而不是仅参数名称：

```js
class Function implements AST {
 constructor(public name: string,
 public signature: FunctionType,
 public body: AST) {}

 visit<T>(v: Visitor<T>) {…}

 equals(other: AST): boolean {…}
}
```

我们还需要更改我们的语法和解析器。在引入必要的标记后，我们可以为类型定义以下语法。

```js
arrayType <- ARRAY LESS_THAN type GREATER_THAN
type <- VOID | BOOLEAN | NUMBER | arrayType
```

`type` 规则是递归的，就像 `expression` 和 `statement` 一样。

接下来，我们需要更改函数的语法和解析器，以包含可选的类型注解：

```js
optionalTypeAnnotation <- (COLON type)?
parameter <- ID optionalTypeAnnotation
parameters <- (parameter (COMMA parameter)*)?
functionStatement <-
  FUNCTION ID LEFT_PAREN
    parameters
  RIGHT_PAREN optionalTypeAnnotation
    blockStatement
```

我们允许函数参数和函数返回类型具有可选的类型注解。为什么是可选的？一些语言可以推断参数和返回类型，但我们这样做是为了以后可以不修改地使用相同的解析器与动态类型一起使用。如果缺少类型注解，我们将其默认为 `number`，这也为我们与测试套件的向后兼容性提供了一些支持。

现在，解析器将接受具有类型注解的函数，如下所示：

```js
function pair(x: number, y: number): Array<number> {
 return [x, y];
}
```

对于此类函数，解析器将生成一个具有 `FunctionType` 签名的 `Function` 节点，如下所示：

```js
new Function(
 "pair",
 new FunctionType(
 new Map([["x", new NumberType()],
 ["y", new NumberType()]]),
 new ArrayType(new NumberType()),
 ),
 new Block([
 new Return(new ArrayLiteral([new Id("x"),
 new Id("y")])),
 ]),
)
```

接下来，对于类型检查，我们将使用一个函数来断言两个类型相同，否则抛出异常以指示错误。我们称之为 `assertType`：

```js
function assertType(expected: Type, got: Type): void {
 if (!expected.equals(got)) {
 throw TypeError(
 `Expected ${expected}, but got ${got}`);
 }
}
```

它使用`equals`方法和`toString`方法，后者在模板字符串变量上隐式调用。在这里，我们稍微滥用内置的`TypeError`异常，它有不同的用途（运行时类型错误）；最好定义一个自定义的异常类型。

我们的`TypeChecker`遍历 AST，要么通过抛出`TypeError`来终止，要么返回表达式的推断`Type`。因此，`TypeChecker`实现了`Visitor<Type>`。

```js
class TypeChecker implements Visitor<Type> {
 constructor(
 public locals: Map<string, Type>,
 public functions: Map<string, FunctionType>,
 public currentFunctionReturnType: Type | null,
 ) {}

 …
}
```

它维护两个环境：

+   `locals`——一个存储函数中局部变量类型的环境，

+   `functions`——一个存储每个函数签名的环境。

由于在我们的语言中函数不是一等公民：它们不能被赋值给变量并传递，因此我们需要为这些变量提供两个独立的环境。所以，当我们遇到一个`Call`时，我们将在`functions`映射中查找函数，当我们遇到一个`Id`时，我们将在`locals`映射中查找非函数类型。

我们还维护一个实例变量`currentFunctionReturnType`，它将帮助我们检查`Return`语句的类型。

现在，我们进入实际的类型检查和推断。

## 标量

文字标量的类型最容易推断。我们知道数字字面量的类型是`number`，布尔节点类型是`boolean`等等。

```js
 visitNumber(node: Number) {
 return new NumberType();
 }

 visitBoolean(node: Boolean) {
 return new BooleanType();
 }
```

在 TypeScript 中，表达式或语句返回`undefined`时，会推断出`void`类型。

```js
 visitUndefined(node: Undefined) {
 return new VoidType();
 }
```

## 运算符

现在，让我们看看最直接的运算符——否定。在 TypeScript 中，这并不完全是这样工作的，但让我们假设否定运算符期望严格的布尔参数。为了强制执行这一点，我们首先通过在它上调用`visit`方法来推断内部项的类型。然后我们断言它是布尔类型：

```js
 visitNot(node: Not) {
 assertType(new BooleanType(), node.term.visit(this));
 return new BooleanType();
 }
```

我们返回`boolean`类型，因为否定操作的结果总是这个类型。

为了检查数值运算符如`Add`的类型，我们推断`left`和`right`参数的类型，并断言它们都是数字。然后我们返回`number`作为结果类型。

```js
 visitAdd(node: Add) {
 assertType(new NumberType(), node.left.visit(this));
 assertType(new NumberType(), node.right.visit(this));
 return new NumberType();
 }
```

现在我们来探讨一些更有趣的内容。例如，等式这样的操作是通用的：只要左侧的类型与右侧的类型相同，它们就可以与任何类型一起工作。我们通过访问它们来推断两边的类型，然后断言它们是相同的。

```js
 visitEqual(node: Equal) {
 let leftType = node.left.visit(this);
 let rightType = node.right.visit(this);
 assertType(leftType, rightType);
 return new BooleanType();
 }
```

## 变量

对于使用`var`定义的新变量，我们通过访问其值来推断其类型，然后将该类型添加到`locals`环境中。

```js
 visitVar(node: Var) {
 let type = node.value.visit(this);
 this.locals.set(node.name, type);
 return new VoidType();
 }
```

当我们遇到一个变量时，我们将在`locals`环境中查找它并返回其类型。如果变量未定义，我们将抛出一个`TypeError`。

```js
 visitId(node: Id) {
 let type = this.locals.get(node.value);
 if (!type) {
 throw TypeError(`Undefined variable ${node.value}`);
 }
 return type;
 }
```

当为一个变量赋新值时，我们检查该变量之前是否已定义，并且赋值操作不会改变变量的类型。

```js
 visitAssign(node: Assign) {
 let variableType = this.locals.get(node.name);
 if (!variableType) {
 throw TypeError(`Undefined variable ${node.name}`);
 }
 let valueType = node.value.visit(this);
 assertType(variableType, valueType);
 return new VoidType();
 }
```

## 数组

推断数组字面量的类型比我们之前覆盖的其他字面量要复杂一些。我们知道任何数组字面量都是`Array<T>`类型，但我们需要找出`T`是什么。首先，我们无法推断空数组的类型——在这个阶段信息不足。有一些双向类型推断算法可以处理这个问题，但通常这是通过要求类型注释来解决的。

如果数组非空，我们会推断每个元素的类型，然后断言它们是相同类型的成对存在。

```js
 visitArrayLiteral(node: ArrayLiteral): Type {
 if (node.args.length == 0) {
 throw TypeError("Can't infer type of empty array");
 }
 let argsTypes =
 node.args.map((arg) => arg.visit(this));
 let elementType = argsTypes.reduce((prev, next) => {
 assertType(prev, next);
 return prev;
 });
 return new ArrayType(elementType);
 }
```

对于像我们的数组长度原语这样的东西，我们需要断言参数类型是数组，但我们不关心元素类型。因此，我们不是使用`assertType`，而是手动检查类型是否是`ArrayType`的实例，如果不是则抛出`TypeError`。这种表达式的推断类型是`number`。

```js
 visitLength(node: Length) {
 let type = node.array.visit(this);
 if (type instanceof ArrayType) {
 return new NumberType();
 } else {
 throw TypeError(`Expected an array, got: ${type}`);
 }
 }
```

当处理数组查找时，我们断言`index`是一个数字，并且`array`是数组类型：

```js
 visitArrayLookup(node: ArrayLookup): Type {
 assertType(new NumberType(), node.index.visit(this));
 let type = node.array.visit(this);
 if (type instanceof ArrayType) {
 return type.element;
 } else {
 throw TypeError(`Expected an array, got: ${type}`);
 }
 }
```

## 函数

当遇到一个函数时，我们将它的签名添加到`functions`环境中。我们不需要推断它，因为它是从源代码中解析出来的。在类型检查函数体之前，我们创建一个新的访问者，并修改环境。由于我们使用可变映射，我们需要为每个函数传递一个新的`Map`，以避免修改错误函数的环境。我们还设置了`currentFunctionReturnType`为签名中的类型。

```js
 visitFunction(node: Function) {
 this.functions.set(node.name, node.signature);
 let visitor = new TypeChecker(
 new Map(node.signature.parameters),
 this.functions,
 node.signature.returnType,
 );
 node.body.visit(visitor);
 return new VoidType();
 }
```

当访问`Call`时，我们从`functions`环境中获取该函数的签名。然后我们推断被调用的函数的类型，并将其与环境中的类型进行比较。在推断函数类型时，我们访问每个参数，由于`FunctionType`要求每个参数都有一个名称，我们给它们分配了虚拟名称，例如`x0`、`x1`、`x2`等。当涉及到函数的返回类型时，我们使用类型注释中的类型。

```js
 visitCall(node: Call) {
 let expected = this.functions.get(node.callee);
 if (!expected) {
 throw TypeError(`Function ${node.callee} undefined`);
 }
 let argsTypes = new Map();
 node.args.forEach((arg, i) =>
 argsTypes.set(`x${i}`, arg.visit(this))
 );
 let got =
 new FunctionType(argsTypes, expected.returnType);
 assertType(expected, got);
 return expected.returnType;
 }
```

当检查返回语句时，我们推断返回值的类型，并将其与函数注释中的类型进行比较，我们方便地将这个类型保存在`currentFunctionReturnType`实例变量中。

```js
 visitReturn(node: Return) {
 let type = node.term.visit(this);
 if (this.currentFunctionReturnType) {
 assertType(this.currentFunctionReturnType, type);
 return new VoidType();
 } else {
 throw TypeError(
 "Return statement outside of any function");
 }
 }
```

## If 和 While

当检查`If`和`While`时，我们会访问每个内部节点以确保它们的类型被检查，但我们不强制执行任何类型。`conditional`可以被检查为布尔类型，但在 TypeScript（以及许多其他语言）中，这并不是必需的。我们也可以强制分支内的语句返回`void`，但这通常也不是必需的。

```js
 visitIf(node: If) {
 node.conditional.visit(this);
 node.consequence.visit(this);
 node.alternative.visit(this);
 return new VoidType();
 }

 visitWhile(node: While) {
 node.conditional.visit(this);
 node.body.visit(this);
 return new VoidType();
 }
```

然而，如果我们有一个三元条件操作，我们需要确保两个分支返回相同的数据类型。

## 可靠性

我们的类型检查器是完整的。但它是否是*可靠的*？一个类型系统如果是可靠的，那么编译时的类型总是与运行时一致。你能在我们的类型检查器中找到任何可靠性问题吗？

虽然我们检查每个`return`语句与注解的返回类型是否一致，但我们并没有检查每个控制流路径都有一个返回语句。以下是一个说明此问题的例子：

```js
function wrongReturnType(x: boolean): Array<number> {
 if (x) {
 return [42];
 }
}

function main() {
 var a = wrontReturnType(false);
 a[0]; // Segmentation fault
}
```

检查每个控制流路径都导向一个返回语句需要一点控制流分析，而这超出了本书的范围。

## 错误信息

我们可以通过将错误信息专门化到每个情况，而不是依赖于`assertType`生成的通用消息来改进我们的错误信息。

下一章：第十四章 动态类型

* * *
