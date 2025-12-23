# V8 JavaScript 引擎内部

> 原文：[`www.thenodebook.com/node-arch/v8-engine-intro`](https://www.thenodebook.com/node-arch/v8-engine-intro)

ℹ️注意

关于本章的简要说明。我们故意进入了一些高级领域。现在我会介绍一些大概念，我们将在第三卷（第二十一章开始）中更详细地探讨。

将这视为一个快速概述——我们进展迅速，只关注最重要的点。为什么？因为首先对 v8 引擎和内存的工作原理有一个深入的理解是绝对必要的。这是所有后续课程的基础，将使所有课程都变得容易理解。

## TL;DR

让我们明确一点：**V8 既是解释器又是多级 JIT 编译器**。当你的 Node.js 应用程序运行时，你的 JavaScript 首先由 **Ignition** 解释，将其编译成字节码并直接执行。随着你的代码逐渐热身，它将通过一个复杂的 4 级编译流程：**Sparkplug**（快速基线编译器）→ **Maglev**（中级优化编译器）→ **TurboFan**（高级优化编译器）。每一级都提供了越来越好的性能，但代价是编译时间。对于重要的代码（你的“热点路径”），最终由 TurboFan 编译成高度优化的、推测性的机器代码，其运行速度几乎与 C++ 相当。这个复杂的多级流程在快速启动和卓越的峰值性能之间取得了平衡。

ℹ️注意

V8 既是解释器又是多级 JIT 编译器。理解这个复杂的编译流程对于性能优化至关重要。

V8 整个性能模型依赖于一个关键假设：你的代码是可预测的。V8 通过内部机制 **Hidden Classes**（或称为形状）对你的对象形状和变量类型进行下注。它通过这种机制为这些隐藏类生成专门的机器代码。这非常快——只要假设成立。

当你的代码打破了这些假设——通过创建具有不同属性顺序的对象、在数组中混合类型或使用动态模式——你会触发性能悬崖。V8 被迫在称为**去优化**的过程中丢弃优化的机器代码，并回退到字节码解释器或更低的编译层级。去优化通常会导致从 2 倍到 20 倍的性能下降，具体取决于上下文。在紧密循环或极端热点路径中，惩罚可能会更严重（可能是 100 倍或更多），但 100 倍的减速是罕见的边缘情况。理解这个概念——V8 奖励可预测、稳定的对象形状，并惩罚动态、不可预测的形状——是解锁一致、高性能 Node.js 应用程序的关键。专注于编写“无聊”的单态 JavaScript，让优化编译器保持高兴。

## 100 倍减速之谜

它就像往常一样，从一个完全合理的代码片段开始。我们有一个 API 端点，负责处理配置对象。它会取一个基本配置，添加一些用户特定的覆盖，然后可能是一些请求特定的设置。很简单的事情。数月来，它一直运行得毫无故障，每个请求大约 2-5 毫秒。

然后延迟警报开始尖叫。P99 延迟飙升至 200 毫秒以上。不是 20 毫秒。*两百毫秒*。100 倍的减速。我们以为这是网络问题，数据库瓶颈，任何不是应用程序代码的问题。代码简单、干净、正确。可能出什么问题了？

我们部署了一个热修复来添加更多的日志。没有效果。我们深入 CPU 分析器，火焰图一片混乱。它们并没有指向一个单一的罪魁祸首函数；相反，整个请求处理器只是……很慢。就好像 CPU 决定只以正常速度的 1/100 来运行，但只针对这个特定的端点。

导致这种情况的变化是无害的。一个新功能要求我们向配置对象添加一个动态的、可选的属性。一行代码：`if (condition) { config.optionalFeature = true; }`。就是这样。一行代码。一个属性的增加。

我们面对的是一个幽灵。代码在“热点路径”上逻辑上完全相同，但速度慢了几个数量级。这是我关于 JavaScript 性能的第一个真正教训。这是你意识到你写的代码并不是运行的代码的时刻。你并不是在为简单的解释器编写指令；你是在为超级积极的优化编译器提供建议。而我们，完全意外地，以最糟糕的方式冒犯了那个编译器。

我们以前把 JavaScript 对象当作方便的哈希表来处理，随时添加属性。我们像大多数开发者一样假设，“JavaScript 就是 JavaScript，对吧？”它是一种动态语言，这就是你使用它的方式。但在我们的代码下面，V8 引擎已经就我们的 `config` 对象的结构做出了一系列假设。它基于这个结构生成了高度专业化的机器代码。而我们添加的一个无辜的属性就使所有这些假设失效，迫使 V8 扔掉所有辛勤的工作，并回退到一个显著更慢的执行路径。那个从 2ms 提升到 200ms 的函数教会了我们一个任何教科书都无法传授的教训：你不仅仅是为其他开发者编写 JavaScript。你是为 V8 编写它。

* * *

## V8 实际上是怎样执行 JavaScript 的

我们大多数人都有一个简单的心理模型：JavaScript 是一种解释型语言。你编写代码，引擎逐行读取它，并执行其指示。这种模型不仅错误，如果你关心性能，它甚至是非常危险的。V8 并不是以传统方式解释你的代码。它通过一个复杂的分层管道编译你的代码。

从你的 `.js` 文件到执行机器代码的过程是一个多阶段管道，设计目的是为了速度。具体来说，它设计用于快速启动（不要花费太多时间编译）以及频繁运行代码的惊人峰值性能。这是即时编译器（**Just-In-Time (JIT)**）的核心思想。

这里是高级流程：

1.  V8 首先要做的是解析你的原始 JavaScript 源代码。它还没有开始执行任何操作。目标是把字符序列转换成机器可以理解的有序表示。这个过程涉及两个关键组件：

    +   **扫描器**：将代码标记化，将其分解成原子片段，如 `const`、`myVar`、`=`、`10`、`;`。

    +   **解析器**：接收标记流并构建一个 **抽象语法树 (AST)**。AST 是一种树形数据结构，它表示了你的代码的语法结构。一个像 `const a = 10;` 的表达式会变成一个具有 `VariableDeclaration` 节点的树，该节点有一个标识符（`a`）子节点和一个值（`10`）子节点。

1.  一旦构建了抽象语法树（AST），V8 的解释器，名为 **Ignition**，就开始工作。Ignition 遍历 AST 并生成 **字节码**。字节码是一组低级、平台无关的指令。它还不是机器代码，但它比原始 JavaScript 更接近底层。例如，一个加法操作 `a + b` 可能会变成类似 `Ldar a`（使用 `a` 的值加载累加器），`Add b`（将 `b` 的值加到累加器）的字节码指令。Ignition 然后可以直接执行这些字节码。对于只运行一次的代码，这通常是故事的开始和结束。它比简单的解释器更快，因为它不必每次都重新解析源代码。

1.  当 Ignition 正在执行字节码时，它也在收集剖析数据。它在观察你的代码运行。这个函数被调用了多少次？传递给它的值类型是什么？使用了哪些对象形状？这种反馈对于下一步至关重要。它就像前线侦察兵向将军汇报，告诉他们应该在哪里集中精力。

1.  当 Ignition 的剖析器识别出一段代码为“热”代码（即，它被执行得相对频繁）时，它会被转交给**Sparkplug**，基线编译器。Sparkplug 从 Ignition 接收字节码，并直接编译成机器代码，而不进行任何优化。这比解释字节码更快，但没有高级分析的开销。Sparkplug 平滑了解释和优化代码之间的性能悬崖。它于 2021 年推出，以填补编译管道中的关键差距。

1.  随着函数变得越来越热门并成为“热门”函数（执行频率更高，并伴随一致的类型反馈），它们被提升到**磁悬浮列车**，这是 Chrome M117（2023 年 12 月）中引入的中端优化编译器。磁悬浮列车代表了编译管道中的最佳平衡点。它接收来自 Ignition 的字节码和类型反馈，并使用静态单赋值（SSA）形式和控制流图生成优化的机器代码。磁悬浮的哲学是“足够好的代码，足够快”——它的编译速度比 Sparkplug 慢约 10 倍，但比 TurboFan 快约 10 倍，生成的代码比基线代码显著更好，但没有全面的重型优化。磁悬浮基于类型反馈进行推测性优化，但比 TurboFan 的假设更为保守。

1.  最后，对于执行了数千次且类型反馈非常稳定的绝对热门代码路径，V8 将函数提升到**TurboFan**，高级优化编译器。TurboFan 从 Ignition 接收字节码和丰富的剖析反馈，并进行激进的**推测性优化**。它说：“好吧，这个函数已经被调用了 10,000 次，而且每次传递给参数`x`的值都是数字。我要赌它将**总是**是数字。”基于这个赌注，它生成针对这个假设的高度优化、低级的机器代码。这种机器代码可以绕过解释器中的许多检查和开销，从而实现巨大的速度提升。

1.  如果优化编译器的任何赌注都失败了怎么办？如果在第 10,001 次调用时，你传递了一个字符串而不是一个数字给那个函数呢？V8 检测到这个假设违反，立即丢弃优化的机器代码，并无缝地将执行切换回较低层级——无论是磁悬浮列车、Sparkplug，还是直接回到 Ignition 的字节码。这被称为**去优化**。它确保了正确性，但这也带来了性能成本。如果这种情况反复发生，你的应用程序将陷入优化和去优化的致命循环，破坏你的性能。

让我们在这里澄清一个常见的误解。

### 误解 1：“V8 只是一个解释器”

这在本质上是不真实的。虽然它确实有一个解释器（Ignition），但其主要目标是通过对多层级编译管道的优化，得到优化的机器码。Ignition 产生的字节码是一个实现细节，是通往优化编译器的垫脚石。

下面是现代 4 层过程的简单图示：

这个管道是 V8 的核心。作为一位注重性能的工程师，你的任务是编写能够顺畅地流经层级到 TurboFan 并停留在那里的代码。每次你的代码导致去优化，你都在迫使 V8 爬回一个较慢的层级，性能惩罚可能非常巨大。

* * *

## 编译管道：从 Ignition 到 TurboFan

让我们放大查看这个管道。理解每个层级中的进展至关重要，因为所有性能魔法——以及所有性能悬崖——都发生在这里。

所有这一切都始于解析之后。现在 AST 已经在内存中。

### 点火器的作用：基准性能

Ignition 的主要任务是让你的代码快速运行。完整的优化是一个重过程；它消耗 CPU 和内存。对于可能在应用程序启动期间只运行一次的代码，启动一个庞大的编译器将是浪费的。

因此，Ignition 遍历 AST 并生成字节码。这是一个一对一的过程；这里没有复杂的优化发生。这种字节码的设计非常迷人。它是一个基于寄存器的机器，而不是基于堆栈的，这减少了所需的指令数量，并且更好地与真实 CPU 的架构相匹配。

在执行这个字节码的过程中，Ignition 收集反馈。它收集的主要数据称为**类型反馈**。对于你代码中的每个操作（如属性访问`obj.x`或加法`a + b`），V8 都会创建一个**反馈向量**槽。随着代码的运行，Ignition 会填充这个向量中的观察到的类型。

例如，对于代码`function add(a, b) { return a + b; }`：

+   Ignition 看到了`add(1, 2)`。它在反馈向量中记录：“看到了`a`作为小整数，`b`作为小整数，操作结果为小整数。”

+   它看到了`add(10, 20)`。反馈是一致的。

+   函数被以整数调用 100 次。现在反馈向量对涉及的类型非常有信心。

这条反馈是所有优化编译器的燃料。没有它，它们就会盲目飞行。

### Sparkplug：快速基准

**Sparkplug**是 2021 年引入的基准编译器，作为第一个优化层级。它将 Ignition 的字节码直接编译成机器码，没有任何优化或类型专门化。这比解释字节码快，但没有高级分析的开销。

Sparkplug 背后的关键洞察是，即使是未优化的机器码通常也比解释的字节码要好。它提供了从 Ignition 到更激进的优化编译器的平滑过渡，减少了曾经存在的性能悬崖。

### Maglev：中层的优化器

**Maglev**是 V8 编译管道中的最新成员，于 Chrome M117（2023 年 12 月）引入。它填补了 Sparkplug 的快速但未优化的代码和 TurboFan 的慢速编译但高度优化的代码之间的关键差距。

Maglev 的设计理念是“足够好的代码，足够快”。以下是它特殊之处 -

1.  与 Sparkplug 直接将字节码转换为机器码的翻译不同，Maglev 通过控制流图构建了适当的静态单赋值（SSA）形式表示。这使得它能够在保持编译时间合理的同时执行有意义的优化。

1.  它使用 Ignition 的类型反馈来生成专用代码。如果一个函数总是接收整数，Maglev 将生成特定于整数的机器码。但它的侵略性不如 TurboFan - 它做出更安全的赌注，不太可能需要去优化。

1.  它的编译速度大约比 Sparkplug 慢 10 倍，但比 TurboFan 快 10 倍。这使得它非常适合那些足够热以从优化中受益但又不那么关键以至于需要 TurboFan 的重型处理的代码。

1.  其中一个令人惊讶的好处是减少了能耗。通过快速提供足够的优化，它防止 CPU 在等待 TurboFan 完成编译时在效率较低的代码中空转。

1.  Maglev 作为一个试验场。在 Maglev 中表现良好的代码，具有稳定的类型反馈，成为 TurboFan 优化的强有力候选者。

下面是现代的四层编译管道 -

### TurboFan 的触发器：热度计数器

V8 如何决定何时进行优化？它使用计数器和启发式算法的组合。每次函数执行时，V8 可能会增加其热度计数器。当这个计数器达到某些阈值时，V8 将函数提升到下一级。

阈值不是固定的 - V8 根据各种因素进行调整：

+   循环迭代次数比函数调用多

+   具有稳定类型反馈的函数被更积极地提升

+   可用的 CPU 资源影响提升决策

一旦标记为 TurboFan 优化，**编译任务**将被发送到后台线程。这很重要：V8 不会阻塞你的主应用程序线程来编译你的代码。它并行处理繁重的工作，确保你的应用程序保持响应。

### TurboFan 的推测性优化

TurboFan 现在有三个东西：来自 Ignition 的**字节码**、丰富的**类型反馈**，以及可能来自 Maglev 的**优化代码**进行分析。它开始工作，这是一门关于推测的精湛课程。

TurboFan 不直接与字节码工作。它将其转换为称为“**节点海洋**”的内部图表示。这个图使得应用高级编译器优化（如冗余/死代码消除、[常量折叠](https://en.wikipedia.org/wiki/Constant_folding)、[循环展开](https://en.wikipedia.org/wiki/Loop_unrolling)）变得更容易。

TurboFan 会查看反馈向量并做出最激进的猜测。“对于 `obj.x` 的反馈表明它总是看到相同的对象形状。我将生成假设这个形状的机器代码。”这意味着它可以直接生成内存访问：`mov rax, [rbx + 0x18]`，而不是一个通用的“在这个对象上查找属性 `x`”操作（这涉及到哈希表查找）。这是缓慢的多步骤过程和单个快速 CPU 指令之间的区别。

ℹ️注意

如果你之前没有见过汇编代码，不要担心。这条指令是直接从精确内存位置获取数据的命令。它使用对象的位置（rbx）加上一个固定偏移量（+ 0x18）来计算地址，完全跳过了缓慢的属性查找。

如果一个热门函数 `foo()` 调用了另一个函数 `bar()`，TurboFan 可能会决定 **内联** `bar()`。它本质上是将 `bar()` 的机器代码直接复制到 `foo()` 的代码中，消除了函数调用的开销。这是 V8 执行的最强大的优化之一。

如果一个函数在变得热门时已经开始运行呢？想象一下一个长时间运行的 `for` 循环。函数被调用了一次，但循环已经进行了数百万次迭代。V8 不能等待函数返回来交换优化后的代码。这就是 **栈上替换**（OSR）发挥作用的地方。当循环在慢速的 Ignition 字节码中运行时，TurboFan 在后台编译一个优化版本。一旦准备好，V8 可以在循环的中间暂停执行，用新的优化帧替换栈上的执行帧，然后在快速机器码中继续执行。

从 TurboFan 获得的是一个基于 Ignition 观察到的行为高度优化的机器代码块。这段代码非常快，但有一个问题：它完全依赖于那些早期观察结果的真实性。

## 隐藏类

**这就是关键**。如果你只想了解关于 V8 内部的一个知识点，那就让它成为这个。**隐藏类**（在 V8 的源代码中也称为“形状”或“映射”）是 V8 用于在 JavaScript 对象上实现快速属性访问的机制。它们是所有优化编译器构建优化基础的部分。

让我们打破另一个误区。

### 误区 2: “JavaScript 对象就像哈希表”

在你的脑海中，你可能将 JavaScript 对象想象成一个字典或哈希表，其中属性名映射到值。虽然这是 *逻辑* 模型，但不是 V8 的物理现实。哈希表查找相对较慢。为了使属性访问快速，V8 假设 JavaScript 有类。

当你创建一个对象时，V8 在幕后为它创建一个 **隐藏类**。这个隐藏类存储有关对象形状的元信息，特别是其属性布局。

让我们看看它是如何发生的。

```js
// This code lives in examples/v8/hidden-classes-demo.js
 // Run with: node --allow-natives-syntax hidden-classes-demo.js
const obj1 = {};
console.log(%HaveSameMap(obj1, {})); // true. Both share the initial hidden class.
 // Adding a property creates a new hidden class and a transition.
obj1.x = 1;
 const obj2 = {};
console.log(%HaveSameMap(obj1, obj2)); // false. obj1's hidden class has changed.
 obj2.x = 5;
// Now obj2 has followed the same transition path as obj1.
console.log(%HaveSameMap(obj1, obj2)); // true. They now share the same hidden class again.
```

🚨 警告

V8 内置函数（如 `%HaveSameMap`）是内部、不受支持的 API，它们在 V8 版本之间会发生变化。仅在使用 `--allow-natives-syntax` 进行实验时使用它们。切勿在生产代码中使用。

### 转换树

V8 并不仅仅为每个可能的对象形状创建一个新的隐藏类。它创建 **转换链**。

当你创建 `const p1 = {}` 时，`p1` 获得一个基本隐藏类，让我们称它为 `C0`。

当你添加 `p1.x = 5` 时，V8 会检查是否存在从 `C0` 到属性 `x` 的转换。如果没有，它会创建一个新的隐藏类 `C1` 并记录一个转换：`C0 + 'x' => C1`。隐藏类 `C1` 现在知道具有其形状的对象在内存中具有一个特定偏移量的属性 `x`。`p1` 的隐藏类被更新为 `C1`。

当你添加 `p1.y = 10` 时，它做的是同样的事情：它创建 `C2` 和一个转换：`C1 + 'y' => C2`。

现在，如果你创建另一个对象，`const p2 = {}`，它从 `C0` 开始。如果你然后添加 `p2.x = 15`，V8 会看到现有的转换，并简单地将 `p2` 移动到隐藏类 `C1`。如果你然后添加 `p2.y = 20`，它将移动到 `C2`。现在，`p1` 和 `p2` 拥有 **完全相同的隐藏类**。V8 可以优化任何操作这些对象的代码，因为它知道它们具有相同的内存布局。

这是关键。如果你创建 `const p3 = {}` 并以不同的顺序添加属性？`p3.y = 1;` `p3.x = 2;`

这创建了一个完全不同的转换路径！`C0 + 'y' => C3` `C3 + 'x' => C4`

即使 `p2` 和 `p3` 具有相同的属性，它们却拥有 **不同的隐藏类**。对 V8 来说，它们是根本不同的形状。

### 我们遇到的“配置对象”灾难

这正是我们在 100 倍减速之谜中遇到的问题。我们的代码看起来像这样：

```js
function createConfig(base, userOverrides, requestParams) {
 let config = { ...base }; // Start with a consistent shape
 // User overrides could have any number of properties in any order
 for (const key in userOverrides) {
 config[key] = userOverrides[key];
 }
 // This was the killer: a conditional property
 if (requestParams.useNewFeature) {
 config.optionalFeature = true; // This addition forks the hidden class tree
 }
 return config;
}
```

由于 `userOverrides` 添加键的顺序不可预测，而 `optionalFeature` 只有时存在，我们为逻辑上相同的对象创建了数十个，有时甚至数百个不同的隐藏类。当优化编译器（TurboFan）尝试优化使用这些 `config` 对象的函数时，它们看到了混乱。它们无法做出任何可靠的猜测，因此只能放弃。或者更糟，它们会针对一个形状进行优化，然后在看到另一个形状时立即降级。

这里是如何在我们的配置对象中发生隐藏类分支的：

对于我们的热点路径配置对象，修复方法是欺骗性地简单：**预先初始化频繁访问的属性**，即使使用 `null` 或 `undefined`。

```js
function createConfigV2(base, userOverrides, requestParams) {
 // Initialize a stable, predictable shape. All possible keys are present.
 let config = {
 ...base,
 settingA: null,
 settingB: null,
 optionalFeature: false, // Always present!
 };
 // Now, we are just *updating* properties, not adding them.
 // This does not change the hidden class.
 for (const key in userOverrides) {
 if (key in config) {
 config[key] = userOverrides[key];
 }
 }
 if (requestParams.useNewFeature) {
 config.optionalFeature = true;
 }
 return config;
}
```

通过创建一个稳定的初始形状，我们确保了几乎所有我们的配置对象都共享相同的隐藏类。这使得优化编译器能够为它们生成高度优化的代码，并且我们的延迟从 200ms 降至 2ms。我们学到了一个残酷的教训：**属性添加顺序不仅仅是一个风格选择，它是一个关键的性能决定因素**。

🚨注意

预先初始化所有属性可以稳定形状，但可能会浪费大对象的内存。仅对性能关键的对象在热点路径上这样做。对于通用对象，优先考虑可读性和可维护性。

## 内联缓存和多态性

因此，V8 使用 **隐藏类** 为你的对象创建一个秘密蓝图。这就是“是什么”。现在来说“如何”：V8 如何实际 *使用* 这个蓝图让你的代码飞快运行？答案是运行时优化的一件杰作，称为 **内联缓存 (IC)**。这是将隐藏类的理论连接到实际速度的机制。

首先，让我们确定一个关键术语：**调用点**。调用点不是一个抽象的概念；它是在你的代码中发生动态操作的一个实际的、物理的位置。

```js
function getX(point) {
 return point.x; // <-- THIS is the call site for the '.x' property access.
}
 getX({ x: 10, y: 20 }); // Execution hits the call site inside getX
getX({ x: 30, y: 40 }); // Execution hits the SAME call site again
```

这行代码 `return point.x;` 是一个单一、独特的调用点。无论你传递什么对象给 `getX`，V8 总是执行相同的这一行来访问 `x` 属性。这个特定的位置就是 V8 优化的地方。

将 IC 想象成有针对性的肌肉记忆。第一次你在特定的调用点上访问 `obj.x`，V8 必须进行慢速、痛苦的查找：

1.  获取 `obj` 的隐藏类。

1.  检查那个隐藏类的元数据以获取属性 `x` 的内存偏移量。

1.  跳转到对象内存中的那个偏移量并获取值。

在完成所有这些工作后，V8 并不会忘记。它会在那个确切的调用点上重写一小段机器代码的存根。这个存根就是内联缓存。下次执行那行代码时，IC 会进行一次单一、超级快速的检查：**“这个新对象的隐藏类和我上次看到的是否相同？”** 如果答案是肯定的，它将完全跳过慢速查找，并使用缓存的偏移量直接访问属性。动态查找现在变得和 C++ 中的直接内存访问一样快。这是一个巨大的胜利。

这将我们引向 IC 可以处于的不同“模式”或状态。这是术语变得重要的地方：

1.  **未初始化 -** 在执行那行代码之前，IC 是一张白纸。

1.  **单态型 -** 这是黄金状态。在这个调用点上，IC 只看到了 **一个** 隐藏类。`Mono` 意味着一个。这是可能的最快状态，因为 V8 可以生成针对这个单一形状的超专用机器代码。

1.  **多态型 -** IC 看到了少数不同的隐藏类（通常是 2 到 4 个）。`Poly` 意味着多个。V8 仍然可以处理这种情况，但速度较慢。它必须添加检查：“这是形状 A 吗？如果是，使用偏移量 X。这是形状 B 吗？如果是，使用偏移量 Y？”这仍然比完整的动态查找要快。

1.  **巨形态 -** IC 已经看到了太多不同的隐藏类。`Mega`意味着巨大。在这个时候，V8 放弃了。IC 被认为是“污染的”，V8 退回到慢速的通用查找。性能直线下降。

这就是 IC 如何通过这些状态转换的 -

当 IC 是多态的，它实际上是在内部运行一系列这样的检查 -

### 一个小例子

谈话是廉价的。让我们看看这个性能差异在真实基准测试中实际上看起来是什么样子。这种模式在数据处理中非常常见。

```js
// This code lives in examples/v8/monomorphic-patterns.js
// Run with: node --allow-natives-syntax monomorphic-patterns.js
 const ITERATIONS = 10000000;
 class Point2D {
 constructor(x, y) {
 this.x = x;
 this.y = y;
 }
}
 class Point3D {
 constructor(x, y, z) {
 this.x = x;
 this.y = y;
 this.z = z;
 }
}
 // This function's call site is MONOMORPHIC. It will always see Point2D objects.
// V8 will heavily optimize this.
function getX_Monomorphic(point) {
 return point.x;
}
 // This function's call site is POLYMORPHIC. It will see two different shapes.
// V8 can handle this, but it's slower.
function getX_Polymorphic(point) {
 return point.x;
}
 // ========= BENCHMARK =============
 // Prime the functions so V8 can optimize them
for (let i = 0; i < 1000; i++) {
 getX_Monomorphic(new Point2D(i, i));
 // Pass both shapes to the polymorphic function
 getX_Polymorphic(new Point2D(i, i));
 getX_Polymorphic(new Point3D(i, i, i));
}
 console.time("Monomorphic");
let mono_sum = 0;
for (let i = 0; i < ITERATIONS; i++) {
 mono_sum += getX_Monomorphic(new Point2D(i, i));
}
console.timeEnd("Monomorphic");
 console.time("Polymorphic");
let poly_sum = 0;
for (let i = 0; i < ITERATIONS; i++) {
 // Alternate between the two shapes
 const point = i % 2 === 0 ? new Point2D(i, i) : new Point3D(i, i, i);
 poly_sum += getX_Polymorphic(point);
}
console.timeEnd("Polymorphic");
 // Note: Ensure sums are used to prevent V8 from optimizing away the loops.
console.log(mono_sum, poly_sum);
```

在我的机器上运行 Node.js v23 时，结果很明显（你的结果可能会有所不同）：`单态：16.05ms` `多态：47.23ms`

多态版本几乎**慢 3 倍**。而且这仅仅是**两种**形状。想象一个接收来自五个不同来源的对象的函数，每个对象的结构略有不同。那个调用点将变成巨形态，性能惩罚可能会高达 10-50 倍。

目标是编写操作可预测数据结构的函数。当你看到一个可以接收“用户对象或公司对象”的函数时，你的性能感觉应该会发痒。可能更好的做法是拥有两个独立的、单态的函数：`processUser(user)`和`processCompany(company)`。无聊、重复的代码通常是最快的代码。

## 去优化 - 当 V8 放弃

去优化是 V8 的紧急弹出按钮。它是丢弃快速优化的机器代码并退回到较低编译层的过程。毫无疑问，它是 Node.js 应用程序中神秘性能问题的最大原因。

记住，来自磁悬浮和 TurboFan 的优化机器代码是**推测性的**。它建立在 Ignition 收集的一堆假设（类型反馈）之上。每当这些假设之一在运行时被证明是错误的，就会发生去优化。

### 常见去优化触发器

V8 会因许多原因去优化，但某些原因比其他原因更常见。

1.  **隐藏类不匹配 -** 这是最大的问题。TurboFan 编译了一个函数，假设它将始终看到具有隐藏类`C2`的对象。突然，一个具有`C4`的对象到达了。专门的机器代码现在无效。**退出！**

1.  **改变数组元素类型 -** V8 跟踪数组中元素的“类型”。它是所有小整数？所有双精度浮点数？所有对象？它根据这个来优化。如果你有一个所有元素都是小整数的数组 `[1, 2, 3]`（打包的小整数，最快的类型），你突然推入一个字符串 `arr.push('a')`，你就强制了一个存储转换。V8 动态升级数组以处理新的元素类型，任何假设整数数组的优化代码可能需要去优化。并非所有转换都会导致持续的减速 - V8 高效地处理了许多转换 - 但在大型数组的热点循环中，这些转换可能会影响性能。

1.  **`try...catch` 块（历史问题） -** 在较老的 V8 版本中，这个问题很成问题，因为处理异常所需的机制可能会干扰 TurboFan 的优化。现代 V8（Node v16+）在很大程度上解决了这个问题。在当前的 Node 版本（v22-24）中，`try...catch`在大多数情况下对性能的影响最小。为了正确性，可以自由地使用错误处理——避免`try...catch`的旧建议已经过时。

💡提示

现代 V8（Node 16+）对`try...catch`块进行了合理的优化。为了正确性，可以自由地使用它们；只有在考虑重构出极端热路径之前，才需要进行分析。

1.  **使用 `arguments` 对象 -** `arguments` 对象是一个经典的去优化器。它是一个“类似数组”的对象，但它不是一个真正的数组，并且有一些魔法属性，这使得编译器难以推理。特别是在 Node 的较老版本中使用它，是保证降低性能的一种方式。现代 V8 更好，但一个泄漏`arguments`的函数仍然可能存在问题。剩余参数（`...args`）几乎总是更好的、优化友好的选择。

1.  **`delete` 关键字 -** 虽然 V8 多年来对其`delete`的处理有了改进，但它仍然会降低热对象的性能。使用`delete obj.x`可以强制 V8 将对象切换到“字典模式”，其中属性存储在较慢的类似哈希表的结构中。如果你只需要在热路径上清除一个值，请使用`obj.x = undefined`。然而，如果你需要属性真正消失以确保正确性（例如，用于键枚举或`in`运算符），则`delete`仍然是正确的选择——只是避免在性能关键路径上使用它。

📌重要

避免在热对象上使用`delete`，但请记住，设置为`undefined`在语义上并不等价。设置为`undefined`的属性仍然存在于`Object.keys()`、`in`运算符和迭代中。

### 被 BigInt 去优化所惊吓

我们正在为我们的交易平台运行一个高性能的交易模拟服务。它监控着交易池，每秒模拟数千笔交易以抢占有利可图的机会。该服务会迅速启动，速度极快，但在一个小时内，其模拟的 TPS 会急剧下降，直到错过了每一个套利窗口。重启是唯一的解决办法，但缓解只是暂时的。

我们一直在头破血流。最后，我们使用一些标志附加了 V8 检查器：`node --trace-opt --trace-deopt simulator.js`。日志如暴雨般倾泻而下，但有一行信息对我们大声疾呼，重复了数千次：

`[去优化：开始 0x... <validateTransaction> (opt #3) 在字节码偏移量 68，原因=意外的 BigInt]`

`validateTransaction`函数——我们最热的代码路径——被 TurboFan 优化得非常漂亮，但随即被立即丢弃。一次又一次。快速查看对应于该字节码偏移量的代码，指向了我们计算代币兑换潜在[滑点](https://en.wikipedia.org/wiki/Slippage_(finance))的那一行。

问题是什么？我们处理的大多数交易——标准的 ETH 转账、小型的 DEX 兑换——其价值都能完美地适应标准的 JavaScript `Number`。但时不时地，某个“鲸鱼”会转移大量高精度代币（比如 SHIB）。这些数值如此之大，只能用`BigInt`来表示。

TurboFan 已经看到了数百万个`Number`类型，并对其下了重注。它为快速浮点运算生成了超优化的机器代码。当`BigInt`出现时，JIT 的世界观崩溃。**退出！**函数去优化。几千笔交易后，V8 会再次尝试，重新优化为`Number`。然后另一个鲸鱼转账会到来。重复上述过程。

这是野外的“去优化循环死亡”：

* * *

#### 解决方案：强制执行类型一致性

我们的第一直觉是标准化数据。我们知道我们不能简单地将所有内容转换为`BigInt`并保留浮点数数学，因为这会立即抛出一个`TypeError`。

```js
// This is what you CAN'T do
const value = BigInt(rawTx.value); // e.g., 100000000000000000000n
const slippage = value * 0.005; // THROWS: TypeError: Cannot mix BigInt and other types
```

解决方案必须是全面的。我们选择通过将所有货币价值视为从进入我们系统的那一刻起就缩放为整数的绝对类型一致性来强制执行。这本身就是金融中避免浮点数不精确的常见模式。

**我们的解决方案：将数据标准化为 BigInt 并使用缩放整数数学**

我们将滑点常数表示为基础点（1 个基点=0.01%）作为`BigInt`，并使用`BigInt`算术进行所有计算。

```js
// Represent slippage in basis points (bps) as a BigInt
const MAX_SLIPPAGE_BPS = 50n; // 50bps = 0.50%
 // The ingestion function now normalizes everything to BigInt
function handleRawTx(rawTx) {
 const tx = {
 ...rawTx,
 value: BigInt(rawTx.value),
 };
 return validateTransaction(tx);
}
 // The hot function is now 100% BigInt-safe and type-stable
function validateTransaction(tx) {
 // tx.value is always a BigInt
 const slippage = (tx.value * MAX_SLIPPAGE_BPS) / 10000n;
 // ... rest of BigInt-only logic
 return isProfitable;
}
```

这完全解决了去优化循环。现在，TurboFan 在这个函数中只看到`BigInt`，并为该特定类型生成了稳定、优化的代码。

##### 权衡和替代解决方案

虽然我们的缩放整数方法有效，但这不是唯一的解决方案，并且它有一个权衡：对于适合标准**数字**的值，`BigInt`算术通常比`Number`算术慢。我们可能正在减慢 99.9%的交易速度，以安全地处理 0.1%的鲸鱼转账。

对于我们的特定用例，稳定性值得这个权衡。然而，这里还有两种其他强大、实用的模式来解决此问题。

##### 调度器——我们决定使用

这据我们团队所说，是我们能想到的最高性能解决方案。想法是拥有两个独立的、专门的“热点”函数和一个单一的“调度器”，该调度器将交易路由到正确的函数。这保持了每个热点路径**单态性**（只看到一种类型），这正是 JIT 所爱的。

```js
function handleRawTx(rawTx) {
 const val = BigInt(rawTx.value);
 // Dispatch to the correct specialized function at the boundary
 if (val <= BigInt(Number.MAX_SAFE_INTEGER)) {
 const tx = { ...rawTx, value: Number(val) };
 return validateTransactionNumber(tx); // Fast path for most transactions
 } else {
 const tx = { ...rawTx, value: val };
 return validateTransactionBigInt(tx); // Path for whale transactions
 }
}
 function validateTransactionNumber(tx) {
 // Hot Function #1
 const slippage = tx.value * 0.005; // Fast, floating-point math
 // ...
}
 function validateTransactionBigInt(tx) {
 // Hot Function #2
 const slippage = (tx.value * 50n) / 10000n; // BigInt math
 // ...
}
```

###### 替代方案 B：受保护的分支（实用折中方案）

如果你想将你的逻辑保持在单个函数中，你可以使用显式的 `typeof` 检查。这会在同一个函数内创建两个不同的路径。虽然这使得函数 **多态**（看到多种类型），但它是对 V8 的一个明确信号，并且通常被很好地优化——肯定比重复 deopts 优化得更好。

```js
function validateTransaction(tx) {
 if (typeof tx.value === "bigint") {
 // BigInt branch
 const slippage = (tx.value * 50n) / 10000n;
 // ...
 } else {
 // Number branch
 const slippage = tx.value * 0.005;
 // ...
 }
}
```

⚠️警告

包含 bigint 路径和数字路径（通过 typeof 检查）的单个函数是多态的。这通常没问题——但在非常热的循环中，它可能会使 V8 生成比将那些路径拆分为两个单态函数并提前分发一次更慢、更不稳定的机器代码。在热路径上选择性能调度器；当分支可预测或路径不是超级热时，选择受保护的分支以保持简单。

## 内存布局和对象表示

要真正理解 V8 的性能，你必须再深入一层，思考你的 JavaScript 值实际上是如何在你的计算机内存中布局的。这不仅仅是一个学术问题；它有直接的性能影响。

V8 使用一个称为指针标记的巧妙技巧来区分立即值（如小整数）和堆上对象的指针。虽然 64 位系统使用 64 位机器字，但 V8 的默认指针压缩优化意味着这些标记值通常存储在堆上的紧凑 32 位槽中，以节省内存。

### 小整数（SMI）

如果 64 位字的最末位是 `0`，V8 就知道剩余的位表示一个 **Small Integer**，或 **Smi**。这是一个极其重要的优化。在具有指针压缩（默认）的 64 位构建中，Smi 是 31 位有符号整数，范围大约为 -1 亿到 +1 亿。V8 不需要在堆上为这些值分配任何额外的内存。这个数字 *就是* 指针。

ℹ️注意

在具有指针压缩的 64 位构建中，Smi 是 31 位有符号整数。在 32 位构建中，范围略有不同。始终假设 ~±1 亿为安全范围，而不是完整的 32 位。

Smi 的算术运算非常快。CPU 的整数算术逻辑单元（ALU）可以直接操作它们。这就是为什么使用整数计数器的 `for` 循环比处理浮点数或对象的循环要快得多。

表示 Smi（例如，数字 5）的 64 位字：

### 堆对象

那么，当那个特殊的标签位是 `1` 时会发生什么？这表示该字不是值本身，而是一个 **指针**——一个引用 V8 堆上实际数据位置的内存地址。

这就是 V8 处理所有无法表示为 Smi 的复杂数据的方式：从字符串和数组到对象，甚至带有小数点的数字。

为了保持其内存使用量低，V8 默认使用一个称为 **指针压缩** 的出色优化。这意味着即使在 64 位系统上，这些指针通常也存储在紧凑的 **32 位**槽中。

这里是 V8 内存节省模式下标签指针的简化视图：

让我们以一个数字`const a = 3.14;`为例。在经典意义上，V8 会通过创建一个`HeapNumber`对象来处理这个问题，这涉及几个步骤 -

1.  首先，它在堆上分配一小块内存。

1.  然后，它将`3.14`的浮点表示写入那个内存。

1.  最后，它将指向那个新位置的标记指针存储在变量`a`中。

但这正是故事变得精彩的地方。V8 团队已经花费多年优化这个过程，现代 V8 有一些惊人的技巧来避免频繁执行代码的堆之旅 -

+   **解包** - V8 可以直接在另一个对象的内存中存储数字的原始值，从而避免分配单独的`HeapNumber`对象的需要。

+   **逃逸分析** - 如果一个数字仅在一个函数内部创建和使用，并且从未“逃逸”到外部作用域，V8 足够智能，可以完全避免堆分配。

因此，虽然堆分配确实会产生性能成本（需要管理内存和跟踪指针），但多亏了 V8 的惊人工程，这种情况发生的频率比你想象的要低得多。

### 内存中的对象布局

当创建对象时，V8 在堆上为其分配内存。这个内存块包含 1)指向其**隐藏类**的指针。这是第一个也是最重要的字段，2)其属性的空间。

对于具有“快速属性”的对象（即不在字典模式中），属性直接存储在对象的内存块中，偏移量由隐藏类确定。

让我们想象我们的`Point`对象：`const p = { x: 1, y: 2 }`。它的内存可能看起来像这样 -

当 TurboFan 优化`p.y`时，它会从`p`中获取隐藏类，看到它是`C2`，并从`C2`知道`y`位于偏移量+8 字节（例如）的位置，然后生成机器代码来直接读取`[p 地址 + 8]`处的内存。没有哈希表查找。没有字符串比较。只是直接的、快速的内存访问。

### 字符串内化

字符串是另一个有巧妙优化的领域。当你的代码中多次出现相同的字符串字面量，比如`'success'`，V8 只会将其存储在内存中一次。这被称为**字符串内化**或“内部化”。所有指向该字符串字面量的变量都将指向相同的内存地址。这节省了内存，并使字符串比较变得更快：V8 只需比较指针（一个 CPU 指令）而不是逐字符比较字符串。

这是字符串内化在内存中的工作方式：

理解这个内存模型有助于解释许多 V8 的性能特性 -

1.  **为什么整数运算快？** Smis 避免堆分配和间接引用。

1.  **为什么隐藏类很重要？** 它们使得固定偏移量属性访问成为可能。

1.  **为什么`delete`操作慢？** 它迫使从这种干净、类似数组的属性存储方式转变为复杂、缓慢的基于字典的方式。

## 常见性能悬崖

在看到数百个生产性能问题之后，你开始一次又一次地看到相同的模式。这些是开发者，即使是高级开发者，在不了解引擎的情况下容易掉入的常见陷阱。

### Cliff #1: 不稳定的对象形状（隐藏类爆炸）

这是我们最近讨论中看到的大问题。任何创建具有不同属性顺序、不同总属性或条件属性的对象的代码都是一个雷区。

+   **症状 -** 处理对象的代码神秘地变慢。火焰图宽而平，没有单个热点函数。

+   **原因？** 由于隐藏类的爆炸，巨形态内联缓存。

```js
// Bad code
 const user = { name: "Alice" };
 if (isAdmin) {
 user.permissions = ["..."];
 }
 // `user` now has two possible shapes.
```

```js
// Good code!
 const user = {
 name: "Alice",
 permissions: null, // or `undefined` - always initialize
 };
 if (isAdmin) {
 user.permissions = ["..."];
 }
 // `user` has one stable shape.
```

### Cliff #2: 多态和巨形态函数

设计用于处理多种不同类型输入的函数是 JIT 编译器的敌人。

+   **症状 -** 特定的实用函数或事件处理器是瓶颈。

+   **原因？** 在属性访问或函数调用位置上的 IC 变成了多态或巨形态。

```js
// Bad code
 function getIdentifier(entity) {
 // This function might get a User, a Product, a Company...
 return entity.id || entity.uuid || entity.productId;
 }
```

```js
// Better!
 // Create separate, monomorphic functions.
 function getUserId(user) {
 return user.id;
 }
 function getProductIdentifier(product) {
 return product.productId;
 }
```

### Cliff #3: 在对象上使用 `delete`

我们为我们的 Node.js 服务构建了一个简单的内存缓存层。它只是一个大型的 JavaScript 对象。当一个项目过期时，我们会使用 `delete cache[key]` 从缓存中删除它。很简单，对吧？我还在 [Reddit](https://www.reddit.com/r/webdev/comments/1n23rbi/i_stopped_deleting_and_my_hot_paths_calmed_down/) 上发了一篇关于这个话题的文章，以防你感兴趣。

服务的吞吐量令人失望，大约只有我们预期的 35-40%。分析显示，大量时间被花费在缓存对象的属性访问函数中。火焰图主要由字典查找和巨形态内联缓存缺失主导。

问题在于 `delete`。当你使用 `delete` 时，你不仅仅是删除一个属性。你从根本上改变了对象的结构，V8 无法通过简单的隐藏类转换来优化。使用 `delete` 强制 V8 将对象的属性切换到 **字典模式**。在这种模式下，属性存储在一个较慢的、类似哈希表的数据结构中。该对象上每个属性访问，*即使是未删除的键*，现在都变成了慢速的字典查找。我们的整个缓存对象受到了削弱。

解决方案？**永远不要在热点对象上使用 `delete`。相反，将属性设置为 `undefined`**。

```js
// This code lives in examples/v8/performance-cliffs.js
 const cache = {};
 function setCache(key, value) {
 cache[key] = value;
}
 // The performance killer
function evictWithDelete(key) {
 delete cache[key]; // This poisons the cache object.
}
 // The performant way
function evictWithUndefined(key) {
 cache[key] = undefined; // The hidden class remains stable.
}
```

将 `undefined` 分配给变量可以保留隐藏类并保持对象处于“快速模式”。在我们将所有 `delete` 调用替换为对 `undefined` 的赋值后，我们的吞吐量立即提高了 3-4 倍。

### Cliff #4: 在数组中混合元素种类

V8 根据其内容对数组进行大量优化，并在元素种类之间动态升级：

+   `PACKED_SMI_ELEMENTS` - 最快的。只包含小整数的内存连续块。

+   `PACKED_DOUBLE_ELEMENTS` - 仍然很快。连续的双精度浮点数块。

+   `PACKED_ELEMENTS` - 指向堆对象的指针的连续块。

+   `HOLEY_ELEMENTS` - 具有空槽的数组（例如，`const a = [1, , 3]`）。较慢，因为 V8 必须检查空槽。

+   `DICTIONARY_ELEMENTS` - 最慢的。数组基本上是一个哈希表。

如果你从 `[1, 2, 3]` 开始，然后 `push('hello')`，V8 必须将整个底层存储从 `PACKED_SMI_ELEMENTS` 转换为 `PACKED_ELEMENTS`。这种转换是单向的，主要影响在数组被重复访问的热点循环中的性能。对于大多数日常数组使用，V8 高效地处理这些转换，但在性能关键的数值计算或处理大数据集时，保持稳定的元素类型可以提供显著的好处。

📌重要

混合元素类型会强制进行存储转换，这可能会在热点循环中损害性能。对于大多数日常使用，这种影响可以忽略不计，但在紧密的数值循环或大数据集中，稳定的元素类型很重要。

## 有效的模式

那么，我们如何保持 V8 的好感？通过编写无聊、可预测、单态的代码。这可能感觉不那么“聪明”，但它将快得多。

### V8 友好的代码模式

这是一个性能关键数据结构和处理它的函数的蓝图。

```js
// This code lives in examples/v8/optimization-triggers.js
 // 1\. Use a class or a constructor function to define a stable shape.
// This is the most reliable way to ensure all objects share a hidden class.
class DataPacket {
 constructor(id, timestamp, payloadType, payload) {
 // 2\. Initialize all properties in the constructor.
 // The order here DEFINES the hidden class. Keep it consistent.
 this.id = id; // Should be a Smi if possible
 this.timestamp = timestamp; // Number
 this.payloadType = payloadType; // String
 this.payload = payload; // Object or null
 }
}
 // 3\. Keep your processing function MONOMORPHIC.
// It's designed to work ONLY with DataPacket objects.
// Use types (e.g., TypeScript) to enforce this at the boundary.
function processPacket(packet) {
 // 4\. Access properties consistently. This keeps ICs warm and monomorphic.
 const id = packet.id;
 const type = packet.payloadType;
 // 5\. Prefer Smi-friendly arithmetic in hot loops.
 // Bitwise operations are a good sign you're dealing with Smis.
 if ((id & 1) === 0) {
 // Do something for even IDs
 }
 // 6\. Avoid deoptimization triggers.
 // No `delete`, no `arguments`, no `try/catch` in the absolute hottest part.
 if (type === "USER_EVENT" && packet.payload) {
 // Process payload
 }
}
 // This is how you could use the points mentioned above
 const packets = [];
// 7\. Create objects with the same shape.
for (let i = 0; i < 1000; i++) {
 packets.push(new DataPacket(i, Date.now(), "USER_EVENT", { data: "..." }));
}
 // 8\. Run the hot function. V8 will optimize `processPacket`
// based on the stable `DataPacket` shape.
function processAll() {
 for (let i = 0; i < packets.length; i++) {
 processPacket(packets[i]);
 }
}
 // 9\. Profile and measure!
console.time("Processing");
processAll();
console.timeEnd("Processing");
```

### 优化策略：检查清单

在你考虑更改代码之前，遵循以下流程：

1.  不要猜测。使用 `--prof` 或 Chrome DevTools 检查器来找到你的实际热点路径。优化只运行 0.1% 时间的代码是徒劳的。

1.  专注于消耗最多 CPU 的 1-3 个函数。这就是你应该关注的焦点——“识别热点路径”。

1.  对于每个热点函数，分析它接收到的数据。对象形状是否一致？数组元素类型是否稳定？使用 `console.log(%HaveSameMap(a, b))` 来验证。

1.  使用 `--trace-deopt` 和 `grep` 查找你的函数名称，在紧密循环中运行你的热点路径。查看 V8 提供的退出原因。

1.  重新构建你的代码以实现可预测性 -

    +   使用所有属性（`null` 或 `undefined` 是可以的）初始化对象。

    +   使用构造函数或工厂函数来强制执行单个创建路径。

    +   如果一个函数需要处理不同的形状，将其拆分为多个单态函数。

    +   将 `delete` 替换为 `obj.prop = undefined`。

    +   避免在性能关键数组中混合元素类型。

1.  重构后，再次运行分析器。变化是否有效？在该函数中花费的时间是否降低？

## V8 标志和运行时选项

Node.js 运行时提供了一组强大的 V8 标志，允许你调整、调试和检查引擎的行为。你可以使用 `node --v8-options` 列出你 Node.js 版本的所有可用标志。以下是一些对性能工程最关键的标志。

### 信息性标志

这些标志不会改变行为，但提供了一串诊断信息。

+   `--trace-opt`：如前所述，它会记录优化编译器（Sparkplug、Maglev、TurboFan）成功优化的每个函数。这对于确认你的热点路径实际上正在被编译非常有用。

+   `--trace-deopt`: 记录每次去优化，包括函数名、字节码偏移和原因。这可能是最重要的性能调试标志。

+   `--trace-ic`: 显示内联缓存的州转换。对于诊断多态和巨形态调用点非常有用。你可以看到属性访问从`1`（单态）变为`P`（多态）到`N`（巨形态）。

+   `--trace-gc`: 为每次垃圾回收事件打印一条日志行，显示收集了多少内存以及花费了多长时间。对于诊断内存压力和 GC 暂停非常有用。

### 行为标志

这些标志可以改变 V8 优化和执行代码的方式。使用时请谨慎。

+   `--allow-natives-syntax`: 提供了`%`函数来脚本化 V8 的内部结构。对于详细分析和基准测试很有帮助，但绝不应该用于生产环境。

+   `--optimize-for-size`: 请求 V8 在优化时更加保守，以减少编译代码的内存占用。这在内存受限的环境中非常有用。

+   `--max-old-space-size=<megabytes>`: 设置旧生代堆的最大大小。增加这个值可以减少内存密集型应用程序的 GC 频率，但也会导致 GC 发生时的暂停时间更长。

+   `--jitless`: 这是一个有趣的选择。它完全禁用了 JIT（Sparkplug、Maglev、TurboFan）。你的代码将只由 Ignition 解释器运行。这提供了更好的安全性（通过防止运行时代码生成），但会以显著的性能成本为代价。这对于建立“基线”性能以查看优化编译器实际帮助了多少非常有用。

### 如何使用标志

你在脚本名称之前传递这些标志给`node`可执行文件：`node --trace-deopt --max-old-space-size=4096 my_app.js`

你也可以通过`NODE_OPTIONS`环境变量来设置它们，这对于开发环境或 CI/CD 管道通常更方便：`export NODE_OPTIONS="--trace-deopt --max-old-space-size=4096"` `node my_app.js`

了解如何使用这些标志就像拥有了一个针对引擎的诊断工具包。当性能出现问题时，你不再需要猜测；你可以直接询问 V8 发生了什么。

## 从全代码生成到 TurboFan

我们讨论过的 V8 架构——包含 Ignition、Sparkplug、Maglev 和 TurboFan 的 4 层管道——是多年演化的结果。要欣赏其设计，了解其前身是有帮助的。

多年来，V8 的管道由两个主要组件组成 -

**Full-Codegen -** 一个简单、非优化的编译器。其任务是尽可能快地将 JavaScript 转换为机器代码。它编译速度快，但生成的机器代码较慢。它是“现在足够好”的编译器。

**Crankshaft -** 优化编译器。与 TurboFan 类似，Crankshaft 会将热函数重新编译成高度优化的机器代码。它使用了一种称为**静态单赋值（SSA）**的形式，并具有复杂的优化流程。

然而，这个架构有几个问题：

+   Full-Codegen 和 Crankshaft 生成的代码之间有很大的性能差距。你的代码要么很慢，要么非常快，中间很少有。

+   从 Crankshaft（或常被称为“退出”的过程）中退化的过程是复杂且缓慢的。

+   Full-Codegen 生成了大量的机器代码，而 Crankshaft 流水线本身也相当消耗内存。

+   两个编译器共享的代码不多。添加新的语言特性通常意味着以两种不同的方式实现它们两次。

### 新的流水线：Ignition 和 TurboFan（以及后来的 Sparkplug 和 Maglev）

V8 团队承担了一个庞大的项目，从底层重新架构引擎。这导致了现代的流水线 -

**Ignition（解释器）-** 团队意识到，对于许多 Web 和 Node.js 应用程序，预先编译成机器代码（即使是 Full-Codegen）都是过度行为。一个生成和运行字节码的解释器将具有更低的内存占用和更快的启动时间。这是一个关键的洞察：**字节码通常比未优化的机器代码更好**。内存节省是显著的，尤其是在移动设备上。

**TurboFan（编译器）-** TurboFan 从一开始就被设计成一个更先进和灵活的优化编译器。

+   它有一个基于“节点海洋”图表示的更清晰的架构。

+   它旨在优化不仅仅是 JavaScript，还包括 WebAssembly 和其他语言。

+   它有一个定义得更好的“分层”模型。从 Ignition 到 TurboFan 的路径更平滑，退化过程不那么灾难性。

+   它可以比 Crankshaft 更优雅地处理像`try...catch`和`with`这样的结构。

**Sparkplug（2021）-** 作为基准编译器添加，以平滑解释器和优化器之间的过渡。

**Maglev（2023）-** 最新加入的，提供了一种平衡编译速度和代码质量的中间层优化级别。这个从 V8 v5.9（2017）到当前 4 层系统的演变，提供了我们在本章中讨论的显著的实际好处。

这是架构演变的过程 -

这段历史澄清了另一个神话。

### 神话 3：“现代 V8 优化一切”

差得远。V8 并不想优化一切。优化是昂贵的。现代 Ignition/Sparkplug/Maglev/TurboFan 流水线的整个设计基于**分层编译**的理念。它只做必要的最小工作来运行你的代码（Ignition），并且只将 CPU 周期用于优化那些真正性能关键的小部分代码。你的任务是让这部分代码易于优化编译器理解和优化。

## V8 性能的最佳实践

这是可操作的总结。如果你正在进行性能审查并需要一个清单，这就是它。

**DO's -**

+   使用构造函数、类或工厂函数确保对象具有相同的隐藏类。初始化所有成员，即使是 `null` 或 `undefined`。

+   编写操作单一、可预测对象形状（单形）的函数。如果您必须处理多个形状，考虑将函数分解成更小的、单形的函数。

+   尽可能利用小整数（Smis）的快速路径。

+   不要猜测瓶颈在哪里。使用 `node --prof` 或 Chrome 检查器来查找它们。

+   使用 `--trace-deopt` 来查找和修复去优化循环。

+   聪明、动态的代码通常是 JIT 编译器的敌人。简单、直接的代码更容易被 V8 优化。

**不要做的事情 -**

+   永远不要在热点路径上的对象上使用 `delete`。将属性设置为 `undefined`。

+   避免编写参数可以是多种不同形状或类型的函数。

+   避免在对象创建后添加属性。

+   **不要使用 `arguments` 对象。** 使用剩余参数（`...args`）代替。它们是完全可优化的。

+   **不要使用 `eval` 或 `with` 语句。** 这些对编译器来说是黑盒，会降低性能。

+   **不要忽略去优化。** 热函数中的去优化是一个关键的性能错误。

### 您可以随时携带的清单

+   我的临界数据对象是否有一致、稳定的形状？

+   我的性能关键函数是否是单形的？

+   我是否在我的热点路径上运行了 `--trace-deopt` 以检查优化退出？

+   我是否在负载下对应用程序进行了性能分析，以确认瓶颈在哪里？

+   我的对象和数组的内存布局是否尽可能高效（例如，避免空隙，使用 Smis）？

+   我是否测量了“之前”和“之后”，以确认我的优化产生了积极的影响？

## 附录：V8 性能分析命令

您最常使用的命令的快速参考。

**基本 CPU 性能分析：**

```js
# 1\. Generate the V8 log file
node --prof my_app.js
 # 2\. Process the log file into a human-readable summary
node --prof-process isolate-XXXX-v8.log > profile.txt
```

**使用 Chrome 开发者工具进行远程调试和性能分析：**

```js
# Run your app with the inspect flag
node --inspect my_app.js
 # Or to break on the first line:
node --inspect-brk my_app.js
```

然后在您的 Chrome 浏览器中打开 `chrome://inspect`。

**跟踪 JIT 编译器行为：**

```js
# See what gets optimized
node --trace-opt my_script.js
 # See what gets deoptimized (and why)
node --trace-deopt my_script.js
 # See Inline Cache state changes
node --trace-ic my_script.js
 # Combine and filter for a specific function
node --trace-opt --trace-deopt my_script.js | grep "myHotFunction"
```

**使用 V8 内置函数进行基准测试/调试：**

```js
# Must be run with --allow-natives-syntax
node --allow-natives-syntax my_benchmark.js
```

常见内置函数：%HaveSameMap(obj1, obj2)，%GetOptimizationStatus(func)，%OptimizeFunctionOnNextCall(func)。

* * *

## 关闭 - 思考像 V8 一样

我们开始这段旅程时带着一个谜团：一行简单的代码导致性能降低了 100 倍。解决方案不是一种巧妙的算法或复杂的重构。这是关于揭开盖子，看看真正让您的 JavaScript 发动起来的原因。

作为 Node.js 开发者，您可以做出的最大转变是停止将 V8 视为一个不可预测的黑盒。它不是。它是一个高度有意见、确定性的系统。它有强烈的偏好：它喜欢稳定的对象形状，它喜欢单形函数，它喜欢整数运算，它讨厌不可预测性。

你的工作不是要战胜编译器。你会失败的。你的工作是要编写让编译器工作变得容易的代码。给它提供它设计来吞噬的简单、可预测的模式的稳定饮食，它将会以甜美的速度回报你。通过动态形状和多态调用制造混乱，它将会通过退回到慢速路径来保护自己——以及你应用程序的正确性。

下次你遇到奇怪的性能问题时，不要只关注你代码的逻辑。问问自己：“V8 会如何看待这个问题？”思考你正在创建的隐藏类。思考你正在生成的类型反馈。思考你的代码是如何通过四层编译管道的。像 V8 一样思考，性能问题将会开始自行解决。

* * *
