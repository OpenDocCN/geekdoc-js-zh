# 与缓冲区一起工作

> 原文：[`www.thenodebook.com/buffers/working-with-buffers`](https://www.thenodebook.com/buffers/working-with-buffers)

ℹ️备注

本章深入探讨了缓冲区。如果任何部分感觉不清楚或令人不知所措，请不要担心，重新阅读该部分，或者在其他（子）章节阅读完毕后再次回顾。

我很确定你在这里是因为你希望学习如何将之前章节中学到的所有知识应用到实践中，或者你的 Node.js 服务正在消耗过量的内存。或者也许是因为你的高吞吐量二进制协议解析器执行速度极慢。罪魁祸首几乎总是对 Node.js `Buffer`如何处理内存的根本性误解。本章将引导你走出这个地狱。我们将拆解 Node.js 开发中最危险的假设：即`Buffer.slice()`的行为类似于`Array.prototype.slice()`。它并不相同。一个创建了一个新的、独立的数据副本；另一个创建了一个“视图”——仅仅是原始数据的窗口。这是零拷贝操作的核心。

🚨警告

`Buffer.slice`已被弃用，取而代之的是`Buffer.subarray()`。然而，理解其行为对于维护遗留代码库和了解视图机制仍然是至关重要的。

当正确使用时，这些视图非常强大，让你几乎无需内存开销就能处理大量数据。当被误解时，它们会创建你遇到的最隐蔽、最难调试的内存泄漏——一个只有 10 字节的切片却可以锁定 1GB 的缓冲区在内存中，阻止垃圾回收器回收它。我们将回顾这些战争故事、深夜调试会话和生产中断，这些经历塑造了这些知识。你将了解视图（`slice`、`subarray`）和真正的副本（`Buffer.copy()`）之间的关键区别，以及何时使用每个。我们将探讨`Buffer`和`TypedArray`之间的密切关系，它们共享相同的内存基础（`ArrayBuffer`），以及这既可以是一种超级能力，也可能是一个微妙的内存损坏的来源。到本章结束时，你不仅会了解 API，而且会对内存语义有深刻的、几乎本能的尊重。你将理解为什么你的服务使用 10GB 的内存来处理 1GB 的数据，更重要的是，如何永久修复它。

## 吉字节级内存泄漏

你是否见过一个本应只使用整洁的 500MB RAM 的服务突然决定它需要 10GB 才能运行？这是对内存理解的惊人误解。让我们一步步了解，即使怀着最好的意图和看起来完美的代码，你如何构建这种灾难。这是学习如何预防它的最佳方式。

⚠️注意

本章中描述的缓冲区内存保留模式是生产环境中 Node.js 内存泄漏的首要原因。单个`Buffer.slice()`可以无限期地锁定数 GB 的内存。

想象一个常见的场景：一个处理大量数据的服务。也许它们是日志，也许是多部分文件上传。任务很简单：对于每个传入的块（可能是几个兆字节），你需要解析一个小的固定大小的头部来提取一个标识符，比如会话 ID。你会编写一个看起来像这样的函数。说实话，你可能已经写了十几次这样的代码。

```js
// This function gets called for thousands of incoming multi-megabyte chunks.
function getSessionId(logBuffer) {
 // The session ID is always the first 16 bytes.
 const headerSlice = logBuffer.slice(0, 16);
```

立即停止。这条确切的一行——`logBuffer.slice(0, 16)`——是你的生产系统开始死亡螺旋的地方。下面是 Node.js 内部实际发生的事情。当你调用`slice()`时，Node.js 不会分配新的内存。相反，它创建了一个微小的 JavaScript 对象（在 V8 上大约是 72 字节），其中包含三个关键信息：指向父缓冲区的 ArrayBuffer 的指针、偏移量（在这种情况下为 0）和长度（16）。这个对象存在于 V8 堆上，但它保持对外部内存的强引用，其中`logBuffer`存储其实际数据。

V8 垃圾回收器看到这个引用并将整个父缓冲区标记为“可到达”。即使你只关心 16 个字节，垃圾回收器也必须保持整个多兆字节的缓冲区存活。在 V8 的代际垃圾回收系统中，这个父缓冲区在经历两次清理后从年轻代提升到老年代，这使得它更难被回收。我见过这种模式在生产环境中让 100MB 的缓冲区存活数小时，只是为了存储一小把 16 字节的会话 ID。

```js
// We'll store this slice in a map or cache to batch requests later.
 return headerSlice;
}
```

这段代码不可避免地会导致生产故障。它看起来很无辜，但`logBuffer.slice(0, 16)`是导致你的生产环境完全失败的那一行。下面是实际发生的情况。你每分钟处理大约 100MB 的日志。你的服务内存使用量（常驻集大小，或 RSS）应该保持相对稳定。相反，你看到它以千兆字节为单位攀升。你保留 10GB 的内存来管理本应最多只有几兆字节的会话 ID。

因此，你进行了一个堆快照，而你看到的东西毫无意义。分析器显示给你成千上万的微小的 16 字节`Buffer`对象，但它声称它们共同保留了数 GB 的内存。你的第一个想法是，“分析器坏了。”其实并没有。它向你展示了一个基本真理：切片不是副本，它是一个视图。那个 16 字节的`headerSlice`对象只是一个轻量级的 JavaScript 包装器，但它持有对原始的多兆字节`logBuffer`的内部引用。只要那个微小的切片存活——比如说，它坐在你的缓存中——垃圾回收器就无法回收整个大的缓冲区。

你并没有泄露几个字节。你泄露了每个请求的整个父缓冲区。乘以数千个请求，你就有了我们正在剖析的 10GB 内存泄露的配方。这是误解`Buffer` API 中最常见方法之一的后果。让我们深入探究原因。

好吧，让我们去掉废话。正如我们在上一章中提到的，试图用 JavaScript 字符串处理原始二进制数据是灾难的配方。要理解解决方案，即`Buffer`，你必须将其刻在你的脑海中：**Node.js 的`Buffer`内存不生活在 V8 堆上**。

### 理解 Buffer 内存架构

记得我们提到 V8 的世界是为小型、相互连接的 JavaScript 对象构建的吗？它的垃圾回收器（GC）在清理字符串和对象方面是一把好手，但在处理巨大的、单一的二进制数据块时却显得力不从心。一次大量的文件读取会触发一个使应用程序冻结的“停止世界”暂停，从而杀死你的性能。

因此，Node 做了明智的事情。它将那大块内存**外部**分配在 C++领域，更接近硬件。这通常被称为“堆外”或“外部”内存。

你在 JavaScript 代码中操作的`Buffer`对象，正如我们之前提到的，只是一个微小的、轻量级的**句柄**，它生活在 V8 堆上。它就像一张钥匙卡。钥匙卡本身很小，V8 GC 很容易追踪。但它持有指向那块巨大原始内存的引用。

这个两部分的系统是既带来惊人性能又造成巨大困惑的源头：

+   Node 可以将这些巨大的内存块直接传递给文件系统或网卡，而无需将它们复制到 JavaScript 的世界中。这非常高效。

+   V8 垃圾回收器只能看到这张微小的钥匙卡。如果你的代码意外地保留了这张钥匙卡（比如在闭包或长期存在的对象中），V8 就不会清理它。只要钥匙卡存在，它就充当锚点，防止那巨大的、多兆字节的块外部内存被释放。你并不是在泄漏几个字节的 JavaScript 对象；你是在泄漏它们指向的巨大内存块。

### 8KB 速度黑客：Buffer 池

现在，正如我们在讨论分配模式时提到的，Node 为较小的缓冲区有一个速度黑客：**buffer 池**。为了避免不断向操作系统请求内存，Node 预先分配了一个 8KB（`Buffer.poolSize`）的内存块。对于任何小于 4KB 的缓冲区，Node 只是从这个池中切下一块，而不是去麻烦操作系统为新内存。

这对于使用大量小缓冲区的应用程序来说是一个巨大的性能提升。这也是使`Buffer.allocUnsafe()`如此危险的确切机制，我们之前已经剖析过。你并没有得到新鲜内存；你得到的是池中回收的碎片，它可能充满了来自应用程序其他部分的秘密——比如另一个用户的会话令牌——而这些部分在几秒钟前运行过。

## 视图和引用：`slice`、`subarray`和`Buffer.from`

现在我们对缓冲区内存所在的位置有了更清晰的了解，让我们谈谈你用来操作它的工具。这是理论变成实践的地方，也是大多数开发者做出错误决定的地方。我们需要了解的三个主要函数是 `Buffer.slice()`、`Buffer.subarray()` 和 `Buffer.from()`（当与另一个缓冲区或 `ArrayBuffer` 一起使用时）。

让我们从引起很多麻烦的那个开始：`slice()`。如果你有 JavaScript 的背景，你的习惯编程模式可能会让你认为 `Array.prototype.slice()` 创建了一个浅拷贝。你切割一个数组，你会得到一个新的数组，你可以修改其中一个而不影响另一个。当涉及到缓冲区时，这其实是一个谎言。

`Buffer.prototype.slice()` **不会** 创建一个拷贝。它创建了一个**视图**。

让我再重复一遍，因为这是本章中最关键的句子：`Buffer.slice()` 创建了一个与原始缓冲区共享内存的视图。它切割出一个新的 `Buffer` 对象，但这个新对象指向与原始对象**完全相同的字节**，在相同的底层 `ArrayBuffer` 中。

🚨注意

`Buffer.slice()` 并不像是 `Array.slice()`。数组创建拷贝，缓冲区创建视图。修改切割的缓冲区会修改原始缓冲区。这个误解导致了 Node.js 生产环境中大多数内存泄漏和数据损坏错误。

让我展示一下那个看似无辜的代码，它几乎让我失去了理智。

```js
// Imagine this is a 50MB buffer read from a network stream.
const massiveBuffer = Buffer.alloc(50 * 1024 * 1024);
massiveBuffer.write("USER_ID:12345|REST_OF_DATA...");
```

那个 `Buffer.alloc()` 调用在 Node 的内部触发了连锁反应。首先，Node.js 检查请求的大小（52,428,800 字节）是否大于 `Buffer.poolSize >>> 1`（4096 字节）。它是，所以 Node 完全绕过了缓冲池。它直接调用 C++ 层来分配 50MB 的内存。在 Linux 上，这通常会导致一个 `mmap()` 系统调用，用于大分配，它将匿名页面映射到你的进程地址空间。内核实际上并没有分配物理 RAM，它使用了一种称为“按需分页”的技术，其中物理页面只有在第一次写入时才会分配。这就是为什么 `Buffer.alloc()` 会将内存清零——它迫使内核立即分配真实的物理页面。

然后 `write()` 操作会将你的字符串数据复制到这个缓冲区中，当可用时使用优化的 SIMD 指令。V8 的字符串编码机制将 UTF-8 JavaScript 字符串转换为原始字节。对于 ASCII 字符，这是一个直接的复制。对于多字节 UTF-8 字符，编码器必须仔细跟踪字节边界，以避免分割字符。这种编码在一个紧密的 C++ 循环中发生，该循环已被优化，以使用向量指令在每次 CPU 周期中处理多个字节。

```js
// This creates a VIEW into the same memory. No copy!
const userIdSlice = massiveBuffer.slice(9, 14); // Extracts "12345"
console.log(userIdSlice.toString()); // Output: 12345
```

那个操作非常快，因为它不需要分配 5 个字节并将数据复制进去。它只是创建了一个带有不同偏移量和长度的微小新 JavaScript 对象，该对象指向原始的 50MB `ArrayBuffer`。现在，如果我们修改这个视图会发生什么呢？

```js
// Let's modify the slice.
userIdSlice.write("99999");
```

这单个写入操作就破坏了你的原始 50MB 缓冲区。当你对切片调用`write()`时，Node.js 会计算父`ArrayBuffer`的绝对位置：切片的基本偏移量（9）加上写入位置（0）等于父内存中的字节位置 9。字符串"99999"被编码为 UTF-8 字节[0x39, 0x39, 0x39, 0x39, 0x39]，并直接写入父缓冲区的内存位置 9-13。没有写时复制机制，没有保护，没有警告。写入是通过 C++中的直接内存指针操作完成的，绕过了 JavaScript 的所有安全机制。在生产环境中，我见过这种模式破坏二进制协议头，覆盖关键元数据，甚至从一个请求泄露敏感数据到另一个请求。

```js
// Now let's look at the original buffer again.
console.log(massiveBuffer.toString("utf-8", 0, 20));
// Output: USER_ID:99999|REST_O
```

你看到了吗？我们改变了`userIdSlice`，它修改了`massiveBuffer`。它们是两个引用相同内存位置的 Buffer 对象。当你通过一个 Buffer 引用修改数据时，通过另一个 Buffer 引用立即可以看到更改，因为它们都指向相同的底层`ArrayBuffer`。基于`TypedArray`视图的 V8 文档，`Buffer`就是基于这个视图的，确认这种共享内存行为是故意的，并且是其设计的基本原则。

那么`subarray()`呢？在当前的 Node.js 版本中，`Buffer.prototype.subarray()`实际上与`Buffer.prototype.slice()`相同。两者都创建了对相同内存的视图，而不是副本。Node.js 文档建议在你想表明你正在`TypedArray`规范内工作时使用`subarray()`，因为`subarray`是创建视图的标准`TypedArray`方法。

ℹ️注意

`Buffer.slice()`和`Buffer.subarray()`在功能上相同。两者都创建视图，而不是副本。为了与`TypedArray`约定保持一致，请使用`subarray()`。

```js
const mainBuffer = Buffer.from([1, 2, 3, 4, 5]);
const sub = mainBuffer.subarray(1, 3); // A view of bytes [2, 3]
```

那个`subarray()`调用创建了一个只包含三个重要属性的新的 Buffer 对象：对`mainBuffer`的`ArrayBuffer`的引用、偏移量为 1、长度为 2。总成本大约是 V8 堆上的 72 字节用于 JavaScript 对象本身。没有为实际数据分配内存。视图的内部`[[ViewedArrayBuffer]]`槽直接指向父的存储。当你访问`sub[0]`时，V8 执行指针运算：它从父内存的基址开始，加上视图的偏移量（1 字节），并从该位置读取。这完全在编译的机器代码中完成，没有任何 JavaScript 开销。

```js
sub[0] = 99; // Modify the view
console.log(mainBuffer); // Output: <Buffer 01 63 03 04 05> (Note the 99 is 0x63)
```

第三个字符是`Buffer.from()`。这个函数有点棘手，因为它的行为完全取决于你提供的输入类型。

+   `Buffer.from(string)`: **分配新的内存**并将字符串数据复制到其中。

+   `Buffer.from(array)`: **分配新的内存**并复制字节数据。

+   `Buffer.from(arrayBuffer)`: **创建一个与提供的`ArrayBuffer`共享内存的视图**。这是一个零复制操作。

+   `Buffer.from(buffer)`: **分配新的内存** 并从源缓冲区复制数据。这是一个完整的拷贝！

⚠️警告

`Buffer.from(arrayBuffer)` 创建一个视图（共享内存），但 `Buffer.from(buffer)` 创建一个拷贝（新内存）。这种不一致性是常见的错误来源。始终根据您的输入类型验证您获得的行为。

`Buffer.from(arrayBuffer)` 和 `Buffer.from(buffer)` 之间的区别是常见的错误来源。前者是一个零拷贝视图，而后者是一个完整的拷贝操作。那个静默地破坏了我们二进制协议的 `TypedArray` 视图教会了我不要不测量就信任。我们有一个函数，有时会传入一个 `ArrayBuffer`，有时会传入一个 `Buffer`，而 `Buffer.from()` 语义的微妙差异在我们的热点路径中导致了意外的拷贝，从而降低了性能。

## 零拷贝操作

“零拷贝”这个术语听起来很有吸引力，但具有误导性。它听起来像是在不付出任何代价的情况下实现性能提升。事实并非如此。这里有一个权衡，你需要理解它。零拷贝操作意味着你不会复制 *数据负载*。然而，你仍然在创建一个新的 JavaScript 对象——视图本身。这个对象在 V8 堆上有很小的内存占用，但它的创建速度比分配新的内存块然后逐字节迭代原始数据来复制它快得多。

让我们量化一下。假设我们有一个 10MB 的缓冲区，我们想要从中获取 1KB 的块。

```js
const largeBuffer = Buffer.alloc(10 * 1024 * 1024); // 10MB
const chunkSize = 1024; // 1KB
```

这个分配触发了单个 `mmap()` 系统调用，用于 10,485,760 字节。内核保留了虚拟地址空间，但尚未分配物理页面——这发生在第一次通过按需分页写入时。Node.js 通过 `Isolate::AdjustAmountOfExternalAllocatedMemory()` 通知 V8 的垃圾收集器，这会影响下一次主要 GC 周期的触发。如果外部内存增长过快，V8 将崩溃并强制进行同步 GC，这可能会阻塞你的事件循环数百毫秒。

```js
// --- The Zero-Copy View ---
console.time("view creation");
const view = largeBuffer.subarray(5000, 5000 + chunkSize);
console.timeEnd("view creation");
```

`subarray()` 操作在纳秒级别完成。它为新的 Buffer 对象在 V8 堆上分配了正好 72 字节，并设置了三个字段：缓冲区指针、偏移量（5000）和长度（1024）。没有内存屏障，没有缓存失效，没有 TLB 清除。CPU 可以将整个操作保持在 L1 缓存中。性能计数器显示 ~0.007ms，因为那主要是 `console.time()` 自身的开销——实际的子数组操作在现代 CPU 上不到 100 纳秒。

```js
// --- The Full Copy ---
console.time("copy creation");
const copy = Buffer.alloc(chunkSize);
largeBuffer.copy(copy, 0, 5000, 5000 + chunkSize);
console.timeEnd("copy creation");
```

在现代 Node.js 安装中，结果是有说服力的：

+   `视图创建`: `0.007ms`

+   `拷贝创建`: `0.024ms`

💡提示

使用 `performance.timerify()` 或 `perf_hooks` 模块在生产环境中准确测量缓冲区操作。`console.time()` 方法很方便，但对于亚毫秒级的测量来说不够精确。

创建视图的成本实际上是常数时间，O(1)。无论你查看 10 字节还是 10 兆字节，你只是在创建一个带有一些指针和偏移的小型 JS 对象。然而，复制的成本是线性时间，O(n)。它与你要复制的数据量成正比。对于一个从 100MB 缓冲区中提取的 1MB 数据块，视图仍然几乎是瞬时的，而复制则需要消耗一个可测量的毫秒时间。在一个热点路径中，这会累积起来。我们自己的遥测数据显示，在关键解析循环中将不必要的复制替换为视图可以减少 CPU 使用率 30%。

但是这里有权衡，开发者常见的错误。你听到“零复制”就会想到“更快”。所以你开始进行“优化”遍历，在看到复制的地方用视图替换它们。但你实际上是在用 CPU 周期换取内存管理复杂性。视图之所以快，是因为它不需要管理自己的内存；它只是借用父级的。这创建了一个强引用，垃圾收集器必须尊重。只要你的视图是活跃的，整个父缓冲区就会在内存中被固定。

这是基本的权衡：你用内存安全性换取速度。你是在告诉运行时，“相信我，我知道我在做什么。保留这块巨大的内存块，因为我需要它的一小部分。”运行时会完全按照你的要求去做。而且如果你不小心，它甚至会一直信任你直到出现内存不足异常。正确的“优化”不是在所有地方都使用视图，而是要理解何时小而明确的复制的成本远远低于保留大型父缓冲区的内存成本。

### 缓冲区、类型化数组以及它们共享的内存

好吧，所以我们已经确定 `Buffer` 是 Node 处理二进制数据的特殊调料。但正如我们在第一章中看到的，它们并不是孤立存在的。它们是更大家族 `TypedArray` 的一部分，理解这种关系至关重要。

自从 Node.js v3.0 版本以来，`Buffer` 类是 `Uint8Array` 的直接子类。

```js
const buf = Buffer.from("hello");
console.log(buf instanceof Uint8Array); // true
```

这不仅仅是你下次面试的 trivia。这是互操作性的金票。这意味着你可以将 Node `Buffer` 传递给任何期望 `Uint8Array` 的现代 API - 在浏览器中、在 WebAssembly 中、在任何地方 - 它都能正常工作。

关键概念是这样的：原始的内存块本身是一个 `ArrayBuffer`。`Buffer`、`Uint8Array`、`Int32Array` 等，都是你可以放置在相同原始内存上的不同 **视图**。将 `ArrayBuffer` 想象成一块原始的钢铁。一个 `Buffer` 是一个模板，让你可以看到它作为一个字节序列。一个 `Int32Array` 是一个不同的模板，将那些字节分组为 4 字节块，并显示 32 位数字。

这非常强大。这也是你可以在不抛出任何错误的情况下静默地损坏所有数据的方法。

让我们来看看犯罪现场。想象一下，我们从网络中获取了一个 12 字节的消息。

```js
// A 12-byte ArrayBuffer, zero-filled for safety.
const messageArrayBuffer = new ArrayBuffer(12); //
```

现在，让我们创建几个视图来处理这个内存。

```js
// View 1: A Buffer to write a status string into the LAST 8 bytes.
const stringView = Buffer.from(messageArrayBuffer, 4, 8); //
stringView.write("CONFIRMD"); //
 // View 2: An Int32Array to read a 4-byte integer from the FIRST 4 bytes.
const intView = new Int32Array(messageArrayBuffer, 0, 1); //
console.log("Initial integer value:", intView[0]); // 0
```

一切都很干净。视图指向同一内存块的不同、不重叠的部分。但随后出现了一个错误——一个经典的“偏移量错误”或偏移量计算中的打字错误。

```js
// Here comes the bug. We accidentally create the string view at offset 0.
const buggyStringView = Buffer.from(messageArrayBuffer, 0, 8); //
 // We write a status update, thinking we're writing to the string part.
buggyStringView.write("CANCELED"); //
```

然后**嘭**。静默的数据损坏。

你的代码“工作”了。没有异常，没有崩溃。但是因为你的`buggyStringView`与你的`intView`重叠，写入那个字符串就彻底摧毁了你的整数。代表“CANC”(`[0x43, 0x41, 0x4E, 0x43]`)的字节现在占据了你的数字曾经所在的确切内存位置。在生产环境中，这种类型的错误会破坏财务数据或使安全令牌无效，并需要几周时间才能追踪到。

```js
// Now let's read the integer from our original, "safe" view.
console.log("Corrupted integer value:", intView[0]); // 1128353859
```

这里的教训简单而残酷：当你开始对单个`ArrayBuffer`创建多个视图时，你已经启动了自动内存管理器，并为自己聘请了这项工作。你必须对每个偏移量和每个长度负责。搞错了，你将陷入调试看似正确但下一毫秒就变成垃圾数据的噩梦。

🚨警告

在同一`ArrayBuffer`上创建多个`TypedArray`视图可能会静默地相互破坏数据。没有对重叠视图进行运行时检查。在偏移量计算中单个“偏移量错误”就可能导致关键数据损坏而不会抛出任何错误。

## 当视图共享内存（以及它们不共享内存时）

到现在为止，你应该对共享内存保持健康的警惕。一般来说：**如果一个操作没有明确说明它“复制”或“分配”，就假设它共享内存**。让我们构建一个常见的`Buffer`和`TypedArray`操作的思维导图，并将它们放入“视图”（共享内存）或“复制”（分配新内存）类别中。

**创建视图的操作（零复制）：**

+   `Buffer.prototype.slice(start, end)`

+   `Buffer.prototype.subarray(start, end)`

+   `new Uint8Array(arrayBuffer, byteOffset, length)` (以及所有其他接受`ArrayBuffer`的`TypedArray`构造函数)

+   `Buffer.from(arrayBuffer, byteOffset, length)`

这些是你的高性能、高风险工具。它们在创建现有数据的临时子部分进行临时处理时速度极快。这里的关键词是*临时*。如果你创建的视图生命周期短，很快就会超出范围，你将获得所有性能优势，而不会承担内存保留风险。

**创建副本的操作（分配新内存）：**

+   `Buffer.alloc(size)`

+   `Buffer.from(string)`

+   `Buffer.from(array)`

+   `Buffer.from(buffer)`

+   `Buffer.prototype.copy()` (该方法本身，它将副本复制到现有的缓冲区)

+   `Uint8Array.prototype.slice(start, end)` (注意关键的区别！`TypedArray.slice()`会复制，而`Buffer.slice()`会创建视图。)

🚨警告

**关键混淆**：`TypedArray.prototype.slice()`创建一个副本，但`Buffer.prototype.slice()`创建一个视图。这是因为 Buffer 覆盖了 TypedArray 的切片方法。如果你意外地调用了 TypedArray 版本，你会得到相反的行为！

最后一点是一个陷阱。我见过它烧毁资深工程师。因为`Buffer`是`Uint8Array`，它继承了这两种方法，但`Buffer`覆盖了`slice()`以创建视图而不是复制。如果你直接在缓冲区上调用`Uint8Array`原型上的`slice`方法（通过`Uint8Array.prototype.slice.call(buf, ...)`），你会得到一个复制而不是视图。`Buffer.slice()`和`TypedArray.slice()`之间的这种不一致性是一个可能导致你精神崩溃的设计怪癖。Node.js 团队已经做了很多努力来确保`Buffer`的行为在内部保持一致，但与标准 TypedArrays 的根本区别仍然存在。

让我们来看一个区分至关重要的场景。你正在读取一个大文件，比如说一个 1GB 的视频文件，而你只需要解析前 1KB 的元数据。

```js
import { readFileSync } from "fs";
const videoBuffer = readFileSync("large_video.mp4"); // 1GB in memory
```

那个`readFileSync`调用阻塞了你的事件循环，同时 Node.js 从磁盘读取了 1GB。在底层，libuv 使用`open()`打开文件，使用`fstat()`获取文件大小，分配相应大小的缓冲区，然后使用单个`read()`系统调用（或对于非常大的文件，使用多个读取）读取整个文件。整个 1GB 被加载到一个单一的连续 ArrayBuffer 中。你的进程的 RSS 值增加了 1GB，操作系统甚至可能开始将其他进程交换到磁盘上以腾出空间。这种同步操作可以在慢速磁盘上冻结你的服务器几秒钟。

```js
// The WRONG way for long-term storage
const metadataView = videoBuffer.slice(0, 1024);
```

这样会创建一个 72 字节的 Buffer 对象，它持有对整个 1GB ArrayBuffer 的引用。视图的保留大小是 1GB，但其浅层大小仅为 72 字节。如果你将此传递给缓存、全局变量或任何长期数据结构，你刚刚就创建了一个内存泄漏。垃圾收集器看到引用链：你的缓存 → 元数据视图 → 视频缓冲区的 ArrayBuffer，并得出整个 1GB 必须保持活跃的结论。我调试过生产系统，其中数百个这样的小视图共同保留了数十 GB 的内存。

```js
// If we pass metadataView to another part of our app that holds onto it...
// we are keeping the entire 1GB videoBuffer in memory just for that 1KB view.
```

如果你需要长时间保留这些元数据，这里正确的做法是进行战略性的复制。

```js
import { readFileSync } from "fs";
const videoBuffer = readFileSync("large_video.mp4"); // 1GB in memory
 // The RIGHT way for long-term storage
const metadataCopy = Buffer.alloc(1024);
videoBuffer.copy(metadataCopy, 0, 0, 1024);
```

`Buffer.alloc(1024)`从 Node 的缓冲池中分配了正好 1024 字节（因为它的尺寸小于 4KB）。出于安全考虑，这段内存被清零。然后`copy()`操作触发一个高度优化的 C++中的`memcpy()`，在现代硬件上可以以数 GB/s 的速度移动数据。CPU 的 SIMD 指令每次复制 32 或 64 字节，这使得这个 1KB 的复制在微秒内完成。最重要的是，`metadataCopy`有一个独立的 ArrayBuffer，没有引用原始的 1GB 缓冲区。

```js
// Now, videoBuffer can be garbage collected as soon as it goes out of scope.
// We've spent a few microseconds copying 1KB to save 1GB of memory.
```

💡提示

经验法则：在函数内部使用视图进行临时处理。对于需要存储、缓存或传递给异步操作的数据，使用复制。复制的微小 CPU 成本可以防止巨大的内存泄漏。

这个决策框架——“这些数据是短期存在还是长期存在？”是有效使用视图和副本的关键。对于临时函数内处理，视图是你的最佳选择。对于需要存储、缓存或在应用程序的不同部分之间传递的数据，显式复制是你的保险政策，以防大规模内存泄漏。

## 复制语义和 Buffer.copy()

因此，我们已经确定有时你绝对需要副本。在 Node.js 中，这个主要工具是 `Buffer.prototype.copy()`。它是一个低级、高性能的方法，旨在成为 JavaScript 世界的 `memcpy`。它的签名是 `buf.copy(targetBuffer, targetStart, sourceStart, sourceEnd)`。

重要的是要注意，`copy()` 将写入一个 *现有* 的 `targetBuffer`。在调用它之前，你必须自己分配目标缓冲区。这给了你细粒度的控制，但也增加了处理步骤。

```js
const source = Buffer.from("abcdefghijklmnopqrstuvwxyz");
const target = Buffer.alloc(10);
```

`Buffer.from(string)` 将 26 个字符的字母表编码为 26 字节的 UTF-8（全部是 ASCII，所以每个字符一个字节）。Node 从其缓冲区池中分配它，因为它的尺寸小于 4KB。`Buffer.alloc(10)` 创建另一个小缓冲区，也来自池，但来自不同的偏移量。这两个缓冲区实际上可能是同一 8KB 池片的不同切片，但它们是非重叠的区域，具有独立的生命周期。

```js
// Copy the first 10 bytes from source into target.
source.copy(target, 0, 0, 10);
console.log(target.toString()); // 'abcdefghij'
```

这个 `copy()` 操作在 C++ 中会解析为单个 `memcpy(target_ptr + 0, source_ptr + 0, 10)` 调用。现代 CPU 使用 SIMD 指令进行优化，每次循环移动多个字节。对于如此小的缓冲区，操作可以在纳秒内完成。数据在物理上是复制的——对 `source` 的更改不会影响 `target`，反之亦然。

```js
// Copy 'klmno' from the source into the middle of the target.
source.copy(target, 3, 10, 15); // target, targetStart, sourceStart, sourceEnd
console.log(target.toString()); // 'abcklmnohij'
```

Node 的 C++ 内核对 `Buffer.copy()` 的性能进行了大量优化。对于在缓冲区之间复制数据，它几乎总是比你在 JavaScript 中编写的任何手动、逐字节循环要快。内存分析结果显示，所需时间与复制的字节数成正比，而常数因子非常低。

然而，有一种更方便的方法来创建副本，许多人都会选择：`Buffer.from(buffer)`。正如我们之前提到的，这个特定的 `Buffer.from()` 重载明确是一个复制操作。

```js
const original = Buffer.from("This is the original buffer");
// Create a new buffer with a copy of the original's data.
const clone = Buffer.from(original);
```

我们已经多次讨论了 `Buffer.from`，但——`Buffer.from(buffer)` 构造函数在其简单性上具有欺骗性。内部，它分配了一个与原始缓冲区大小完全相同的新 ArrayBuffer（这里为 28 字节），然后执行整个内容的 `memcpy()`。这发生在 Node 的 C++ 层，通过 `node::Buffer::Copy()` 函数。新的缓冲区是完全独立的——它有自己的后端存储，没有对原始缓冲区的引用。这对于内存隔离和防止我们一直在讨论的保留问题至关重要。

```js
clone.write("That"); // Modify the clone
console.log(original.toString()); // 'This is the original buffer'
console.log(clone.toString()); // 'That is the original buffer'
```

在内部，`Buffer.from(buffer)`基本上为你执行了一个`alloc`和一个`copy`。这是两步过程的语法糖。在大多数情况下，性能差异是可以忽略不计的，一行代码的便利性通常更胜一筹。然而，如果你处于一个极端的热路径中，需要重用现有的目标缓冲区以避免分配开销（一种称为缓冲区池化的技术），那么直接使用`Buffer.copy()`是最佳选择。

知道何时复制是一种艺术。科学在于知道如何做。规则很简单：如果数据需要比其原始的、庞大的父缓冲区存活更久，你必须给它一个新的家。分配一个所需的确切大小的新的缓冲区，并将数据复制到其中。这打破了与父缓冲区的链接，允许垃圾回收器完成其工作。

📌重要

`Buffer.copy()`需要一个预先分配的目标缓冲区。常见模式：`const copy = Buffer.alloc(size); source.copy(copy, 0, start, end);`。为了方便，可以使用`Buffer.from(source.subarray(start, end))`在一行中创建一个副本。

这是我们最终为我们的日志解析器实现的解决方案。我们不是存储`slice`，而是这样做：

```js
function getSessionId(logBuffer) {
 // Instead of a view, we make an explicit copy.
 const sessionId = Buffer.alloc(16);
 logBuffer.copy(sessionId, 0, 0, 16);
```

这种模式使我们分配了 16 个字节的内存，并且`memcpy()`操作需要几纳秒。`Buffer.alloc(16)`从 Node 的缓冲池（小于 4KB）中获取内存，并且出于安全考虑，内存被初始化为零。然后`copy()`操作从源中精确地移动 16 个字节。关键的区别是`sessionId`有一个自己的 ArrayBuffer，没有对`logBuffer`的引用。当这个函数返回并且`logBuffer`超出作用域时，整个多兆字节的缓冲区可以立即被垃圾回收。你的堆分析器将显示 16 字节的缓冲区，保留大小正好是 16 字节——这正是你所期望的。

```js
// Now, storing 'sessionId' retains only 16 bytes, not the whole logBuffer.
 return sessionId.toString("utf-8");
}
```

⚠️警告

永远不要在可能包含敏感数据的复制操作中使用`Buffer.allocUnsafe()`。未初始化的内存可能会暴露来自先前释放的缓冲区的密码、令牌或其他秘密。始终使用`Buffer.alloc()`来确保安全关键代码。

从`slice`到`alloc`+`copy`的一行更改为我们节省了数 GB 的 RAM。从表面上看，它可能看起来不太“高效”，因为它做了更多的工作（分配和复制），但在整个系统健康的大背景下，它要高效得多。

## SharedArrayBuffer 和跨线程视图

当我们引入 Node.js 工作线程时，情况变得更加复杂。长期以来，JavaScript 都是单线程的。如果你想要进行 CPU 密集型工作，你会阻塞主事件循环，你的应用程序的性能会降至停顿。工作线程改变了游戏规则，允许真正的并行性。但是，你如何在不需要昂贵的序列化和复制的情况下在线程之间共享数据？

答案是 `SharedArrayBuffer` (SAB)。普通的 `ArrayBuffer` 不能被多个线程访问。如果你传递一个给工作线程，就会创建一个副本。然而，`SharedArrayBuffer` 是一种特殊的 `ArrayBuffer` 类型，其底层的内存块可以被多个线程同时引用和操作。

⚠️警告

由于 Spectre 漏洞，`SharedArrayBuffer` 在浏览器中（2018-2020 年）被暂时禁用。虽然重新启用并加入了安全缓解措施，但仍需要谨慎处理。在 Node.js 中，始终使用 `Atomics` 操作来防止多线程场景中的竞态条件和数据损坏。

这是我们对视图理解变得强大的地方。你可以在主线程上创建一个 `SharedArrayBuffer`，将其传递给工作线程，然后两个线程都可以在相同的内存块上创建 `TypedArray` 或 `Buffer` 视图。

```js
// main.js
import { Worker } from "worker_threads";
 // Create a SharedArrayBuffer of 4 bytes.
const sab = new SharedArrayBuffer(4);
```

这分配了 4 字节的内存，可以被多个 JavaScript 上下文同时访问。与常规 `ArrayBuffer` 不同，这种内存使用平台特定的机制（POSIX 上的共享内存，Windows 上的内存映射文件）映射到多个地址空间。分配是页面对齐的，以支持原子操作。V8 会特别跟踪这一点 - 在垃圾回收期间，它不能移动或压缩这种内存，因为多个隔离区可能同时访问它。

```js
// Create a view over it on the main thread.
const mainThreadView = new Int32Array(sab);
mainThreadView[0] = 123; // Initial value
```

默认情况下，这个写入操作不是原子的。在 x86-64 上，32 位对齐的写入在硬件级别上是原子的，但 JavaScript 并不提供这样的保证。如果不使用 `Atomics.store()`，这个写入可能会被撕裂 - 另一个线程可能会看到部分写入的值。值 123 直接写入共享内存，没有任何同步原语，这意味着由于 CPU 缓存一致性延迟，无法保证其他线程何时会看到这个更新。

```js
const worker = new Worker("./worker.js");
worker.postMessage({ sab });
```

`postMessage` 不会复制 `SharedArrayBuffer` - 它传递了对相同内存的引用。现在两个线程都可以访问相同的 4 字节 RAM。这与常规 `ArrayBuffer` 消息传递有根本的不同，后者会克隆数据。工作线程获得自己的 Int32Array 视图，但它指向与主线程视图完全相同的内存页。

```js
worker.on("message", () => {
 console.log("Main thread sees:", mainThreadView[0]); // Output: 456
});
```

```js
// worker.js
import { parentPort } from "worker_threads";
 parentPort.on("message", ({ sab }) => {
 const workerView = new Int32Array(sab);
 console.log("Worker sees initial value:", workerView[0]); // Output: 123
```

工作线程立即看到主线程写入的值 123。但这并不保证在没有适当同步的情况下。由于 CPU 缓存一致性协议，一个线程写入和另一个线程看到更新之间可能会有延迟。在弱顺序内存架构（如 ARM）上，如果没有内存屏障，你可能根本看不到更新。

```js
// Modify the memory from the worker thread.
 workerView[0] = 456;
 parentPort.postMessage("done");
});
```

这非常强大，令人难以置信。我们只是在一条线程上修改了内存，并在另一条线程上立即看到了结果，没有任何复制和序列化开销。这是 Node.js 中高性能并行计算的基础。你可以有一个工作线程在大型数据集上执行复杂计算，而主线程则读取结果，这些结果一旦可用即可读取。

然而，这引入了一个全新的问题类别：竞争条件。由于两个线程可以同时读取和写入相同的内存，你需要同步原语来协调访问。这就是 `Atomics` 发挥作用的地方。`Atomics` 对象提供了在 `SharedArrayBuffer` 视图上执行原子读取、写入和读取-修改-写入操作的方法。这些操作保证在没有被另一个线程中断的情况下完成，从而防止数据损坏。

📌重要

没有使用 `Atomics`，`SharedArrayBuffer` 访问不是线程安全的。常规数组索引（`array[0] = value`）可能导致数据竞争。始终使用 `Atomics.store()`、`Atomics.load()` 和其他原子操作进行线程安全的访问。

使用 `SharedArrayBuffer` 是一种高级技术，它将并发编程的挑战直接带入你的 Node.js 应用程序。但理解它全部建立在相同的视图（`TypedArray`s）共享内存块（`SharedArrayBuffer`）的基础之上，就能揭开魔法的面纱。这与 `slice` 和 `subarray` 的原理相同，只是扩展到了线程边界之外。

## 记忆保留与垃圾回收

我们已经讨论了很多关于记忆保留的内容，但让我们正式化它。这是我们 10GB（假设）日志解析器泄露背后的机制。在像 JavaScript 这样的垃圾回收语言中，只要从“根”集合（例如，全局对象，当前调用栈）有可到达的引用，对象就会保留在内存中。

当你使用 `slice()` 或 `subarray()` 创建 `Buffer` 视图时，你创建了两个具有关系的对象。

1.  **视图对象** - 新的 `Buffer` 实例（`userIdSlice`）。它是在 V8 堆上的一个小对象。

1.  **父缓冲对象** - 原始的 `Buffer` (`massiveBuffer`)，它持有对大型外部 `ArrayBuffer` 的引用。

视图对象维护对其父缓冲区的内部引用。根据 V8 的内存模型，只要视图对象是可到达的，它的父缓冲区也被认为是可到达的。垃圾回收器看到从 `userIdSlice` 到 `massiveBuffer` 的引用，会说：“不行，还不能收集 `massiveBuffer`，还有人需要它。”它不知道你只关心 50MB 中的 16 字节。它只看到有效的引用并尊重它。

这就是为什么堆快照如此令人困惑。分析器正确地识别出 `userIdSlice` 对象很小。但它还有一个“保留大小”与“浅层大小”的概念。

+   **浅层大小** 是对象本身的大小。对于我们的切片，这非常小，只是 JavaScript 对象包装器几十字节。

+   **保留大小** 是所有仅因为该对象存在而被保留的内存的大小。对于我们的切片，保留大小非常大，因为它们是唯一阻止 50MB 父缓冲区被垃圾回收的东西。

堆快照显示 890MB 的切片保留了 10KB。它看起来像是一个会计错误，但这是视图语义的残酷真相。一旦我们理解了这一点，修复就显而易见了：我们必须切断我们所需的小数据片段与其庞大的父对象之间的联系。唯一的方法是进行复制。

```js
// Before: A view that retains the parent
function createView(parent) {
 return parent.slice(0, 10);
}
```

此函数返回一个视图，它保持对 `parent` 的 `ArrayBuffer` 的强引用。如果 `parent` 是 10MB，你的 10 字节视图将保持所有 10MB 的活跃状态。V8 垃圾收集器追踪引用链，并将整个父对象标记为可到达。这种模式是生产环境中 Node.js 应用程序中与 Buffer 相关的大多数内存泄漏的原因。

```js
// After: A copy that lets the parent be freed
function createCopy(parent) {
 return Buffer.from(parent.slice(0, 10));
}
```

`Buffer.from(buffer)` 构造函数调用是关键。它接受由 `slice()` 创建的 10 字节视图，分配一个新的 10 字节 `ArrayBuffer`，将数据复制到其中，并返回一个新的 `Buffer` 对象，该对象指向这个新的、小的分配。原始父缓冲区不再被返回的对象引用，由 `slice()` 创建的临时视图可以立即回收。这种模式，`Buffer.from(buf.slice(...))`，是创建大型缓冲区一小部分“修剪”副本的常见且有效的方法。它是基于视图的内存保留的解药。在经历了足够的生产事故后，你就能像鹰一样迅速发现缺失的副本。

## 基于视图的二进制协议解析

现在，让我们将这些概念应用到实际场景中：解析自定义二进制协议。这在高性能系统、物联网和游戏中很常见，在这些场景中，JSON 或 XML 的开销是不可接受的。二进制协议定义了字节序列中的数据严格布局。

例如，一条消息可能的结构如下：

+   字节 0-1：消息类型（Uint16）

+   字节 2-3：消息长度（Uint16）

+   字节 4：标志（Uint8）

+   字节 5-20：会话 ID（16 字节 UUID 字符串）

+   字节 21-结束：有效载荷（原始字节）

解析此内容的一个简单方法涉及大量的切片和复制。

```js
// Naive, copy-heavy parsing
function parseMessageWithCopies(buffer) {
 const messageType = buffer.slice(0, 2).readUInt16BE();
 const messageLength = buffer.slice(2, 4).readUInt16BE();
 const flags = buffer.slice(4, 5).readUInt8();
```

每行代码都创建一个临时视图，仅用于读取原始值。`slice(0, 2)` 创建一个缓冲区对象（堆上 72 字节），然后 `readUInt16BE()` 读取两个字节并将它们从大端转换为本地端序。视图立即被丢弃，但在 V8 分配、跟踪并最终垃圾回收它之前。每秒数千条消息，你正在无端地创建大量的 GC 压力。这些中间视图没有任何作用——你可以直接从原始缓冲区读取。

```js
const sessionId = buffer.slice(5, 21).toString("utf-8");
 const payload = buffer.slice(21); // This slice could be huge!
 return { messageType, messageLength, flags, sessionId, payload };
}
```

⚠️警告

上述模式为每条消息创建 5 个缓冲区视图。以每秒 1MB 的负载处理 1000 条消息将保留 1GB 的内存，即使你只需要 16 字节的会话 ID！

这段代码*工作*，但它为每条消息创建了五个新的 `Buffer` 对象。如果你每秒处理数千条消息，这些分配会累加起来，给垃圾收集器施加压力，并减慢你的应用程序。

另一方面，零拷贝方法利用视图读取数据，而不创建数据的副本。我们可以在主缓冲区上直接使用基于偏移量的读取方法，或者为更复杂的数据类型创建`TypedArray`视图。

```js
// Efficient, zero-copy parsing
function parseMessageWithViews(buffer) {
 const messageType = buffer.readUInt16BE(0); // Read directly from offset
 const messageLength = buffer.readUInt16BE(2);
 const flags = buffer.readUInt8(4);
```

这些直接读取操作非常快速。没有中间对象，没有分配，没有 GC 压力。`readUInt16BE()` 方法计算内存地址（缓冲区基础地址 + 偏移量），读取两个字节，并在优化的 C++代码中执行字节序转换。整个操作都保持在 CPU 缓存中。对于高频解析，创建视图然后读取与直接读取之间的差异可能意味着每秒 10,000 条和 100,000 条消息之间的差异。

```js
// For the session ID and payload, we create views
 const sessionIdView = buffer.subarray(5, 21);
 const payloadView = buffer.subarray(21);
 return { messageType, messageLength, flags, sessionIdView, payloadView };
}
```

📌重要

这个零拷贝版本速度快 10 倍，但返回保留整个父缓冲区的视图。明确记录：如果调用者需要将数据存储在立即处理作用域之外，调用者必须复制数据。

这个版本效率显著更高。它为原始数字类型不创建任何中间副本。它为会话 ID 和有效载荷创建了两个视图，但没有数据被重复。`sessionIdView`和`payloadView`是轻量级指针，指向原始消息缓冲区。

这是我们 TCP 服务中最终节省了 8GB RAM 的模式。我们使用视图是因为处理是临时的。如果我们需要长期存储`sessionIdView`或`payloadView`（例如，在缓存或请求映射中），我们就会回到内存保留陷阱。此类函数的契约应该是清晰的：它返回仅在处理立即作用域内有效的视图。如果调用者需要持久化该数据，它就是调用者的*责任*来执行复制。

这是对高性能库至关重要的设计模式。解析函数应执行零拷贝操作并返回视图。函数的消费者然后决定数据是短期的（直接使用视图）还是长期的（创建副本）。这分离了关注点，并将内存管理决策交给了具有最多上下文的代码。

## 平台字节序和 TypedArray 视图

当你处理来自网络或文件的二进制数据时，你无法避开字节序的概念。它指的是多字节数字（如 16 位或 32 位整数）在内存中存储的顺序。

ℹ️注意

如果位操作和位掩码仍然有点模糊，不要担心；我们将在后面的专门章节中深入探讨它们。现在，只需耐心等待。

+   **大端（BE）** - 最显著的字节首先。这在网络协议中很常见（通常称为“网络字节序”）。数字`0x12345678`将存储为`12 34 56 78`。

+   **小端（LE）** - 最不显著的字节首先。这是大多数现代 CPU 的本地格式，包括 Intel 和 AMD x86-64。相同的数字将存储为`78 56 34 12`。

忽略字节序问题在读取二进制协议时会导致数据完全混乱。Node.js 的 `Buffer` 提供了明确的方法来处理这个问题：`readUInt16BE`、`readUInt16LE`、`writeInt32BE` 等。当你知道你正在解析的数据的确切字节序时，这些方法是你的最佳选择。

但如果你直接在底层的 `ArrayBuffer` 上使用 `TypedArray` 视图呢？这就变得复杂了。`TypedArray`（如 `Int16Array`、`Float64Array`）使用宿主系统的本地字节序读取和写入数据。在我的 x86 笔记本电脑上，这是小端字节序。如果我在包含大端网络数据的缓冲区上创建一个 `Int16Array` 视图，我将读取垃圾数据。

```js
// A 16-bit integer, 258, in Big-Endian format is [0x01, 0x02]
const networkBuffer = Buffer.from([0x01, 0x02]);
 // Using the Buffer method correctly:
console.log(networkBuffer.readUInt16BE(0)); // 258, Correct!
```

`readUInt16BE()` 方法明确处理字节序转换。它读取位置 0 和 1 的字节，然后将它们组合为 `(buffer[0] << 8) | buffer[1]`，这可以正确地解释大端数据，无论平台字节序如何。当需要时，这发生在 Node 的 C++ 层，使用优化的字节交换指令，如 x86 上的 `bswap` 或 ARM 上的 `rev`。

```js
// Using a TypedArray view on a little-endian machine:
const int16View = new Int16Array(networkBuffer.buffer, networkBuffer.byteOffset, 1);
console.log(int16View[0]); // 513, Incorrect! (It read 0x0201)
```

🚨 警告

`TypedArray` 视图使用平台字节序（通常在 x86/ARM 上是小端字节序）。网络协议通常使用大端字节序。永远不要使用原始的 `TypedArray` 视图来处理网络数据 - 总是使用 `Buffer` 的 BE/LE 方法或带有明确字节序的 `DataView`。

这是一个即将发生的灾难。我们如何在使用通用 `TypedArray` 视图时控制字节序？答案是 `DataView` 对象。`DataView` 是一个低级接口，用于读取和写入 `ArrayBuffer` 中的数据，允许你为每个操作明确指定字节序。它比使用 `TypedArray` 更冗长，但它提供了绝对的控制。

```js
const arrayBuffer = new ArrayBuffer(4);
const dataView = new DataView(arrayBuffer);
 // Write a 32-bit integer in Big-Endian format
dataView.setInt32(0, 123456789, false); // false for big-endian
```

`setInt32()` 方法使用 `false` 将字节写入为 [0x07, 0x5B, 0xCD, 0x15] - 最高有效字节首先写入。`DataView` 内部处理字节序，无论平台字节序如何。在小端系统上，它在写入之前反转字节。在大端系统上，它直接写入。这个抽象层会消耗一些 CPU 周期，但确保了所有平台上的正确行为。

```js
// Read it back in Little-Endian format (will be wrong)
console.log(dataView.getInt32(0, true)); // Some garbage number
 // Read it back correctly in Big-Endian format
console.log(dataView.getInt32(0, false)); // 123456789
```

这个 `DataView` 类型转换看起来没问题，直到它破坏了一切。在我们的一个服务中，一位开发者使用 `Float32Array` 从网络流中快速解析一系列浮点数，假设主机字节序与网络字节序匹配。在他们的开发机器上运行正常。但当部署到具有不同字节序的不同云架构（这些天很罕见，但确实会发生）时，服务开始读取完全无意义的随机数据。修复方法是替换直接的 `Float32Array` 视图，使用循环和 `DataView` 来以正确、明确声明的字节序读取每个浮点数。这是一个痛苦的提醒，隐藏的执行环境假设是生产失败的配方。当有疑问时，要明确。使用 `Buffer` 的 `BE`/`LE` 方法或使用 `DataView`。

## 零拷贝的生产模式

经历了这些生产问题后，我的团队开发了一套经过充分验证的模式，用于处理缓冲区。这些不仅仅是理论上的最佳实践；它们是生产系统中必不可少的模式。

**模式 1：用于同步处理的临时视图**

这是零拷贝最常见且最安全的使用方式。当你需要在单个函数作用域内处理更大缓冲区的一个块时，视图是完美的。

```js
// Strategic view for temporary processing
function processChunk(largeBuffer, offset, length) {
 const view = largeBuffer.subarray(offset, offset + length);
 const result = performComplexCalculation(view);
```

这个模式是安全的，因为视图的寿命限制在函数执行范围内。视图被创建、使用，并在函数返回时变得不可达。V8 的逃逸分析通常可以进一步优化这一点——如果视图没有逃逸函数，它甚至可能不在堆上分配 Buffer 对象，而将所有内容保持在寄存器中。关键洞察：同步、函数作用域的视图几乎总是安全于保留问题。

```js
// Once the function returns, 'view' is eligible for GC.
 return result;
}
```

我们在这里使用视图是因为处理是临时的且同步的。视图不会超出函数的作用域。如果我们需要长期存储这个块或用于异步回调，我们会选择复制。这里就是为什么这个决定很重要的原因...

**模式 2：用于异步操作和存储的防御性复制**

任何需要跨异步边界或存储在集合中的缓冲区数据，你必须假设它需要被复制。原始缓冲区可能在你回调执行时被重用或垃圾回收。

```js
const longLivedCache = new Map();
 function processAndCache(dataBuffer) {
 const key = dataBuffer.subarray(0, 16); // Temporary view for the key
 const value = dataBuffer.subarray(16); // Temporary view for the value
```

这些视图是为了立即处理而创建的。它们很轻量级——在堆上每个视图只有 72 字节——但它们持有`dataBuffer`整个 ArrayBuffer 的引用。如果我们直接将这些视图存储在我们的缓存中，就会创建内存泄漏。整个`dataBuffer`将保留，直到缓存条目存在，这在生产系统中可能是几个小时或几天。

```js
// Before storing, we make a defensive copy.
 const storedValue = Buffer.from(value);
 // The key is converted to a string, which is implicitly a copy.
 longLivedCache.set(key.toString("hex"), storedValue);
}
```

在这里，我们创建视图以最初解析缓冲区。但当我们决定将`value`放入我们的`longLivedCache`时，我们立即创建一个副本。这确保我们的缓存条目是自包含的，并且不会意外地持有对更大的`dataBuffer`的引用。

**模式 3：解析器协议（输出视图，输入副本**）

这是库作者的模式。编写纯零拷贝的解析函数，并返回视图。明确记录返回的值是视图，如果原始缓冲区发生变化，它们可能会被无效化。

```js
/**
 * Parses a message header from a buffer.
 * WARNING: Returns a view into the original buffer. Do not store
 * the returned value long-term without creating a copy.
 * @param {Buffer} buffer The source buffer.
 * @returns {{id: Buffer, body: Buffer}} Views for id and body.
 */
function parseHeader(buffer) {
 return {
 id: buffer.subarray(0, 8),
 body: buffer.subarray(8),
 };
}
```

这个函数合约至关重要。JSDoc 明确警告返回的值是视图。这把内存管理决策权交给了调用者，调用者对数据寿命有更多的上下文。这个函数本身是纯的且快速的——除了为视图的两个小 Buffer 对象进行分配之外，没有其他分配。因为这个模式将最小必要的工作扩展到每秒数百万次操作。

```js
// Consumer of the function decides the memory strategy
const rawMessage = getMessageFromNetwork();
const { id, body } = parseHeader(rawMessage);
 // I need to use 'id' later, so I'll copy it.
const savedId = Buffer.from(id);
// I'm just logging the body, so the temporary view is fine.
logBodyPreview(body);
```

这个模式通过强制他们明确自己的意图，为可以立即处理数据的消费者提供最大性能，为需要存储数据的消费者提供最大安全性。

## 使用视图调试内存问题

当你怀疑与视图相关的内存泄漏时，你的主要工具是堆快照。你可以使用 Node.js 的 Chrome DevTools 或使用像`heapdump`这样的模块来生成这些快照。

该过程通常如下：

1.  当你的应用程序处于稳定、低内存状态时，进行堆快照。

1.  向你的应用程序施加你认为可能触发泄漏的负载。

1.  再次进行堆快照。

1.  在一段时间后再次进行第三次快照以确认增长趋势。

在快照查看器中，你将想要使用“比较”视图来查看快照之间分配了哪些对象。当调试我们的日志解析器时，我们看到了`Buffer`对象数量的巨大增加。

💡提示

使用`node --inspect-brk`与 Chrome DevTools 进行内存分析。关键的是“保留大小”列——它显示了每个对象保留的内存。寻找具有巨大保留大小的微小缓冲区——这是基于视图的泄漏的标志。

当你点击这些`Buffer`对象之一时，分析器将显示其属性。关键是寻找对父缓冲区的内部引用。在 Chrome DevTools 中，这通常在像`[[backing_store]]`这样的属性下显示，或者通过检查对象保留者。你会看到你那微小的 16 字节`Buffer`切片，在其保留链中，你会找到它所保持活的巨大多兆字节父`Buffer`。

另一种强大的技术是使用我们已多次讨论过的`process.memoryUsage()`。

💡提示

在 Node.js 13.9.0+中，使用`process.memoryUsage().arrayBuffers`来专门跟踪缓冲区内存。这比包括其他 C++分配的`external`更准确。

在我们的泄漏中，`heapUsed`缓慢增长，但`external`和`rss`却在爆炸性增长。这告诉我们泄漏不在标准 JavaScript 对象中，而是在 Node.js 管理的外部内存中——这是`Buffer`保留问题的经典特征。

在分析后，我们发现我们的视图在另一个服务中相互别名。我们有一个循环缓冲区实现，我们会通过创建视图来环绕。我们偏移逻辑中的错误导致新视图与旧视图略有重叠，无意中使旧视图（以及整个缓冲区）比预期存活得更久。堆快照是唯一可视化这些引用链的方法。

## 缓冲区操作的最佳实践

如果我能将所有这些痛苦和苦难浓缩成一套指导原则，那将是这些。

在经历了足够的生产事故后，你学会了在部署任何大量操作缓冲区的新代码之前**分析内存保留情况**。你不仅测试正确性，还测试负载下的内存行为。你开始使用**视图进行临时、同步处理**，但对于任何长期存在、异步或存储在集合中的数据，你都会寻求**显式复制**。你内化了视图与其底层缓冲区之间的父子关系，因为你已经在凌晨 3 点调试了替代方案。你**使用内存分析器进行测试**，因为你已经因为假设而多次被烧伤。

你**不懈地记录函数签名**。如果一个函数返回一个视图，你会在 JSDoc 注释中大声疾呼。你使下一个开发者不可能意外地误用你的 API 并创建泄漏。你学会了识别 `slice()` 或 `subarray()` 的代码异味，其结果被分配给具有更广作用域的变量，如对象属性或模块级变量。你看到这一点，就会立刻问，“那不应该是一个复制吗？”

最重要的是，你**对每个零拷贝操作都持怀疑态度**。你不将其视为免费性能提升；你将其视为一个具有重大风险的强大工具。你问自己，“我创建的数据的生命周期是什么？我引用的数据的生命周期是什么？”如果这两个生命周期不同，复制几乎总是正确的答案。

## 内存分析数据

这里是我们调查期间收集到的数据样本。测试创建了一个由单个 50MB 缓冲区派生出的 100,000 个小对象。

**测试场景 1：使用 `slice()`（创建视图**）

```js
const largeBuffer = Buffer.alloc(50 * 1024 * 1024);
const views = [];
for (let i = 0; i < 100000; i++) {
 views.push(largeBuffer.slice(0, 10));
}
// At this point, take a heap snapshot.
```

ℹ️注意

在 Node.js 22+的本地 TypeScript 支持下，你可以直接运行 TypeScript 缓冲区代码，无需转译。对于进行缓冲区操作的.ts 文件，使用 `node --experimental-strip-types`。

+   **`process.memoryUsage()` 输出：**

    +   `rss`: ~78 MB

    +   `heapUsed`: ~8 MB

    +   `external`: ~50.5 MB

+   **堆快照分析：**

    +   `views` 数组中所有 `Buffer` 对象的浅层大小：~800 KB（100,000 * ~8 字节/对象）

    +   保留大小：**~50 MB**。整个 `largeBuffer` 都被视图保留。

**测试场景 2：使用战略复制**

```js
const largeBuffer = Buffer.alloc(50 * 1024 * 1024);
const copies = [];
for (let i = 0; i < 100000; i++) {
 // Creating a copy for each item
 copies.push(Buffer.from(largeBuffer.slice(0, 10)));
}
```

每次迭代都会使用 `slice(0, 10)` 创建一个临时视图，然后立即使用 `Buffer.from()` 进行复制。临时视图在 `Buffer.from()` 完成后即可进行回收。复制拥有自己的 10 字节 ArrayBuffer，与 `largeBuffer` 无关。在循环结束后，我们拥有 100,000 个独立的 10 字节缓冲区，总共约 1MB 的内存，而 `largeBuffer` 可以被垃圾回收，释放 50MB。

```js
// At this point, 'largeBuffer' can be garbage collected.
```

+   **`process.memoryUsage()` 输出（触发垃圾回收后）：**

    +   `rss`: ~32 MB

    +   `heapUsed`: ~9 MB

    +   `external`: ~1.5 MB

+   **堆快照分析：**

    +   50MB 的 `largeBuffer` 已消失。

    +   `copies`数组包含 100,000 个小的`Buffer`对象，每个对象都有自己的 10 字节后端存储。总外部内存大约为 1MB（100,000 * 10 字节）加上一些开销。

这些测量清楚地量化了权衡。基于视图的方法在初期使用了更少的 CPU，但保留了它不需要的 50MB 内存。基于副本的方法在循环中使用了稍微更多的 CPU，但结果内存占用大大减小。

## 结束语

ℹ️注意

在 Node.js 22+的本地 TypeScript 支持下，你可以编写无需构建步骤的类型安全缓冲区操作。TypeScript 的类型系统可以在编译时帮助捕获缓冲区误用，从而防止本章讨论的许多运行时问题。

我仍然记得我的一个徒弟，他带着真正的好奇心问道：“那么我们为什么不在每个地方都使用副本呢？这似乎更安全。”这是一个合理的问题。答案是，真正的工程是关于做出明智的权衡。我们可以复制一切，我们的应用程序将更容易推理，但也会更慢、效率更低。我们可以拥有使用两倍 CPU 和内存的服务，在一个大规模系统中，这是你负担不起的成本。

目标不是害怕零拷贝操作，相反，我们应该尊重它们。这是要理解共享内存是一种既有强大优势又有严重风险的机制。当你创建一个视图时，你是在向运行时做出承诺——一个承诺，即你理解视图及其父视图的生命周期。

精通是关于理解每个调用的后果。这是关于查看`const view = buf.slice(0, 10)`，而不仅仅是看到一行代码，而是看到它创建的指向父缓冲区的内部引用，并问自己：“这是否是我准备管理的引用？”当你本能地回答这个问题时，你将永远不会以同样的方式看待内存。
