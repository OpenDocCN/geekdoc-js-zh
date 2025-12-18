# 第八章：第二遍：代码生成

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/08-second-pass-code-generation`](https://keleshev.com/compiling-to-assembly-from-scratch/08-second-pass-code-generation)

从头开始编译到汇编

由 Vladimir Keleshev 编写

我们编译器的第二遍和最后一遍被称为 *发射器* 或 *代码生成器*，因为它生成汇编代码。它将程序的 AST 转换为汇编指令。我们如何组织这些呢？我们通过添加一个名为 `emit` 的新方法来扩展 `AST` 接口。

```js
interface AST {
 emit(): void;
 equals(other: AST): boolean;
}
```

这是一个很酷的方法——你可能这么说——它不接受任何参数，也不返回任何内容（或 `void`）。那么，它的用途是什么呢？首先，它接受隐式的 `this` 参数，以及每个节点的所有实例变量。其次，尽管签名返回类型是 `void`，当这个方法被调用时，它将作为 *副作用* 输出汇编代码。为此，我们将使用一个 `emit` 函数（而不是一个方法）。使用这个函数，我们可以输出这样的指令：

```js
emit("  add r1, r2, r3");
```

更常见的是，在输出代码时，我们会使用模板字符串进行字符串插值：

```js
let x = 42;
emit(`  add r1, r2, #${x}`);
```

这将输出 `add r1, r2, #42`。我们如何实现 `emit` 函数呢？我们可以定义它，使其向数组追加一行，或者将一行写入文件。但为了简单起见，我们先这样定义它：

```js
let emit = console.log;
```

没错，`emit` 简单地打印到控制台的标准输出通道。目前来说，这已经足够好了。我们可以将标准输出重定向到文件，然后将其组装起来。

那么，关于 `emit` 这个 *方法* 呢？首先，我们在每个 AST 节点上定义这个方法以满足接口。一开始，一个虚拟实现就足够了。让我们以 `Number` 节点为例。

```js
class Number implements AST {
 constructor(public value: number) {}

 emit() { throw Error("Not implemented yet"); }

 equals(other: AST) {…}
}
```

接下来，我们将修改每个节点以输出正确的汇编代码。例如，对于 `Number` 节点，它最终将是这样的：

```js
emit() {
 emit(`  ldr r0, =${this.value}`);
}
```

这样，每个 AST 节点都将能够输出相应的汇编代码。具有其他节点作为实例变量的节点将能够调用它们的 `emit` 方法。请注意，调用顺序很重要。

我们将遵循一条规则：每个节点在输出时，都会生成将结果放入寄存器 `r0` 的汇编代码。这已经是如何函数调用的方式了，但我们将其扩展到所有产生值的节点。这样，当我们输出时，我们将知道每个值最终会去哪里。

## 测试平台

为了测试我们正在开发的发射器，我建议添加几个临时的 AST 节点：`Main` 和 `Assert`。当我们教会编译器输出每种节点时，我们希望立即看到结果，而不是等到我们可以定义一个稍微复杂一些的函数，比如 `assert`，所以我们将其简化。对于 `Main` 也是一样：我们希望在定义函数之前，我们的编译器能够生成程序的入口点。我们将 `Main` 定义为接受一个语句数组。

```js
class Main implements AST {
 constructor(public statements: Array<AST>) {}

 emit() { /* TODO */ }

 equals(other: AST) {…}
}
```

虽然 `Assert` 只接受一个 `condition` 条件，它将基于此进行断言：

```js
class Assert implements AST {
 constructor(public condition: AST) {}

 emit() { /* TODO */ }

 equals(other: AST) {…}
}
```

我们目前省略了 `emit` 方法。

我们对`functionStatement`解析器进行了轻微修改，以便临时生成`Main`节点，以防函数名称是`main`：

```js
// functionStatement <-
//   FUNCTION ID LEFT_PAREN parameters RIGHT_PAREN
//   blockStatement
let functionStatement: Parser<AST> =
 FUNCTION.and(ID).bind((name) =>
 LEFT_PAREN.and(parameters).bind((parameters) =>
 RIGHT_PAREN.and(blockStatement).bind((block) =>
 constant(
 name === 'main'
 ? new Main(block.statements)
 : new Function(name, parameters, block)))));
```

与`Assert`类似：如果被调用者名为`assert`，我们就在`call`解析器内部生成它。

```js
// call <- ID LEFT_PAREN args RIGHT_PAREN
let call: Parser<AST> =
 ID.bind((callee) =>
 LEFT_PAREN.and(args.bind((args) =>
 RIGHT_PAREN.and(constant(
 callee === 'assert'
 ? new Assert(args[0])
 : new Call(callee, args))))));
```

一旦我们有足够的功能来定义自己的函数，如`assert`，我们就能回滚这个操作。

## 主入口点

