# 第二章：TypeScript 基础

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/02-typescript-basics`](https://keleshev.com/compiling-to-assembly-from-scratch/02-typescript-basics)

从头开始编译汇编

由 Vladimir Keleshev 提供

本章将使你熟悉 TypeScript。如果你已经了解它，可以自由跳过。

TypeScript 被设计为 JavaScript 的超集（或扩展），它增加了类型注解和类型检查。因此，除了类型注解之外，它只是现代 JavaScript。

让我们开始吧。首先，`console.log` 将消息打印到控制台：

```js
console.log("Hello, world!");
```

字符串使用双引号或单引号。使用反引号编写的字符串称为 *模板字符串*。它们可以是多行的，*并且* 可以进行插值（或注入表达式），就像在模板中一样：

```js
console.log(`2 + 2 = ${2 + 2}`); // Prints: 2 + 2 = 4
```

接下来，`console.assert` 是一种快速且便携的方式来测试某些内容，如果你手头没有测试框架：

```js
console.assert(2 === 2); // Does nothing
console.assert(0 === 2); // Raises an exception
```

三重等于 `===` 是严格（或精确）相等，而双等于 `==` 是松散相等。同样，`!==` 和 `!=` 也是如此。例如，在松散相等中 `true` 等于 `1`，但在严格相等中它们不相等。我们将在编译器代码中仅使用严格相等，但在基线语言中我们将使用 `==` 和 `!=` 操作符。

要异常终止程序，你可以抛出一个异常：

```js
throw Error("Error message goes here.");
```

定义自己的异常是一个好习惯，但为了简洁，我们在这本书中会使用内置的异常。

可以使用 `function` 关键字定义函数：

```js
function add(a: number, b: number) {
 return a + b;
}
```

在许多情况下，TypeScript 中的类型注解是可选的。然而，对于大多数函数（和方法）参数来说，它们是必要的。到目前为止，我们一直在描述只是纯 JavaScript 的特性。然而，这里我们有一个带有 *类型注解* 的函数。参数的类型被注解为 `number`。

定义函数的另一种方式是编写所谓的 *箭头函数*（也称为 *匿名函数* 或 *lambda 函数*）：

```js
let add = (a, b) => a + b;
```

接下来是变量。在 TypeScript（和 JavaScript）中有几种形式的变量。变量通过 `var`、`let`（和 `const`）绑定。虽然 `var` 的作用域在函数（或方法）级别，但 `let` 的作用域由花括号限定。

比较 `var`:

```js
function f() {
 var x = 1;
 {
 var x = 2;
 }
 console.log(x); // Prints 2
}
```

使用 `let`：

```js
function f() {
 let x = 1;
 {
 let x = 2;
 }
 console.log(x); // Prints 1
}
```

我们将在编译器代码中仅使用 `let`，然而，需要提及 `var`，因为我们将为基线语言实现 `var`。

虽然大多数情况下是可选的，但绑定可以进行类型注解：

```js
let a: Array<number> = [1, 2, 3];
```

这里，`Array<number>` 表示一个数字数组。

TypeScript，像 JavaScript 一样，允许使用字面量正则表达式。字符串用引号分隔，而字面量正则表达式用正斜杠分隔：

```js
let result = "hello world".match(/[a-z]+/);
console.assert(result[0] === "hello");
```

类是 TypeScript 中创建自定义数据类型的首选方式。以下是一个简单的数据类：

```js
class Pair {
 public first: number;
 public second: number;

 constructor(first: number, second: number) {
 this.first = first;
 this.second = second;
 }
}
```

你可以使用 `new` 关键字创建一个新对象：

```js
let origin = new Pair(0, 0);
```

TypeScript 为定义像这样简单的数据类提供了快捷方式，其中构造函数参数立即赋值为实例变量：

```js
class Pair {
 constructor(public first: number,
 public second: number) {}
}
```

注意在参数列表中指定的`public`关键字。这与之前的定义完全相同。在这本书中，我们将大量使用这个快捷方式。

我们还将定义一些可能与内置的冲突的类，比如`Boolean`和`Number`。为了避免这种情况，请确保在模块作用域而不是全局作用域中定义它们。实现这一点最简单的方法是确保在你的源文件中添加一个空的`**export**` `{}`。使用`**import**`或`**export**`关键字将文件作用域从全局作用域更改为模块作用域。

接下来是一个更详细的示例，包括静态变量`zero`、静态方法`origin`和实例方法`toString`。记住，静态变量和静态方法只是通过类名命名空间的全局变量和全局方法。

```js
class Pair {
 static zero = 0;

 static origin() {
 return new Pair(0, 0);
 }

 constructor(public first: number,
 public second: number) {}

 toString() {
 return `(${this.first}, ${this.second})`;
 }
}
```

你可能会发现这本书中的一些空白和格式选择看起来很奇怪。这是因为代码列表是为了紧凑性而优化的，这对于印刷书籍来说很重要。

* * *

TypeScript 还有很多内容，但我们将主要限制在本部分描述的子集。

下一章：第三章 高级编译器概述

* * *
