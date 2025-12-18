# 第十四章：动态类型

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/14-dynamic-typing`](https://keleshev.com/compiling-to-assembly-from-scratch/14-dynamic-typing)

从头开始编译汇编

由 Vladimir Keleshev 提供

在动态类型中，你期望能够查询任何变量的数据类型，使用 `instanceof` 或其他机制。目前我们无法在运行时区分数字和（指向数组的）指针。在我们所拥有的静态类型系统中，这并不重要，因为这种比较在运行时是不允许的。

那么，我们如何区分指针和数字，false 和零等等？这通常是通过 *标签* 来实现的。

## 标签

标签是在每个字中为特定目的保留一个或多个位，例如，用于区分其类型。

对于我们的编译器，我们将采用以下图中描述的两位标签方案。

![我们的两位标签方案](img/b09eab6a73f6e677f3e9d1a02bb4ccf8.png)

我们的两位标签方案

但我们如何“承担”这两个位？这不会改变实际的数值或指针值吗？

让我们先谈谈指针。

## 指针标签

我们期望指针是字对齐的。这意味着字中的最后两位总是 `0b00`。这意味着我们可以在那里存储一个两位标签来区分指针。

处理这个问题最简单的方法是选择标签 `0b00`。这正是许多实现所做的事情。但我们也可以选择不同的标签，比如 `0b01`，就像我们做的那样。在一般情况下，当我们想要操作一个指针时，我们可以移除标签，执行操作，如果需要的话再放回去。然而，拥有 `0b01` 的标签仅仅意味着我们指向字中的第二个字节。因此，为了加载第一个字，我们可以在偏移量 `-1` 处加载，而不是在偏移量 `0` 处加载。这样 `ldr r0, [r1]` 就变成了 `ldr r0, [r1, #-1]`，而 `ldr r0, [r1, #4]` 就变成了 `ldr r0, [r1, #3]` 等等！这意味着我们并没有失去太多。最坏的情况下，只需要一条指令来清除标签。

目前我们处理的唯一类型的指针是数组指针。为了将它们与其他堆分配的对象区分开来，通常会在数据结构本身中编码关于对象类型的信息。

接下来，让我们谈谈整数。

## 整数标签

如果我们为标签分配两个位，这意味着我们的整数从 32 位缩小到 30 位。这是一个奇数大小的整数！然而，在动态类型语言中，整数通常默默地提升为浮点数或任意精度数。这意味着整数是 30 位的这一事实并没有暴露给语言的用户。这些奇数大小的整数被称为 *小整数*。

因此，当然，为了处理小整数，我们可以移除标记，执行操作，然后添加标记。然而，这并不总是必要的。我们选择了标记`0b00`用于整数。这意味着大多数算术和逻辑操作将直接在标记整数上工作。例如，`3 + 4 = 7`以前是`0b11 + 0b100 = 0b111`，但现在它是`0b11_00 + 0b100_00 = 0b111_00`。（我们使用了下划线在视觉上分隔标记和值。）一个值得注意的例外是乘法，它需要一个指令——右移——来修复它，即使这样，也有方法将这个移位“免费”地作为另一个指令的一部分。

通过选择这个标记，我们可以实现良好的性能，并避免过多地改变我们的代码生成过程。

## 真值和假值标记

我们还有两个可用的标记：`0b10`和`0b11`。我们分别用它们表示所有假值和真值。这是为了强调标记通常以创造性的方式选择，以简化常见操作。在我们的语言中，我们只剩下`true`、`false`和`undefined`。我们将分别将它们赋值为`0b1_11`、`0b1_10`和`0b0_10`。这也允许我们通过检查位`0b1_10`是否设置来快速检查一个值是否为布尔值。

## 代码生成

现在，我们将修改我们的代码生成过程以适应我们新的标记策略。但首先，让我们介绍一些有用的常量。我们开始于四个标记：

```js
let numberTag = 0b00;
let pointerTag = 0b01;
let falsyTag = 0b10;
let truthyTag = 0b11;
```

然后我们文字值`true`、`false`和`undefined`的位模式：

```js
let trueBitPattern = 0b111;
let falseBitPattern = 0b110;
let undefinedBitPattern = 0b010;
```

以及一个辅助函数，用于将整数转换为小整数（换句话说，为整数添加一个标记）：

```js
let toSmallInteger = (n: number) => n << 2;
```

最后，一个掩码，它将帮助我们从一个单词中提取标记：

```js
let tagBitMask = 0b11;
```

现在，让我们继续到代码生成过程。

## 文字

现在生成整数数现在使用`toSmallInteger`辅助函数：

```js
 visitNumber(node: Number) {
 emit(`  ldr r0, =${toSmallInteger(node.value)}`);
 }
```

布尔值和未定义值使用定义的常量：

```js
 visitBoolean(node: Boolean) {
 if (node.value) {
 emit(`  mov r0, #${trueBitPattern}`);
 } else {
 emit(`  mov r0, #${falseBitPattern}`);
 }
 }

 visitUndefined(node: Undefined) {
 emit(`  mov r0, #${undefinedBitPattern}`);
 }