让我们想象一下我们的主入口点在汇编中的样子。

```js
.global main
main:
 push {fp, lr}
 …
 mov r0, #0
 pop {fp, pc}
```

与之前一样，我们需要将`main`标签声明为`.global`。我们不会严格需要它，直到我们实现调用，但让我们保存`lr`寄存器，这样它就不会最终被破坏，并且我们可以安全地从`main`返回。由于我们需要在双字边界对齐堆栈，我们可以将`lr`和其他寄存器一起推入。我们可以将`ip`作为虚拟寄存器推入，但为什么不也推入帧指针`fp`以启动“经典”函数前导呢。然后我们生成内部语句。我们将返回值设置为零，这将成为程序的默认返回代码。我们以“经典”后导结束，其中我们恢复`fp`，通过将保存的`lr`推入`pc`，我们从`main`返回。 

现在我们可以实现`Main`的`emit`方法：

```js
class Main implements AST {
 constructor(public statements: Array<AST>) {}

 emit() {
 emit(`.global main`);
 emit(`main:`);
 emit(`  push {fp, lr}`);
 this.statements.forEach((statement) =>
 statement.emit()
 );
 emit(`  mov r0, #0`);
 emit(`  pop {fp, pc}`);
 }

 equals(other: AST) {…}
}
```

在前导和后导之间，我们按顺序发出内部语句，使用`forEach`循环。尽管如此，我们还没有定义可以在这里发出的任何语句节点。然而，我们的`Main`节点足以使我们的发射器通过第一次测试：一个什么也不做的程序成功！

当测试发射器通过时，我们可以单独进行：

```js
new Main([]).emit();
```

或者我们可以将其与解析器集成：

```js
parser.parseStringToCompletion(`
 function main() {

 }
`).emit();
```

我们可以将生成的汇编代码管道输入到文件中，进行汇编和执行！

恭喜，我们的编译器刚刚从头到尾编译了它的第一个程序！

## Assert

就像函数调用一样，断言将期望其唯一参数位于`r0`寄存器中。我们将其与`1`进行比较，我们在没有适当布尔值的无类型语言中将其视为真值。如果相等，我们将点（`.`）的 ASCII 码保存到`r0`中以表示成功。否则，我们将`F`的代码保存到`r0`以表示失败。我们从`libc`调用`putchar`来打印它。我们不终止程序。

```js
 cmp r0, #1
 moveq r0, #'.'
 movne r0, #'F'
 bl putchar
```

要实现`Assert`节点，我们只需将这段汇编复制粘贴到`emit`方法中。我们还确保发出条件，以便结果最终存储在`r0`中。我们还没有可以在这里发出的任何节点，但这种情况会很快改变。

```js
class Assert implements AST {
 constructor(public condition: AST) {}

 emit() {
 this.condition.emit();
 emit(`  cmp r0, #1`);
 emit(`  moveq r0, #'.'`);
 emit(`  movne r0, #'F'`);
 emit(`  bl putchar`);
 }

 equals(other: AST) {…}
}
```

## 数字

让我们从字面整数值的节点开始。这样，我们就可以尽快使用`Assert`。

我们需要将整数值加载到`r0`中。我们可以使用带立即数的`mov`指令，但立即数的范围相当有限。这就是为什么我们使用`ldr`伪指令。如您所记，汇编器如果可能，会将其转换为带立即数的`mov`指令。

```js
class Number implements AST {
 constructor(public value: number) {}

 emit() {
 emit(`  ldr r0, =${this.value}`);
 }

 equals(other: AST) {…}
}
```

现在，我们可以在断言中使用`Number`。

```js
parser.parseStringToCompletion(`
 function main() {
 assert(1);
 }
`).emit();
```

我们编译、汇编并运行这个程序，可以看到它打印了一个强大的点，表示断言通过。如果你将整数改为 0，你可以看到程序打印了 `F`，正如它应该的那样。

## 否定

下一个是否定操作符！它在某种程度上类似于 `Assert`。它接受一个项，我们发出它，并将其与零进行比较。根据这个结果，我们将 `1` 或 `0` 移入 `r0`，从而逻辑上否定结果。

```js
class Not implements AST {
 constructor(public term: AST) {}

 emit() {
 this.term.emit();
 emit(`  cmp r0, #0`);
 emit(`  moveq r0, #1`);
 emit(`  movne r0, #0`);
 }

 equals(other: AST) {…}
}
```

现在我们可以扩展我们的测试程序：

```js
function main() {
 assert(1);
 assert(!0);
}
```

从这里开始，当我们讨论测试程序时，我们将跳过样板代码。

## 中缀操作符

接下来是中缀操作符：`==`、`!=`、`+`、`-`、`*`、`/`，对应节点 `Equal`、`NotEqual`、`Add`、`Subtract`、`Multiply`、`Divide`。它们都非常相似：它们接受两个项 `left` 和 `right` 并对它们进行操作。

让我们以 `Add` 为例。

我们可以发出一个节点，将值移动到一个临时寄存器（比如说，`r1`），然后发出第二个节点（其值最终在 `r0` 中），然后将它们相加：

```js
class Add implements AST {
 constructor(public left: AST, public right: AST) {}

