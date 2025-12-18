# 第十一章：数组和堆分配

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/11-arrays-and-heap-allocation`](https://keleshev.com/compiling-to-assembly-from-scratch/11-arrays-and-heap-allocation)

从零开始编译汇编

由 Vladimir Keleshev 提供

让我们实现可以使用字面量表示法创建的简单数组：`[10, 20, 30]`。我们希望能够通过索引访问数组中的元素（`a[0]`），以及查询数组的长度（`length(a)`）。

```js
let a = [10, 20, 30];
assert(a[0] == 10);
assert(a[1] == 20);
assert(a[2] == 30);
assert(a[3] == undefined);  // Bounds checking
assert(length(a) == 3);
```

我们还希望实现边界检查。换句话说，如果我们请求一个超出边界的索引，我们不希望出现段错误。在这种情况下，在 JavaScript 中，我们期望返回 `undefined`。在其他语言中，它可能是 `null` 或者可以抛出异常。我们暂时忽略 `undefined` 仅仅是美化过的 `0` 这一事实。

为了实现这一功能，我们需要三个新的抽象语法树（AST）节点，如下表所示。

AST 构造函数签名摘要及示例

| AST 构造函数签名 | 示例 |
| --- | --- |
| `ArrayLiteral(args: Array<AST>)` `ArrayLookup(array: AST, index: AST)` `Length(array: AST)` | `[a1, a2, a3]` `array[index]` `length(array)` |

我们选择了 `ArrayLiteral` 这个名字，而不是 `Array`，是为了避免与我们在实现中已经大量使用的 JavaScript 内置 `Array` 类冲突。我们决定使用一个 `length` 函数而不是一个方法（如 JavaScript 中的那样），因为我们还没有支持方法。然而，我们可以为 `x.length` 这个特定目的提供一个特殊的语法。

我们添加了两个新的标记：`LEFT_BRACKET` 和 `RIGHT_BRACKET`，分别代表 “`[`” 和 “`]`”。我们还扩展了我们的语法（和解析器）并添加了两个新的规则，我们将它们集成到 `atom` 规则中。

```js
arrayLiteral <- LEFT_BRACKET args RIGHT_BRACKET

arrayLookup <- ID LEFT_BRACKET expression RIGHT_BRACKET

atom <- call / arrayLiteral / arrayLookup / scalar
 / LEFT_PAREN expression RIGHT_PAREN
```

在内存中布局数组有多种方式。最简单的是固定大小数组的布局。这种数组表示为一个指向包含数组长度和元素的单一内存块的指针。在下面的图中，你可以看到一个示例数组 `[10, 20, 30]` 的数组布局。

![固定大小数组布局的词图](img/9c7748769afe1802b02a494fc713f270.png)

固定大小数组布局的词图

然而，固定大小数组是，嗯，固定大小的。为了实现可增长或可调整大小的数组，我们需要一个更复杂的布局，如下面的图所示。它存储数组长度和其 *容量*，以及指向实际元素数组的指针。想法是在某种程度上 *超额分配* 数组，使得容量大于长度。这样，元素可以添加到数组中而不需要内存分配（在某种程度上）。当一个数组超出容量时，为元素分配一个新的内存块。

![可调整大小数组布局的词图](img/b757793282193066c44158e7efed93ff.png)

可调整大小数组布局的词图

让我们看看固定大小数组的代码生成。

## 数组字面量

我们从数组字面量的代码生成开始。

```js
class ArrayLiteral implements AST {
 constructor(elements: Array<AST>) {}

 emit(env: Environment) {
 let length = this.elements.length;
 emit(`  ldr r0, =${4 * (length + 1)}`);
 emit(`  bl malloc`);
 emit(`  push {r4, ip}`);
 emit(`  mov r4, r0`);
 emit(`  ldr r0, =${length}`);
 emit(`  str r0, [r4]`);
 this.elements.forEach((element, i) => {
 element.emit(env);
 emit(`  str r0, [r4, #${4 * (i + 1)}]`);
 });
 emit(`  mov r0, r4`);
 emit(`  pop {r4, ip}`);
 }

