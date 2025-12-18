# 第四章：抽象语法树

> 原文：[从零开始编译汇编](https://keleshev.com/compiling-to-assembly-from-scratch/04-abstract-syntax-tree)

从零开始编译汇编

由 Vladimir Keleshev 编写

抽象语法树，或 AST，是编译器的核心概念。

AST 是一种数据结构。它是一个**树**，它模拟了编程语言的**语法**。但它**抽象**了语法的一些平凡细节，例如括号或分号的精确位置。

这棵树由**节点**组成，其中每个节点都是一个数据对象，它表示语言中的语法结构。一个`Return`节点可以表示一个`return`语句，一个`Add`节点可以表示`+`运算符，一个引用变量的标识符可以使用`Id`节点，等等。

例如，以下代码行：

```js
return n * factorial(n - 1);
```

可以有如下 AST：

```js
new Return(
 new Multiply(
 new Id("n"),
 new Call("factorial", [
 new Subtract(new Id("n"), new Number(1)),
 ])))
```

在下一张图中，你可以看到同一棵树的图形表示。

![对应于 n * factorial(n - 1) 的 AST](img/12cdfb9105052361e252391b35466581.png)

对应于`**return**` `n * factorial(n - 1)`的 AST

为什么使用抽象语法树（AST）？当处理语言结构时，AST 使得对其进行操作变得方便：查询、构建和修改。

我们设计 AST，使其便于我们使用：我们想要进行哪些类型的转换，哪些内容要查询或更改。

例如，AST 可以包含源位置信息以进行错误报告。或者，如果我们的编译器需要处理这些，它可以包含文档注释。或者，如果我们想要有风格保留的转换，如自动重构，它可以包含所有注释和一些空白的概念。

> **实际上，**
> 
> 还有**具体语法树**，也称为**解析树**。它们反映了结构层次直到每个输入符号。它们通常包含太多我们不关心的细节，当我们编写编译器时。但它们非常适合风格保留的转换。

我们将代表 AST 中的每种节点类型作为一个单独的类：`Return`、`Multiply`、`Id`等。然而，在类型级别上，我们希望能够引用“任何 AST 节点”。为此，TypeScript 提供了几个工具：

+   接口，

+   抽象类，

+   联合类型。

这两种方法都有效。我们将使用一个接口：

```js
interface AST {
 equals(other: AST): boolean;
}
```

每种 AST 节点类型都将是一个实现了`AST`接口的类。它从简单开始，只有`equals`方法。我们将根据需要向此接口（和类）添加方法。

`equals`方法主要用于单元测试。实现相当平凡，所以大部分情况下，我们将省略它，并用省略号替换其主体。

## 数字节点

我们的第一个节点是`Number`。

```js
class Number implements AST {
 constructor(public value: number) {}

 equals(other: AST) {…}
}
```

在这里，我们使用了 TypeScript 的快捷方式来快速定义实例变量，使用`public`作为构造函数参数。记住，它等同于以下内容：

```js
class Number implements AST {
 public value: number;

 constructor(value: number) {
 this.value = value;
 }

 equals(other: AST) {…}
}
```

它节省了我们相当多的输入，这将会很有用，因为我们需要定义许多类型的 AST 节点。

我们将我们的 AST 节点命名为 `Number`，因为 JavaScript 和 TypeScript 中的数据类型被称为 `number`。然而，我们的编译器将只处理无符号整数。

`Number` 节点的示例

| 源代码 | 抽象语法树 (AST) |
| --- | --- |
| `0` | `new Number(0)` |
| `42` | `new Number(42)` |

## 标识符节点

标识符，或简称为 *id*，指的是代码中的变量。

```js
class Id implements AST {
 constructor(public value: string) {}

 equals(other: AST) {…}
}
```

`Id` 节点的示例

| 源代码 | 抽象语法树 (AST) |
| --- | --- |
| `x` | `new Id("x")` |
| `hello` | `new Id("hello")` |

## 操作符节点

下一个 AST 节点是 `Not`，它代表逻辑否定运算符 (`!`)。

```js
class Not implements AST {
 constructor(public term: AST) {}

 equals(other: AST) {…}
}
```

`Not` 节点的示例

| 源代码 | 抽象语法树 (AST) |
| --- | --- |
| `!x` | `new Not(new Id("x"))` |
| `!42` | `new Not(new Number(42))` |

否定是唯一的一种 *前缀运算符*（或 *一元运算符*）。然而，我们定义了几个 *中缀运算符*（或 *二元运算符*）。

等于运算符 (`==`) 和其相反运算符 (`!=`)。

```js
class Equal implements AST {
 constructor(public left: AST, public right: AST) {}

 equals(other: AST) {…}
}
```

```js
class NotEqual implements AST {
 constructor(public left: AST, public right: AST) {}

 equals(other: AST) {…}
}
```

`Equal` 和 `NotEqual` 节点的示例

| 源代码 | 抽象语法树 (AST) |
| --- | --- |
| `x == y` | `new Equal(new Id("x"), new Id("y"))` |
| `10 != 25` | `new NotEqual(new Number(10), new Number(25))` |

加法 (`+`)、减法 (`-`)、乘法 (`*`) 和除法 (`/`)：

```js
class Add implements AST {
 constructor(public left: AST, public right: AST) {}

 equals(other: AST) {…}
}
```

```js
class Subtract implements AST {
 constructor(public left: AST, public right: AST) {}

 equals(other: AST) {…}
}
```

```js
class Multiply implements AST {
 constructor(public left: AST, public right: AST) {}

 equals(other: AST) {…}
}
```

```js
class Divide implements AST {
 constructor(public left: AST, public right: AST) {}

 equals(other: AST) {…}
}
```

`Add` 和 `Multiply` 节点的示例

| 源代码 | 抽象语法树 (AST) |
| --- | --- |
| `x + y` |

```js
 new Add(new Id("x"),
 new Id("y"))
```



|  |  |
| --- | --- |
| `10 * 25` |

```js
 new Multiply(new Number(10),
 new Number(25))
```



注意，由于参数本身就是 AST，这意味着它们可以任意嵌套。

例如，`42 + !(20 != 10)` 将变为：

```js
new Add(new Number(42),
 new Not(new NotEqual(new Number(20),
 new Number(10))))
```

并非所有 AST 节点的组合都是有意义的。然而，上述组合在 JavaScript 中恰好是有效的。

## 调用节点

`Call` 指的是函数名（或 *callee*），以及参数数组。例如，`f(x)` 变为：`new Call("f", [new Id("x")])`。

```js
class Call implements AST {
 constructor(public callee: string,
 public args: Array<AST>) {}

 equals(other: AST) {
 return other instanceof Call &&
 this.callee === other.callee &&
 this.args.length === other.args.length &&
 this.args.every((arg, i) =>
 arg.equals(other.args[i]));
 }
}
```

我们的基础编译器的语言限制使得只能调用命名函数。

我们不能将参数数组命名为 `arguments`，因为这会与 JavaScript 内置的 `arguments` 对象冲突。所以我们有些不情愿地将其称为 `args`。

`Call` 节点很有趣，因为它既有原始字符串，又有 AST 数组作为其成员。JavaScript 没有一个统一的协议来处理相等性；这就是为什么 `Call` 是实现 JavaScript 中 `equals` 方法的优秀示例：

+   它使用 `instanceof` 运算符来检查另一个 AST 是否也是 `Call`。

+   它使用 `===` 运算符比较 `callee` 字符串。

+   它使用每个参数的 `.equals` 方法来比较 AST 节点。

+   它通过长度比较数组并检查`每个`元素。

除了 JavaScript 之外的语言通常有更优雅的方式来处理结构相等性。

`Call` 节点的示例

| 源代码 | 抽象语法树 (AST) |
| --- | --- |
| `f(x, y)` |

```js
 new Call("f", [
 new Id("x"),
 new Id("y"),
 ])
```



|  |  |
| --- | --- |
| `factorial(n - 1)` |

```js
 new Call("factorial", [
 new Subtract(new Id("n"),
 new Number(1)),
 ])
```



## 返回节点

```js
class Return implements AST {
 constructor(public term: AST) {}

 equals(other: AST) {…}
}
```

`Return` 节点的示例

| 源代码 | 抽象语法树 (AST) |
| --- | --- |
| `return 0;` |

```js
new Return(new Number(0))
```



|  |  |
| --- | --- |
| `return n - 1;` |

```js
new Return(
 new Subtract(new Id("n"),
 new Number(1)))
```



## 块节点

`Block` 指的是由花括号分隔的代码块。

```js
class Block implements AST {
 constructor(public statements: Array<AST>) {}

 equals(other: AST) {…}
}
```

`Block` 节点的示例

| 源代码 | 抽象语法树 (AST) |
| --- | --- |



```js
{}
```



```js
new Block([])
```



|  |  |
| --- | --- |



```js
{
 f(x);
 return y;
}
```



```js
new Block([
 new Call("f", [new Id("x")]),
 new Return(new Id("y")),
])
```



## If 节点

`If`节点有三个分支：

+   `conditional`指的是评估为真或假的表达式，

+   `consequence`是在真情况下采取的分支，并且

+   `alternative`是在假情况下采取的分支。

```js
class If implements AST {
 constructor(public conditional: AST,
 public consequence: AST,
 public alternative: AST) {}

 equals(other: AST) {…}
}
```

这样，以下：

```js
if (x == 42) {
 return x;
} else {
 return y;
}
```

变成：

```js
new If(
 new Equal(new Id("x", new Number(42))),
 new Block([new Return(new Id("x"))]),
 new Block([new Return(new Id("y"))]),
)
```

大括号对于 if 语句是可选的，因此以下：

```js
if (x == 42)
 return x;
else
 return y;
```

变成：

```js
new If(
 new Equal(new Id("x", new Number(42))),
 new Return(new Id("x")),
 new Return(new Id("y")),
)
```

我们如何表示没有`else`分支的`if`？我们可以为它创建一个单独的节点，或者我们可以做一个简单的技巧：将`if (x) y`以与`if (x) y else {}`相同的方式表示。换句话说，通过放置一个空的`Block`作为`alternative`。

## 函数定义节点

函数定义由函数名、参数数组以及函数体组成。

```js
class Function implements AST {
 constructor(public name: string,
 public parameters: Array<string>,
 public body: AST) {}

 equals(other: AST) {…}
}
```

考虑以下函数定义。

```js
function f(x, y) {
 return y;
}
```

当转换为 AST 时，它变成如下所示。

```js
new Function("f", ["x", "y"], new Block([
 new Return(new Id("y")),
]))
```

注意，在函数定义中，参数是字符串，而在函数调用中，它们是 AST。这一事实反映了函数调用可以有嵌套表达式，而函数定义只是简单地列出传入的变量名。

## 变量声明节点

`Var`节点用于变量声明。所以`var x = 42;`变成`new Var("x", new Number(42))`。

```js
class Var implements AST {
 constructor(public name: string, public value: AST) {}

 equals(other: AST) {…}
}
```

## 赋值节点

赋值用`Assign`节点表示。

```js
class Assign implements AST {
 constructor(public name: string,
 public value: AST) {}

 equals(other: AST) {…}
}
```

赋值与变量声明在以下方面有所不同：赋值改变现有变量的值，而不是定义一个新的变量。至少，这是我们将会假设的区分。JavaScript 允许对尚未定义的变量进行赋值；在这种情况下，它将创建一个全局变量。另一方面，TypeScript 不允许这种可能导致错误的行为。

`Assign`节点的示例

| 源代码 | AST |
| --- | --- |
| `x = 42;` |

```js
new Assign("x", new Number(42))
```



|  |  |
| --- | --- |
| `y = a + b;` |

```js
new Assign("y",
 new Add(new Id("a"),
 new Id("b"))))
```



## While 循环节点

我们 AST 的最后一个节点和我们的基线语言的最后一个结构是 while 循环。

```js
class While implements AST {
 constructor(public conditional: AST,
 public body: AST) {}

 equals(other: AST) {…}
}
```

`While`节点的示例

| 源代码 | AST |
| --- | --- |



```js
while (x)
 f();
```



```js
new While(
 new Id("x"),
 new Call("f", []))
```



|  |  |
| --- | --- |



```js
while (x) {
 f();
}
```



```js
new While(new Id("x"), new Block([
 new Call("f", []),
]))
```



## 总结

让我们看看一个更大的代码片段转换成 AST 的例子。记住我们的阶乘函数：

```js
function factorial(n) {
 var result = 1;
 while (n != 1) {
 result = result * n;
 n = n - 1;
 }
 return result;
}
```

这里是相应的 AST：

```js
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
]))
```

我们以一个表格结束本章，该表格总结了我们所涵盖的所有 AST 构造函数，它们的签名以及这些 AST 节点可以表示的源代码示例。

AST 构造函数签名及其示例总结

| AST 构造函数签名 | 示例 |
| --- | --- |



```js
Number(value: number)
Id(value: string)
Not(term: AST)
Equal(left: AST, right: AST)
NotEqual(left: AST, right: AST)
Add(left: AST, right: AST)
Subtract(left: AST, right: AST)
Multiply(left: AST, right: AST)
Divide(left: AST, right: AST)
```



```js
42
x
!term
left == right
left != right
left + right
left - right
left * right
left / right
```



|  |  |
| --- | --- |



```js
Call(callee: string,
 args: Array<AST>)
```



```js
callee(a1, a2, a3)
```



|  |  |
| --- | --- |



```js
Return(term: AST)
```



```js
return term;
```



|  |  |
| --- | --- |



```js
Block(statements: Array<AST>)
```



```js
{ s1; s2; s3; }
```



|  |  |
| --- | --- |



```js
If(conditional: AST,
 consequence: AST,
 alternative: AST)
```



```js
if (conditional)
 consequence
else
 alternative
```



|  |  |
| --- | --- |



```js
Function(
 name: string,
 parameters: Array<string>,
 body: AST,
)
```



```js
function name(p1) {
 body;
}
```



|  |  |
| --- | --- |



```js
Var(name: string,
 value: AST)
```



```js
var name = value;
```



|  |  |
| --- | --- |



```js
Assignment(name: string,
 value: AST)
```



```js
name = value;
```



|  |  |
| --- | --- |



```js
While(conditional: AST,
 body: AST)
```



```js
while (conditional)
 body;
```



在接下来的两章中，你将学习如何将程序从源代码转换为 AST，换句话说，就是关于*解析*。

下一章：第五章\. 解析器组合器

* * *