 emit() { /* First flawed attempt */
 this.left.emit();
 emit(`mov r1, r0`);
 this.right.emit();
 emit(`add r0, r1, r0`);
 }

 equals(other: AST) {…}
}
```

然而，这不会持续太久：一旦 `right` 节点比仅仅是 `Number` 更复杂，发出它很可能会覆盖存储 `left` 结果的 `r1`。例如，如果我们有两个嵌套的中缀节点，就会发生这种情况。这就是为什么在我们发出 `right` 之前需要将其保存到栈上，然后在我们执行加法之前恢复它：

```js
 emit() { /* Second attempt */
 this.left.emit();
 emit(`push {r0, ip}`);
 this.right.emit();
 emit(`pop {r1, ip}`);
 emit(`add r0, r1, r0`);
 }
```

为了保持 8 字节栈对齐，我们也保存和恢复虚拟 `ip` 寄存器。如果 `left` 节点是另一个嵌套表达式，推送和弹出将匹配，我们将在发出它之前和之后处于相同的栈指针位置。

就像在我们的第一次尝试中一样，我们可以避免在某些情况下使用栈。我们还可以做得比每次浪费栈空间来保存 `ip` 更好。

我们所有的中缀操作符都将具有相同的结构：发出右，推送，发出左，弹出，然后是动作。只有最后一部分会有所不同。所以，这里有一个快速表格，展示了每个中缀节点的动作部分。

每个中缀节点的动作部分

| AST 节点 | 发出 |
| --- | --- |
|  |  |
| `Add` |

```js
add r0, r1, r0
```



|  |  |
| --- | --- |
| `Subtract` |

```js
sub r0, r1, r0
```



|  |  |
| --- | --- |
| `Multiply` |

```js
mul r0, r1, r0
```



|  |  |
| --- | --- |
| `Divide` |

```js
udiv r0, r1, r0
```



|  |  |
| --- | --- |
| `Equal` |

```js
cmp r0, r1
moveq r0, #1
movne r0, #0
```



|  |  |
| --- | --- |
| `NotEqual` |

```js
cmp r0, r1
moveq r0, #0
movne r0, #1
```



|  |  |
| --- | --- |

如果你想要减少重复，你可以将公共部分拉入一个新的 AST 节点，称为 `Infix`，例如。

最后，我们可以为中缀操作符添加一个集成测试：

```js
function main() {
 assert(1);
 assert(!0);
 assert(42 == 4 + 2 * (12 - 2) + 3 * (5 + 1));
}
```

我们正在取得进展！

## 块语句

块语句类似于 `Main` 但没有所有入口点的麻烦。它只是发出每个语句。

```js
class Block implements AST {
 constructor(public statements: Array<AST>) {}

 emit() {
 this.statements.forEach((statement) =>
 statement.emit()
 );
 }

 equals(other: AST) {…}
}
```

简单测试：

```js
function main() {
 assert(1);
 assert(!0);
 assert(42 == 4 + 2 * (12 - 2) + 3 * (5 + 1));
 { /* Testing a block statement */
 assert(1);
 assert(1);
 }
}
```

## 函数调用

下一个是函数调用。正如我们在上一章所学，根据 ARM 调用约定，我们需要将前四个参数放入寄存器 `r0`–`r3`，并期望返回值在 `r0` 中。让我们提醒自己，`Call` 节点包含一个 `callee` 字符串和一个 `args` 数组。

```js
class Call implements AST {
 constructor(public callee: string,
 public args: Array<AST>) {}

 emit() {
 …
 }

 …
}
```

我们可以从一些简单的事情开始：调用不带参数的函数。这仅仅是分支和链接到调用者标签：

```js
 // One argument
 emit(`  bl ${this.callee}`);
```

那一个参数呢？如果我们发出第一个参数，它将在 `r0` 中。这正是我们需要它的地方！然后我们 `bl` 到它，就像之前一样：

```js
 // Two arguments
 this.args[0].emit();
 emit(`  bl ${this.callee}`);
```

更多参数吗？我们可以为两个、三个和四个参数设置特殊案例，这也有利于性能，但让我们直接跳到一个更通用的案例。以下将处理两个、三个或四个参数。对于零个或一个参数，我们没有使用任何栈空间，这真是太好了。然而，为了评估更多的参数，我们需要暂时将它们放在栈上。否则，当我们评估一个参数时，我们可能会破坏其他参数的值。以下是我们的做法：

```js
 // Up to four arguments
 emit(`  sub sp, sp, #16`);
 this.args.forEach((arg, i) => {
 arg.emit();
 emit(`  str r0, [sp, #${4 * i}]`);
 });
 emit(`  pop {r0, r1, r2, r3}`);
 emit(`  bl ${this.callee}`);