 equals(other: AST): boolean {…}
}
```

首先，我们调用`malloc`来分配足够的内存以存储数组的长度和元素。由于`malloc`接受要分配的字节数，我们需要将长度乘以四，并额外加一个字来存储长度本身。

```js
 let length = this.elements.length;
 emit(`  ldr r0, =${4 * (length + 1)}`);
 emit(`  bl malloc`);
```

然后，`malloc`返回指向新分配内存的指针到`r0`。然而，`r0`不是一个好的寄存器来执行这个操作。由于我们为每个数组元素生成代码，`r0`将被覆盖。因此，让我们使用调用保留寄存器`r4`，但在这之前，我们需要在堆栈上保存`r4`的先前值。

```js
 emit(`  push {r4, ip}`);
 emit(`  mov r4, r0`);
```

接下来，我们将数组长度存储在分配的内存空间的第一字中。

```js
 emit(`  ldr r0, =${length}`);
 emit(`  str r0, [r4]`);
```

之后，我们为每个元素生成代码并将其存储在相应的内存槽中。

```js
 this.elements.forEach((element, i) => {
 element.emit(env);
 emit(`  str r0, [r4, #${4 * (i + 1)}]`);
 });
```

我们通过返回`r0`中的指针并恢复调用保留的`r4`来完成。

```js
 emit(`  mov r0, r4`);
 emit(`  pop {r4, ip}`);
```

## 数组查找

接下来是数组查找的代码生成。

```js
class ArrayLookup implements AST {
 constructor(array: AST, index: AST) {}

 emit(env: Environment) {
 this.array.emit(env);
 emit(`  push {r0, ip}`);
 this.index.emit(env);
 emit(`  pop {r1, ip}`);
 emit(`  ldr r2, [r1]`);
 emit(`  cmp r0, r2`);
 emit(`  movhs r0, #0`);
 emit(`  addlo r1, r1, #4`);
 emit(`  lsllo r0, r0, #2`);
 emit(`  ldrlo r0, [r1, r0]`);
 }

 equals(others: AST) {…}
}
```

首先，我们将数组指针存储在`r1`中，将数组索引存储在`r0`中。

```js
 this.array.emit(env);
 emit(`  push {r0, ip}`);
 this.index.emit(env);
 emit(`  pop {r1, ip}`);
```

然后，我们将数组长度加载到`r2`中，并与数组索引进行比较以执行边界检查。如果数组索引超出范围，我们执行带有`hs`后缀的条件`mov`并返回零。如果成功，则执行带有`lo`后缀的三条指令。在这三条指令中，我们将数组索引转换为字偏移量。我们加 4 以跳过长度槽，并执行*逻辑左移*或`lsl`以将字偏移量转换为字节数。左移两位实际上等同于乘以 4。

```js
 emit(`  ldr r2, [r1]`);
 emit(`  cmp r0, r2`);
 emit(`  movhs r0, #0`);
 emit(`  addlo r1, r1, #4`);
 emit(`  lsllo r0, r0, #2`);
 emit(`  ldrlo r0, [r1, r0]`);
```

然而，使用一些我们在书中没有涵盖的 ARM 汇编语言特性（如自动递增和桶形移位器），我们可以将相同的代码缩短为以下形式：

```js
 emit(`  ldr r2, [r1], #4`);
 emit(`  cmp r0, r2`);
 emit(`  movhs r0, #0`);
 emit(`  ldrlo r0, [r1, r0, lsl #2]`);
```

这是一种同时执行边界检查并将数组索引转换为字节数的优雅方式。

## 数组长度

数组长度可以通过直接跟随数组指向的槽来获得。

```js
class Length implements AST {
 constructor(array: AST) {}

 emit(env: Environment) {
 this.array.emit(env);
 emit(`  ldr r0, [r0, #0]`);
 }

 equals(others: AST) {…}
}
```

## 字符串

字符串可以被视为具有特定编码的字节数组。它们可以像数组一样实现，但使用加载和存储指令的字变体，即`ldrb`和`strb`，而不是常规的`ldr`和`str`。然而，当我们看到`a[i]`时，我们需要知道它是一个数组还是一个字符串以执行正确的代码。我们要么需要在编译时知道变量的静态类型，要么通过检查数据结构在运行时知道其动态类型（或标记）。但在我们深入之前，我们需要了解一个特定的模式，这将帮助我们更好地维护代码。

下一节：第十二章 访问者模式

* * *