```

## 运算符

以前，类型系统强制要求运算符与正确的数据类型一起使用。例如，加法只允许在数字上使用。现在，我们需要定义加法，以便它在某种程度上与任何类型一起工作。例如，JavaScript 有复杂的强制规则。如果你将一个空数组`[]`和一个数字`1`相加，结果是字符串`"1"`。对于我们的编译器，我们将采用简化的强制规则。如果两个操作数都是数字，结果是数字，否则——`undefined`。

以加法为例，我们可以这样实现这样的运算符：

```js
 visitAdd(node: Add) {
 node.left.visit(this);
 emit(`  push {r0, ip}`);
 node.right.visit(this);
 emit(`  pop {r1, ip}`);

 // Are both small integers?
 emit(`  orr r2, r0, r1`);
 emit(`  and r2, r2, #${tagBitMask}`);
 emit(`  cmp r2, #0`);

 emit(`  addeq r0, r1, r0`);
 emit(`  movne r0, #${undefinedBitPattern}`);
 }
```

要实现否定运算符，我们第一次需要处理真值和假值。在我们的标记系统中，如果一个单词为零，或者以假值标记`0b10`结尾，那么它就是假的。这就是为什么我们需要进行两次比较。

```js
 visitNot(node: Not) {
 node.term.visit(this);

 // Is falsy?
 emit(`  cmp r0, #0`);
 emit(`  andne r0, r0, #${tagBitMask}`)
 emit(`  cmpne r0, #${falsyTag}`);

 emit(`  moveq r0, #${trueBitPattern}`);
 emit(`  movne r0, #${falseBitPattern}`);
 }
```

我们需要在更多的地方进行假值检查，所以让我们将其提取到一个辅助方法中：

```js
 emitCompareFalsy() {
 emit(`  cmp r0, #0`);
 emit(`  andne r0, r0, #${tagBitMask}`)
 emit(`  cmpne r0, #${falsyTag}`);
 }
```

在`If`和`While`节点中的条件将需要使用`emitCompareFalsy`代替`cmp r0, #0`。

## 数组

数组字面量与之前基本相同，但有两大例外。首先，长度现在存储为一个小的整数。其次，添加了一个指针标签，如下所示。之前，`mov r0, r4` 将我们的调用保留指针移动到返回寄存器 `r0`。现在，我们通过使用 `add r0, r4, #1` 同时完成相同操作并添加一个 `0b01` 标签。我们可以这样做，因为 `malloc` 只会返回最后两位为 `0b00` 的字对齐指针。

```js
 visitArrayLiteral(node: ArrayLiteral) {
 emit(`  ldr r0, =${4 * (node.args.length + 1)}`);
 emit(`  bl malloc`);
 emit(`  push {r4, ip}`);
 emit(`  mov r4, r0`);
 emit(`  ldr r0, =${toSmallInteger(node.args.length)}`);
 emit(`  str r0, [r4]`);
 node.args.forEach((arg, i) => {
 arg.visit(this);
 emit(`  str r0, [r4, #${4 * (i + 1)}]`);
 });
 emit(`  add r0, r4, #1`);  // Move to r0 and add tag
 emit(`  pop {r4, ip}`);
 }
```

数组长度原语几乎与之前相同，只是我们使用 `-1` 偏移量来取消标签。

```js
 visitLength(node: Length) {
 node.array.visit(this);
 emit(`  ldr r0, [r0, #-1]`);
 }
```

对于数组查找，我们不使用任何技巧来消除标签的开销，而是简单地发出一个额外的指令在查找之前移除标签。此外，如果边界检查失败，我们返回 `undefined` 而不是零。

```js
 visitArrayLookup(node: ArrayLookup) {
 node.array.visit(this);
 emit(`  bic r0, r0, #${pointerTag}`); // Remove tag
 emit(`  push {r0, ip}`);
 node.index.visit(this);
 emit(`  pop {r1, ip}`);
 emit(`  ldr r2, [r1], #4`);
 emit(`  cmp r0, r2`);
 emit(`  movhs r0, #${undefinedBitPattern}`);
 emit(`  ldrlo r0, [r1, r0]`);
 }
```

注意一个有趣的巧合。由于我们现在将数组长度存储为一个小整数，我们不需要左移来将数组长度转换为字节偏移。小整数由于 `0b00` 标签已经左移了两位。

* * *

`Function` 节点的代码生成需要稍作调整：当函数以没有返回语句结束时要返回 `undefined` 而不是零。其余的节点（`Call`、`Return`、`Id`、`Assign`、`Var`、`Block`）在代码生成中不需要更改以适应动态类型。

当启用类型检查时，我们的现有测试套件应该无需修改即可成功。唯一可观察到的变化是整数现在为 30 位而不是 32 位。然而，如果我们关闭类型检查，我们可以表达更多有趣的程序，如下所示：

```js
function isBoolean(x) {
 if (x == true) {
 return true;
 } else if (x == false) {
 return true;
 } else {
 return false;
 }
}

function main() {
 var a = [];
 assert(a[1] + 2 == undefined);

 assert(!isBoolean(undefined));
}
```

下一章：第十五章 垃圾回收

* * *