```

首先，我们为最多四个参数分配足够的栈空间，或者说，16 字节。我们通过从栈指针中减去来实现这一点，因为栈是从高内存地址向低内存地址增长的。然后我们使用 `forEach` 遍历参数，它接受一个带有每个项的回调，以及可选的索引 `i`。对于每个参数，我们首先发出它，然后将结果存储在每个栈槽中。我们乘以四，将数组索引 `0`、`1`、`2`、`3` 转换为栈偏移量（以字节为单位）：`0`、`4`、`8`、`12`。此时，所有参数都将被发出并存储在栈上。然而，我们希望它们在寄存器中。所以我们一次性将它们 `pop` 到寄存器中。现在，我们已经准备好了，可以跳转到标签。

这是一个结合了所有三种方法的完整版本，通过根据参数数量进行分派。

```js
class Call implements AST {
 constructor(public callee: string,
 public args: Array<AST>) {}
 emit() {
 let count = this.args.length;
 if (count === 0) {
 emit(`  bl ${this.callee}`);
 } else if (count === 1) {
 this.args[0].emit();
 emit(`  bl ${this.callee}`);
 } else if (count >= 2 && count <= 4) {
 emit(`  sub sp, sp, #16`);
 this.args.forEach((arg, i) => {
 arg.emit();
 emit(`  str r0, [sp, #${4 * i}]`);
 });
 emit(`  pop {r0, r1, r2, r3}`);
 emit(`  bl ${this.callee}`);
 } else {
 throw Error("More than 4 arguments: not supported");
 }
 }

 equals(other: AST) {…}
}
```

如果有超过四个参数，我们抛出错误，因为基线编译器不支持这种情况。

> **探索**
> 
> 通过为两个、三个和四个参数分别专门化代码生成器（就像我们对零个和单个参数所做的那样），你将能够生成更好的代码（例如，不要为两个参数分配四个槽位，就像我们现在所做的那样）。从上一章中，你可能已经对如何处理超过四个参数有了想法。

我们如何测试这个？我们还没有定义函数的方法。但我们可以使用一些 `libc` 函数来测试这个。

如你所记，`putchar` 只接受一个参数：让我们通过打印点（46）的 ASCII 码来使用它，作为我们的测试。

还有一个名为 `rand` 的函数，它不接受任何参数并返回一个随机数。这不是基于测试的最佳函数，但我们可以使用它，直到我们可以定义自己的函数。

```js
putchar(46);
assert(rand() != 42);
```

我找不到任何可移植的 `libc` 函数，它接受两个、三个或四个整数参数，所以我们必须等待测试。

> **探索**
> 
> 修改解析器，使其能够生成一个与字符 `'.'` 对应的 ASCII 码的 AST 节点，例如 `new Number(46)`。你可以使用 `string.charCodeAt(index)` 方法。

## 如果语句

下一个更有趣：条件语句，或者说，if 语句。它很有趣，因为我们终于可以处理控制流了。

让我们先看看一个例子。如果我们想编译以下 if 语句：

```js
if (rand()) {   /* Conditional */
 putchar(46);  /* Consequence */
} else {
 putchar(70);  /* Alternative */
}
```

我们如何手动编译这个呢？我们可以从两个标签`ifTrue`和`ifFalse`开始。如果条件为零，我们分支到`ifFalse`，否则分支到`ifTrue`。在每个标签下面，我们发出对应分支的代码。最后，我们放置一个`endIf`标签，并确保两个分支在到达末尾时都跳转到它。

```js
 bl rand
 cmp r0, #0
 beq ifFalse
 bne ifTrue

ifTrue:
 ldr r0, =46
 bl putchar
 b endIf

ifFalse:
 ldr r0, =70
 bl putchar
 b endIf

endIf:
 /* Whatever follows next */
```

我们可以做到这一点，但我们可以做一些改进，以减少指令的数量。首先，`ifFalse`分支以`b endIf`立即跟随`endIf:`标签结束。这意味着我们不需要跳转到它，因为它无论如何都指向下一个指令。类似地，对于`bne ifTrue`，但有一个细微差别：尽管这个分支是条件性的（`ne`后缀），但实际上它不是。因为两个条件`beq`和`bne`是互斥的，如果执行到达`bne ifTrue`，我们已经知道它将执行。此外，由于`bne ifTrue`是这个标签的唯一用途，我们可以完全删除`ifTrue`。以下是生成的代码：

```js
 bl rand
 cmp r0, #0
 beq ifFalse

 ldr r0, =46
 bl putchar
 b endIf

ifFalse:
 ldr r0, =70
 bl putchar

endIf:
 /* Whatever follows next */
```

它更短，分支更少，因此可以更有效地执行。但在我们直接在我们的发射器中实现它之前，我们需要解决一个问题。我们不能有两个不同的标签称为`ifFalse`或`endIf`：标签必须是唯一的。我们需要生成唯一的标签。

* * *

按照惯例，以前缀`.L`开始的标签，例如`.L123`，用于代码生成。让我们创建一个类来帮助我们生成这样的标签。

```js
class Label {
 static counter = 0;
 value: number;

 constructor() {
 this.value = Label.counter++;
 }

 toString() {
 return `.L${this.value}`;
 }
}
```

这个类有一个全局的`counter`，每次调用构造函数时都会增加。当创建一个`Label`对象时，这个计数器的值存储在对象的`value`实例变量中。我们定义了一个`toString`方法，添加了`.L`前缀，这样这些对象就可以很好地与字符串插值一起使用。

* * *

现在我们已经准备好实现发射器了。我们创建了两个标签对象`ifFalseLabel`和`endIfLabel`，并在`emit`调用中使用它们进行字符串插值。

```js
class If implements AST {
 constructor(public conditional: AST,
 public consequence: AST,
 public alternative: AST) {}

 emit() {
 let ifFalseLabel = new Label();
 let endIfLabel = new Label();
 this.conditional.emit();
 emit(`  cmp r0, #0`);
 emit(`  beq ${ifFalseLabel}`);
 this.consequence.emit();
 emit(`  b ${endIfLabel}`);
 emit(`${ifFalseLabel}:`);
 this.alternative.emit();
 emit(`${endIfLabel}:`);
 }

 equals(other: AST) {…}
}
```

现在所有我们的 if 语句都将有唯一的标签！

为了确保质量，我们可以添加一个小型单元测试：

```js
if (1) {
 assert(1);
} else {
 assert(0);
}

if (0) {
 assert(0);
} else {
 assert(1);
}
```

## 函数定义和变量查找

很好，我们可以在不定义函数的情况下开发和测试函数调用，但是接下来的两个概念高度交织：函数定义和变量查找。我们已经有了一个名为`Main`的特殊节点，可以将其视为一个不接受参数的函数，并且总是返回`0`。我们将使用它作为我们的`Function`节点发射器的模板：

```js
class Function implements AST {
 constructor(public name: string,
 public parameters: Array<string>,
 public body: AST) {}

 /* First attempt */
 emit() {
 if (this.parameters.length > 4)
 throw Error("More than 4 parameters: not supported");

 emit(``);
 emit(`.global ${this.name}`);
 emit(`${this.name}:`);
 this.emitPrologue();
 this.body.emit();
 this.emitEpilogue();
 }

 emitPrologue() {
 emit(`  push {fp, lr}`);
 emit(`  mov fp, sp`);
 emit(`  push {r0, r1, r2, r3}`);
 }

 emitEpilogue() {
 emit(`  mov sp, fp`);
 emit(`  mov r0, #0`);
 emit(`  pop {fp, pc}`);
 }

 equals() {…}
}
```

与`Call`类似，这次我们将限制自己使用四个参数。我们创建一个守卫来强制执行此操作，并在其他情况下引发错误。我们首先发出一个空行，以使汇编函数的可读性更好。类似于`Main`，我们发出`.global`指令和标签。然而，这次我们不是将其硬编码为`main`，而是使用函数的名称，使用字符串插值。

这里函数前导部分开始。我们将函数前导和后导部分拆分成独立的方法以提高可读性。这不是我们以前做过的事情。

让我们首先看看前导部分。我们保存帧指针和链接寄存器，并设置新的帧指针。我们预计参数将在寄存器`r0`–`r3`中。然而，一旦我们开始生成函数体，我们就有破坏它们的危险，因此我们需要将它们保存在堆栈上。幸运的是，ARM 允许我们使用单个`push`指令来完成它。（优化编译器会进行称为寄存器分配的分析，以确定哪些参数可以保留在寄存器中。）

接下来，我们生成函数体，然后是函数后导部分。在这里，我们撤销函数前导部分的影响：通过将堆栈指针值设置为帧指针值，我们实际上释放了我们在前导部分或其它地方分配的任何堆栈空间。我们将`r0`设置为我们的返回值，即`0`。这是为了模仿 JavaScript 函数在没有显式返回时返回`undefined`的事实。我们选择`0`因为它是一个假值，就像`undefined`一样。这也消除了返回垃圾值的危险，这些值可能是某些计算留下的遗留物，以防我们忘记编写`return`语句。作为最后一条指令，我们恢复调用者的帧指针，并将保存的链接寄存器弹出程序计数器，从而有效地从函数返回。

> **探索**
> 
> 我们将四个寄存器推入堆栈。一个改进的方法是将此代码专门化以更有效地处理参数较少的情况。如果你这样做，别忘了 8 字节堆栈对齐。

到目前为止，一切顺利。现在，我们如何在函数体中访问这些参数呢？它们位于从帧指针的已知偏移量处。由于堆栈是从高地址向低地址增长的，我们知道我们可以使用相对于帧指针的负偏移量来访问参数。我们还知道`push {r0, r1, r2, r3}`以相反的顺序推入寄存器，这与这个等效（但更庞大）的代码相同：

```js
 push {r3}
 push {r2}
 push {r1}
 push {r0}
```

这意味着第四个参数（从寄存器`r3`）将存储在位置`fp - 4`，第三个（`r2`）在`fp - 8`，第二个（`r1`）在`fp - 12`，第一个在`fp - 16`。

因此，当我们遇到一个引用参数之一的标识符时，我们只需要知道从帧指针到其值的偏移量。我们可以使用`ldr`来加载它。以下是一个加载第一个参数的示例：

```js
 ldr r0, [fp, #-16]
```

我们可以做到的是存储从参数名称到帧指针偏移量的映射，并确保将其传递给所有发射器。这意味着每当我们在映射中遇到标识符节点`Id`时，我们都可以查找偏移量，然后我们将知道如何加载它。这种映射通常被称为*环境*。

我们可以只是传递一个`Map<string, number>`，但我的水晶球告诉我，我们将在不久的将来需要扩展这个数据结构。所以，我们将引入一个具有这个映射作为实例变量`locals`的数据类：

```js
class Environment {
 /* Initial version */
 constructor(public locals: Map<string, number>) {}
}
```

我们称这个类为`Environment`。目前它只包含`locals`映射，这是我们的局部变量的环境，但它可以包含其他环境，例如全局变量环境、类型环境或具有函数签名的环境。我们称我们的映射为`locals`而不是`parameters`，因为我们预见我们将用它来存储其他局部变量，而不仅仅是函数参数。

现在，我们需要进行一项重大更改。我们需要调整`AST`接口以接受新的`Environment`参数，我们将将其传递给`emit`。我们将一致地调用此参数为`env`。

```js
interface AST {
 emit(env: Environment): void;
 equals(other: AST): boolean;
}
```

这也将要求我们修改所有 AST 节点以接受此环境并将其传递给所有其他节点发射器。这是一个非常侵入性的、但机械性的更改。我们不会遍历所有节点，让我们以`Add`为例。

```js
class Add implements AST {
 constructor(public left: AST,
 public right: AST) {}

 emit(env: Environment) {
 this.left.emit(env);
 emit(`  push {r0, ip}`);
 this.right.emit(env);
 emit(`  pop {r1, ip}`);
 emit(`  add r0, r1, r0`);
 }

 equals(other: AST) {…}
}
```

发射器现在接受一个`env`参数，并将其传递给其子 AST 节点：在这个例子中是`left`和`right`。

现在，回到我们的`Function`。我们需要修改它以符合新的`AST`接口和`Environment`。然而，我们将忽略传入的环境，因为我们不会支持嵌套函数。我们将它绑定到下划线，以向阅读此代码的读者表明这个变量是有意不使用的。以下是我们的新函数定义方法：

```js
class Function implements AST {
 …

 /* Second attempt */
 emit(_: Environment) {
 if (this.parameters.length > 4)
 throw Error("More than 4 parameters: not supported");

 emit(``);
 emit(`.global ${this.name}`);
 emit(`${this.name}:`);
 this.emitPrologue();
 let env = this.setUpEnvironment();
 this.body.emit(env);
 this.emitEpilogue();
 }
```

```js
 setUpEnvironment() {
 let locals = new Map();
 this.parameters.forEach((parameter, i) => {
 locals.set(parameter, 4 * i - 16);
 });
 return new Environment(locals);
 }
}
```

在这里，在函数定义节点中，我们知道参数的名称以及它们将被存储的偏移量。因此，这是创建新环境的理想位置。我们在`setUpEnvironment`方法中这样做。首先，我们创建一个空环境，然后使用`forEach`遍历参数，并将每个参数的本地映射设置为相对于帧指针的偏移量。在这里，`forEach`不仅传递项目，而且（可选地）传递每个项目的索引`i`。为了将每个索引映射到其帧指针偏移量，我们将索引乘以四，从单词转换为字节，并减去`16`，因为我们弹出序言中的四个参数时在堆上分配了这么多字节。我们在发出体时传递构建的环境，这将反过来将其传递给所有子节点，依此类推。

现在，一个需要访问局部参数的节点可以在环境中查找它。如果不是我们的代码中标识符的节点`Id`，那它还能是哪个节点呢？所有困难的工作都在`Function`节点中完成，所以我们的`Id`节点将会非常简单：

```js
class Id implements AST {
 constructor(public value: string) {}

 emit(env: Environment) {
 let offset = env.locals.get(this.value);
 if (offset) {
 emit(`  ldr r0, [fp, #${offset}]`);
 } else {
 throw Error(`Undefined variable: ${this.value}`);
 }
 }

 equals() {…}
}
```

它通过标识符名称查找本地环境中的偏移量。如果该名称不在环境中，它会抛出一个错误。这意味着在定义之前已经使用了变量。否则，它将使用相对于帧指针的`ldr`加载值。

这些帧指针很方便，不是吗？

现在我们已经实现了带有参数的函数，我们可以实现相当多的有趣函数！首先，我们可以去掉`Main`节点，并像任何其他函数一样生成`main`函数。其次，我们可以去掉我们的`Assert`原语，并实现它作为一个函数：

```js
function assert(x) {
 if (x) {
 putchar(46);
 } else {
 putchar(70);
 }
}
```

最后，测试我们是否正确实现了参数传递是很好的。这里有一个这样的测试：

```js
function assert1234(a, b, c, d) {
 assert(a == 1);
 assert(b == 2);
 assert(c == 3);
 assert(d == 4);
}
```

它可以从`main`中调用为`assert1234(1, 2, 3, 4)`。

现在感觉我们几乎可以实施任何东西；然而，仍然缺少一个关键的部分。

## 返回

函数必须能够返回值。即使是延续传递风格的支持者也会同意。多亏了帧指针，实现`Return`节点是容易的：

```js
class Return implements AST {
 constructor(public term: AST) {}

 emit(env) {
 this.term.emit(env);
 emit(`  mov sp, fp`);
 emit(`  pop {fp, pc}`);
 }

 equals(other: AST) {…}
}
```

我们只需要将节点（返回值）发出到`r0`，然后重复与结尾相同的指令。

现在，从技术角度讲，我们的编译器语言是图灵完备的。循环？你说？我们可以通过递归来实现循环，因为我们的函数正在使用栈。（我们并没有免费获得尾调用优化）。这一步需要广泛的测试，但我心中想的测试是这样的：

```js
function factorial(n) {
 if (n == 0) {
 return 1;
 } else {
 return n * factorial(n - 1);
 }
}
```

阶乘函数！我们可以从`main`中这样调用它：

```js
assert(factorial(5) == 120);
```

这是一个小小的测试套件，但对于我们的编译器来说是一个巨大的飞跃！

* * *

现在我们已经准备好了所有基本要素。我们将以一些小优点结束这一章：本地变量、赋值和 while 循环。作为奖励。

## 本地变量

在这里，我们将着手实现本地变量，就像 JavaScript 中使用`var`关键字声明的，并在我们的 AST 中表示为`Var`节点。

为什么是`var`而不是`let`？我们在这个编译器的实现中只使用了`let`！

让我们提醒一下它们之间的区别。这是一个使用`let`的函数：

```js
function f() {
 let x = 1;
 {
 let x = 2;
 }
 console.log(x);
}
```

使用`var`的相同例子：

```js
function f() {
 var x = 1;
 {
 var x = 2;
 }
 console.log(x);
}
```

第一个打印`1`，第二个打印`2`。区别在于`var`绑定在函数作用域内工作，而`let`绑定在块作用域内工作。所以这两个`var`指向相同的变量，但这两个`let`是不同的：第二个是在由大括号分隔的另一个作用域内。

我们实现`var`的原因是它简化了作用域处理：每个函数只需要一个作用域。这也可能是 JavaScript 首先引入`var`，而`let`则晚得多的原因。还有`const`，它与`let`类似，但禁止对其绑定进行赋值。

那么，我们如何实现`var`？我们可以将值压入栈中，然后在环境中查找它，就像参数一样。然而，当我们定义参数时，我们知道它们的偏移量：它们在帧的起始位置。但是，当我们发出一个`var`时，我们不知道当前栈有多深。但我们可以通过环境来跟踪这一点！

让我们修改`Environment`类以存储`nextLocalOffset`数字，它表示下一个可用的帧指针偏移量。这有时也被称为*栈索引*。

```js
class Environment {
 constructor(public locals: Map<string, number>,
 public nextLocalOffset: number) {}
}
```

首先，我们可以将其初始化为`0`。然而，在`Function`中，我们需要根据我们分配的参数数量来设置它。目前我们总是分配四个，偏移量为`-4`、`-8`、`-12`和`-16`。所以下一个可用的偏移量是`-20`。这就是我们在环境设置中使用的值：

```js
class Function implements AST {
 …
 setUpEnvironment() {
 let locals = new Map();
 this.parameters.forEach((parameter, i) => {
 locals.set(parameter, 4 * i - 16);
 });
 nextLocalOffset = -20;
 return new Environment(locals, nextLocalOffset);
 }
 …
}
```

现在，让我们看看`Var`节点。从理论上讲，我们可以将变量名添加到`local`环境映射中，使其映射到`nextLocalOffset`值，然后将值推送到栈上，并更新`nextLocalOffset`以指向下一个可用的偏移量。然而，我们需要保持 8 字节对齐，所以这稍微复杂一些：

```js
class Var implements AST {
 constructor(public name: string,
 public value: AST) {}

 emit(env: Environment) {
 this.value.emit(env);
 emit(`  push {r0, ip}`);
 env.locals.set(this.name, env.nextLocalOffset - 4);
 env.nextLocalOffset -= 8;
 }

 equals(other: AST) {…}
}
```

我们仍然从发出值并将其推送到栈上开始，但我们也需要推送一个类似`ip`寄存器的值以保持栈 8 字节对齐。这样，`ip`就会位于`nextLocalOffset`处，因此在更新本地环境时，我们从它减去另一个`4`字节。现在，我们需要将`nextLocalOffset`向前推进两个栈槽；换句话说，我们将其减去`8`。

> **探索**
> 
> 这浪费了本地变量栈空间的 50%。这里有一个修复方法：你可以在`Environment`中添加一个偏移量数组`vacantOffsets`。然后每次你需要栈槽时，首先检查是否有任何空槽，并尝试使用它（并从数组中移除）。只有当没有这样的槽时，你才分配新的栈空间。这种技术也可以用于利用其他情况下的过度分配，例如，当你有奇数个函数参数时。还有一种方法可以将所有这些组织得井井有条，放入一个`Frame`数据结构中，该结构在一个地方处理这种偏移量调整，这样每个相关的发射器就不必跟踪偏移量数学。

让我们添加一个断言来测试我们的`var`处理：

```js
var x = 4 + 2 * (12 - 2);
var y = 3 * (5 + 1);
var z = x + y;
assert(z == 42);
```

## 赋值

赋值处理与标识符查找非常相似。除了不读取其值，我们正在写入它。我们将值发出到`r0`。然后我们在本地环境中查找帧指针偏移量。如果环境查找失败，我们抛出一个错误：在那个时间点该变量未定义。否则，我们使用`str`指令将`r0`中的值存储到帧指针偏移量。

```js
class Assign implements AST {
 constructor(public name: string,
 public value: AST) {}

 emit(env: Environment) {
 this.value.emit(env);
 let offset = env.locals.get(this.name);
 if (offset) {
 emit(`  str r0, [fp, #${offset}]`);
 } else {
 throw Error(`Undefined variable: ${this.name}`);
 }
 }

 equals(other: AST) {…}
}
```

对赋值的简单测试：

```js
var a = 1;
assert(a == 1);
a = 0;
assert(a == 0);
```

> **探索**
> 
> 实现`const`绑定类似于`var`，但对其不允许赋值。实现这一点的办法是将局部映射更改包括不仅是一个偏移量，还有一个标志来表示绑定是否为常量。然后，当赋值查找时，检查常量标志是否未设置，否则失败。

## 当循环

当循环处理在很多方面与`If`语句处理相似。它有一个条件表达式，该表达式被检查为真值，但它只有一个分支。另一个区别是在那个分支的末尾，它会跳回开始。

```js
class While implements AST {
 constructor(public conditional: AST,
 public body: AST) {}

 emit(env: Environment) {
 let loopStart = new Label();
 let loopEnd = new Label();

 emit(`${loopStart}:`);
 this.conditional.emit(env);
 emit(`  cmp r0, #0`);
 emit(`  beq ${loopEnd}`);
 this.body.emit(env);
 emit(`  b ${loopStart}`);
 emit(`${loopEnd}:`);
 }

 equals(other: AST) {…}
}
```

在这里，我们生成了两个唯一的标签：`loopStart`和`loopEnd`。我们检查条件，如果条件为假，则跳转到`loopEnd`。然后是循环的主体。最后，我们无条件地跳转到`loopStart`标签。对`while`循环的一个快速测试可能看起来像这样：

```js
var i = 0;
while (i != 3) {
 i = i + 1;
}
assert(i == 3);
```

我们已经将`while`循环的处理放了一段时间，因为要测试它，我们首先需要赋值功能正常工作。现在，我们也可以实现一个使用`while`循环的`factorial`函数版本：

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

> **探索**
> 
> 你如何实现一个`for`循环？`for`循环可以被视为`while`循环的一种特殊情况。考虑以下`for`循环：
> 
> ```js
> for (i = 1; i < 5; i = i + 1) {
>  body();
> }
> ```
> 
> 等价于以下`while`循环：
> 
> ```js
> i = 1;
> while (i < 5) {
>  i = i + 1;
>  body();
> }
> ```

* * *

基线编译器已完成。现在，进入**第二部分**，我们将继续通过添加新功能来扩展和扩展编译器。

下一章：第二部分简介

* * *
