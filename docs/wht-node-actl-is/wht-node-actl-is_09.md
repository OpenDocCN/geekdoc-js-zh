# 缓冲区碎片化和挑战

> 原文：[`www.thenodebook.com/buffers/fragmentation-and-challenges`](https://www.thenodebook.com/buffers/fragmentation-and-challenges)

⚠️警告

你已经提前获得了这一章节的内容。你的反馈非常宝贵，请在下面的评论部分或[GitHub 讨论](https://github.com/ishtms/nodebook)中分享你的想法。

好的，我们已经覆盖了很多内容。你现在对`Buffer`有一个稳固的心理模型，知道它在内存中的位置，以及视图和副本之间的关键区别。你看到了零拷贝操作的原始力量，以及如果你不小心可能会造成的泄漏。我们讨论了内部缓冲池以及 Node.js 如何巧妙地优化小而频繁的分配，以避免频繁系统调用带来的性能惩罚。

本章是所有内容汇聚的地方。我们将深入探讨一个经典且低级的问题，大多数 JavaScript 开发者从未需要考虑：**内存碎片化**。这是一个感觉抽象的问题，直到它让你的生产服务器崩溃，即使你的监控仪表板显示你有足够的 RAM 可用。我们将剖析它是如何发生的，为什么会发生，以及 Node 的内存架构如何既帮助又有时阻碍这种情况。

然后，我们将转换方向。本章的第二部分，更大的部分，致力于一系列综合性的代码挑战。本章是关于将我们讨论的所有内容——从字节级解释和字节序到视图与副本的权衡以及缓冲池——应用到解决现实世界的问题。你将构建一个二进制协议解析器，分析内存使用情况以亲自查看泄漏，实现一个有状态的流处理器，甚至构建你自己的应用级缓冲池。

我不会在这里给你答案。阅读是一回事；做是另一回事。到本章结束时，你不仅会*知道*关于缓冲区的内容。你将拥有证明你可以在高性能生产环境中有效、安全、高效地使用它们的经验，而不仅仅是 Node.js，还包括任何其他抛给你的语言。

## 内存碎片化

💡提示

想要深入了解内存基础知识？请查看我的博客文章[内存：栈和堆](https://www.ishtms.com/blog/basic-system-concepts/memory-stack-heap)，其中我涵盖了从 RAM 和虚拟内存的工作原理，到栈帧和堆分配，缓存性能，常见的内存问题（泄漏、悬垂指针、碎片化），以及为什么不同的语言选择不同的内存管理策略。

内存碎片化是长期运行应用程序的隐形杀手之一。其核心概念很简单：随着时间的推移，你的应用程序的内存被分割成许多小而分散的块。尽管可用内存总量可能很大，但如果它被分散成成千上万的微小部分，它对于满足大型分配请求就毫无用处。你可能有一个 100MB 的可用 RAM 供你的进程使用，但如果请求一个单独的 1MB 缓冲区，请求可能会失败，因为没有一块连续的 1MB 可用内存。

要真正理解这一点，我们必须谈谈操作系统是如何将内存分配给 Node.js 进程的。

### 虚拟内存与物理内存

你的 Node.js 进程并不直接与你的计算机的物理 RAM 条交互。相反，它在一个 **虚拟地址空间** 内运行。这是操作系统为每个进程提供的庞大、连续的地址范围。在 64 位系统上，这个地址空间非常巨大——理论上 16 兆字节。这是一个干净、线性的抽象。当你的代码请求内存时，操作系统会在这个虚拟地址空间中找到一个空闲块并将其分配给你的进程。

在幕后，内存管理单元 (MMU)，这是你 CPU 中的一块硬件，与操作系统合作将这些虚拟地址映射到 RAM 中的实际物理地址。这种映射发生在称为 **页** 的块中，通常大小为 4KB。这个系统允许像将内存交换到磁盘上和防止进程相互干扰这样的魔法。

这里的重要启示是，当 Node.js 分配一个大缓冲区时，它是在向操作系统请求一个连续的 *虚拟* 内存块。然后操作系统有责任找到足够的空闲 *物理* 内存页来支持这个虚拟分配。

### 分配器的困境

当你调用 `Buffer.alloc(65536)` 来获取一个用于文件读取的 64KB 缓冲区时，Node.js 会绕过其内部的 8KB 池。它需要从系统中获取这些内存。它通过系统调用如 Linux/macOS 上的 `mmap` 或 Windows 上的 `VirtualAlloc` 来实现。系统的内存分配器（如 Linux 上的 `glibc` 的 `malloc`）在你的进程的虚拟地址空间中找到一个合适的 64KB 块并将其映射。

现在，你的代码处理文件，最终那个 64KB 缓冲区不再被引用。V8 垃圾收集器回收了 JavaScript 处理器，Node 的 C++ 层被通知释放底层内存。它调用 `munmap` 或 `free`，将那个 64KB 块返回给系统分配器。

问题始于你的应用程序多次使用不同大小的缓冲区进行这种操作。这种持续的分配和释放，尤其是不同大小的，消耗了你的内存空间。这就像拿一张整张纸，剪下一个 5 英寸的正方形，然后是一个 2 英寸的正方形，然后把 5 英寸的正方形放回去，然后剪下一个 3 英寸的正方形。过了一会儿，纸上满是洞。你可能还有足够的总纸张，但你无法剪出一个新的 6 英寸正方形。

这导致了两种类型的碎片化：**外部碎片化**和**内部碎片化**。

1.  **外部碎片化**是我们一直在描述的情况。总空闲内存足够，但它被分割成许多非连续的块（空洞）。新的分配请求失败，因为没有足够大的单个空洞来满足请求。这是对分配和释放许多大型、非池化缓冲区的应用程序的主要关注点。

1.  **内部碎片化**是一个更微妙的问题。它发生在内存以固定大小的块分配时，且一个分配请求被一个比请求更大的块满足。例如，如果一个分配器只处理 32、64 和 128 字节的块，而你请求 33 字节，它将给你一个 64 字节的块。剩余的 31 字节被浪费了。它们被分配了但未使用——这是你分配块内部的一个空洞。Node 的内部 8KB 缓冲池是这样一个可以导致内部碎片化的系统的完美例子。如果它从其 8KB 的 slab 中满足数百个 10 字节的请求，那么该 slab 的很大一部分可能会在分配之间的间隙中被“浪费”。然而，这是为了防止外部碎片化和减少系统调用开销而做出的有意识的权衡，并且通常是非常有效的。

初始状态

你的进程有一个大型的干净区域空闲虚拟内存。

Node.js 运行时 & V8 堆 F R E E M E M O R Y

为图像上传分配一个 1MB 的缓冲区（bufA）

运行时 bufA (1MB)F R E E

为视频块分配一个 512KB 的缓冲区（bufB）

运行时缓冲区 A (1MB)bufB (512KB)F R E E

图像处理已完成。释放 bufA。

运行时 F R E E (1MB)bufB (512KB)F R E E 查看现在的内存。我们创建了一个 1MB 的**空洞**。总空闲内存很大，但它被分割成两个非连续的块。

**一个新的请求到来，需要一个 1.2MB 的缓冲区来执行数据库转储。**

分配失败。尽管你有超过 1.2MB 的总空闲内存，但没有足够大的单个块来满足请求。这是外部碎片化的实际表现。在一个运行数天的真实服务器上，这个过程会重复数千次，使内存空间看起来像瑞士奶酪。最终，一个关键的分配失败，你的应用程序会因`ENOMEM`（内存不足）错误而崩溃。

### 我能做什么...？

当你处理比池大的缓冲区（默认情况下大于 4KB）时，碎片化的风险就会出现。如果你的应用程序分配和释放了许多大小不一的大型缓冲区，它就像一个混乱的内存客户端。这种碎片是逐渐分割你 Node.js 进程可用空闲内存的原因。

那么，你如何应对这个问题？你不能改变操作系统分配器的工作方式，但你可以通过改变你的应用程序*行为*来改变它。关键是减少内存碎片。

#### 缓冲区重用

这是减少分配 churn 的最强大技术。不是为每个任务分配一个新的缓冲区，而是预先分配一个单一、更大的缓冲区并重复使用它。这在代码的热路径中尤为重要，比如在网络的 `data` 事件处理程序内部或一个紧密的循环中。

让我们想象一个处理传入 TCP 数据包的服务器。每个数据包都需要用 4 字节长度头进行封装。

**不良、高 churn 方法**

```js
// BAD
// This code allocates two new buffers for every single data chunk,
// creating massive memory churn and risking fragmentation over time.
socket.on("data", (chunk) => {
 // Allocation #1: New header buffer
 const header = Buffer.alloc(4);
 header.writeUInt32BE(chunk.length, 0);
 // Allocation #2: Buffer.concat creates a new buffer and copies data
 const framedPacket = Buffer.concat([header, chunk]);
 sendToNextService(framedPacket);
});
```

如果这个服务器每秒处理 10,000 个数据包，那么每秒将有 20,000 个缓冲区分配（尽管 Node 的小缓冲池可能会优化其中一些）。垃圾收集器将超负荷工作，内存分配器将努力跟上，导致潜在的碎片化。

**更好的、可重用方法**

```js
// BETTER (but see "shared memorey hazard" section below!)
// A single, larger buffer is allocated once and reused.
const MAX_PACKET_SIZE = 65536; // 64KB
const reusableBuffer = Buffer.alloc(MAX_PACKET_SIZE);
 socket.on("data", (chunk) => {
 const framedPacketLength = chunk.length + 4;
 if (framedPacketLength > MAX_PACKET_SIZE) {
 console.error("Packet too large for reusable buffer!");
 return;
 }
 // No new backing buffer allocation. Write header into our existing buffer.
 reusableBuffer.writeUInt32BE(chunk.length, 0);
 // No new backing buffer allocation. Copy packet data after the header.
 chunk.copy(reusableBuffer, 4);
 // Create a view over the valid data. This creates a small Buffer wrapper
 // object but shares the underlying memory (zero-copy of bytes).
 const framedPacketView = reusableBuffer.subarray(0, framedPacketLength);
 sendToNextService(framedPacketView);
});
```

在这个版本中，我们消除了大后端内存分配（从每个数据包两个减少到零）。虽然我们确实为视图创建了一个小的 Buffer 包装对象，但我们已经移除了 `Buffer.concat` 执行的昂贵内存分配和复制。性能差异是显著的。

**共享内存风险**

上述优化有一个**严重问题**，如果不正确处理，可能会导致数据损坏。由 `subarray()` 创建的 `framedPacketView` 与 `reusableBuffer` 共享底层内存。

**如果 `sendToNextService` 是异步的**（这对于网络操作、排队系统或管道来说是典型的），并且你立即处理下一个数据包，你将在**前一个消费者仍在读取时覆盖缓冲区内容**。这会导致数据损坏。

只有当消费者在函数返回之前同步使用数据（很少见），或者你仔细协调缓冲区生命周期时，这种方法才是安全的。

**更安全的替代方案？缓冲池**

对于异步消费者，使用具有多个缓冲区的环形缓冲池：

```js
// SAFER: Ring buffer pool
const POOL_SIZE = 32; // Must be >= max in-flight packets
const MAX_PACKET_SIZE = 65536;
const pool = Array.from({ length: POOL_SIZE }, () => Buffer.alloc(MAX_PACKET_SIZE));
let poolIndex = 0;
 socket.on("data", (chunk) => {
 const framedPacketLength = chunk.length + 4;
 if (framedPacketLength > MAX_PACKET_SIZE) {
 console.error("Packet too large!");
 return;
 }
 // Get next buffer from pool (rotates through POOL_SIZE distinct buffers)
 const buf = pool[poolIndex];
 poolIndex = (poolIndex + 1) % POOL_SIZE;
 buf.writeUInt32BE(chunk.length, 0);
 chunk.copy(buf, 4);
 // Safe because we won't reuse this specific buffer until
 // we've cycled through all POOL_SIZE buffers
 const framedPacketView = buf.subarray(0, framedPacketLength);
 sendToNextService(framedPacketView);
});
```

池为每个在途数据包提供其自己的独立后端缓冲区，只要 `POOL_SIZE` 足够大以容纳你的最大并发操作，就可以防止覆盖。

所以，在（你很可能不会）遇到问题的前提下，你将如何决定何时使用哪种方法 -

+   **单一可重用缓冲区**仅当消费者真正同步时（非常罕见）

+   **缓冲池**用于具有有限并发性的异步消费者

+   如果无法限制在途工作，请**复制到新缓冲区**（例如，`Buffer.from(framedPacketView)`）- 这会为每个数据包分配一次，但简单且安全。

📌重要

Node.js 有一个用于小分配的内部缓冲池，因此微小的缓冲区可能已经从某些优化中受益。你仍然需要为 `chunk.copy()` 复制字节支付 CPU 成本 - 你是在交换分配成本和 CPU 复制成本（通常在 GC 敏感的热路径中是值得的）。

关键的收获是 - 缓冲区重用可以显著提高性能，但共享内存需要仔细的生命周期管理以避免损坏。

理解碎片化是将`Buffer.alloc()`不仅仅视为一个便宜的操作，而视为一个具有实际成本、在服务器生命周期内累积成本的操作。通过有意识地设计你的应用程序以通过重用和池化来减少这种 churn，你可以构建不仅速度快，而且稳定、有弹性的系统，可以在数月或数年内无问题地运行。

## 代码挑战

理论很重要，但没有什么能替代亲自动手。我已经创建了以下挑战，以便将前几章的概念应用到实际环境中。每个挑战都建立在上一个挑战的基础上，复杂性增加，并试图尽可能接近你在使用 Node.js 处理二进制数据时可能遇到的真实世界问题。

我不会提供解决方案。目标是让你自己构建它们。与代码斗争。查阅 Node.js 文档。你自己构建一个有效解决方案所获得的见解比从复制粘贴答案中获得的东西要宝贵得多。

让我们开始。

### 挑战 #1

想象你正在做一个物联网项目。一组传感器通过 TCP 将数据包发送到你的 Node.js 服务器。该协议简单且固定大小。每个数据包正好 24 字节长，具有以下结构：

| 偏移量（字节） | 长度（字节） | 数据类型 | 描述 |
| --- | --- | --- | --- |
| 0-3 | 4 | `UInt32BE` | 传感器 ID |
| 4-11 | 8 | `Float64BE` | 时间戳（Unix 纪元，毫秒） |
| 12-13 | 2 | `UInt16BE` | 传感器类型代码 |
| 14 | 1 | `UInt8` | 状态标志（位掩码） |
| 15 | 1 | `Int8` | 温度（°C） |
| 16-19 | 4 | `Float32BE` | 湿度（%） |
| 20-23 | 4 | `Float32BE` | 压力（kPa） |

#### 你的任务

编写一个名为`parseSensorData`的 Node.js 函数，该函数接受一个 24 字节的`Buffer`作为输入。该函数应根据上述规范解析缓冲区，并返回一个包含解码值的 JavaScript 对象。

使用此示例`Buffer`来测试您的函数。

```js
const samplePacket = Buffer.from([
 0x00,
 0x00,
 0x01,
 0xa4, // Sensor ID: 420
 0x41,
 0xd9,
 0x5c,
 0x38,
 0x2d,
 0x5b,
 0x81,
 0x24, // Timestamp: 1672531200000
 0x00,
 0x01, // Sensor Type: 1 (Thermometer)
 0x05, // Status Flags: 00000101 (Bit 0 and Bit 2 are set)
 0x19, // Temperature: 25°C
 0x42,
 0x48,
 0x00,
 0x00, // Humidity: 50.0
 0x42,
 0xc8,
 0x66,
 0x66, // Pressure: 100.2
]);
```

**目标**

你的`parseSensorData(samplePacket)`函数应该返回一个看起来像这样的对象：

```js
{
 "sensorId": 420,
 "timestamp": 1672531200000,
 "sensorType": 1,
 "statusFlags": 5,
 "temperature": 25,
 "humidity": 50,
 "pressure": 100.19999694824219
}
```

ℹ️注意

浮点精度可能在最后几位小数上造成轻微的变化，这是正常的。

**需要考虑的事项**

+   你需要哪些`Buffer`方法来处理每个字段？方法名非常具有描述性。

+   请密切关注数据类型（`UInt`、`Int`、`Float64`/`Double`、`Float32`/`Float`）和字节序（`BE` - 大端）。

+   每次读取的偏移量至关重要。这是一个固定大小的协议，因此偏移量是固定的。

+   良好的实践规定，在尝试解析之前，你应该验证输入缓冲区的长度。

### 挑战 #2

我们已经详细讨论了内存保留问题，其中一个小型的`Buffer`视图可以控制一个庞大的父缓冲区。现在是时候用代码来证明这一点了。

#### 你的任务

编写一个 Node.js 程序来演示和量化这种内存泄漏。该脚本应执行两个单独的测试：

1.  **“视图”测试**

    +   分配一个单个、大`Buffer`（例如，50MB）。

    +   在循环中，使用`buf.slice()`或`buf.subarray()`（首选）从这个大缓冲区创建大量小的*视图*（例如，每个 16 字节的 100,000 个视图）。

    +   将这些视图存储在数组中，以便它们不会被垃圾回收。

    +   循环结束后，使用`process.memoryUsage()`记录内存使用情况。请密切关注`external`属性。

1.  **"Copy"测试**

    +   分配一个大小相同的单个、大`Buffer`（例如，50MB）。

    +   在循环中，创建大量小的*副本*（例如，每个 16 字节的 100,000 个副本）。

    +   将这些副本存储在数组中。

    +   循环结束后，确保原始大缓冲区适用于垃圾回收，如果可能，调用 GC。

    +   再次记录内存使用情况。

**目标**

您的脚本输出应显示两次测试中`process.memoryUsage()`报告的`external`内存存在显著差异。 "View"测试的外部内存应略超过 50MB，而"Copy"测试的外部内存应小得多。

**需要考虑的事项**

+   您需要使用`--expose-gc`标志运行您的脚本，以便能够调用`global.gc()`。这使得结果更加确定。

+   为什么`process.memoryUsage()`中的`external`值是这个实验最重要的指标？`rss`和`heapUsed`代表什么？

+   复制的总大小为`100,000 * 16 字节 = 1.6MB`。您的复制测试结果应该在这个范围内。

+   一个将字节数格式化为 KB/MB 的辅助函数将使您的输出更容易阅读。

### 挑战#3

您在挑战#1 中使用的固定协议是成功的，但现在您需要处理一个更复杂、可变长度的协议。这在网络应用中很常见。您将解析使用类型-长度-值（TLV）编码格式化的消息流。

问题在于，您正在从 TCP 流中读取。数据可以以任意块的形式到达。单个`data`事件可能包含多个 TLV 消息，或者只是一个部分消息。您的解析器需要是状态的 - 它必须保留部分数据并等待下一块中到达的其余消息。

#### 协议规范

每个 TLV 消息都有一个 3 字节的头部，后面跟着一个可变长度的值。

| 偏移（字节） | 长度（字节） | 数据类型 | 描述 |
| --- | --- | --- | --- |
| 0 | 1 | `UInt8` | 消息**T**ype（1-255 之间的数字） |
| 1-2 | 2 | `UInt16BE` | 值部分的长度（以字节为单位，0-65535） |
| 3 到 3+L | L | `Buffer` | **V**alue（有效负载） |

**你的任务**

ℹ️注意

[NodeBook 的下一章涵盖了流](https://thenodebook.com/streams/intro-to-streams)。如果您还没有在 Node 中使用流或者对它们不熟悉，请随意跳过这个挑战。如果您想继续，请在尝试挑战之前阅读流的章节。简介性流的章节将在挑战上线之前发布。

创建一个扩展`stream.Transform`的`TlvParser`类。这个类将是你的解决方案的核心。它需要：

1.  维护一个用于不完整消息块的内部缓冲区。

1.  在其`_transform`方法中，将传入的数据追加到内部缓冲区。

1.  不断尝试从其内部缓冲区中解析完整的 TLV 消息。

1.  如果一条完整的消息被解析，它应该将一个 JavaScript 对象`{ type, value }`推送到下游。`value`应该是有效载荷缓冲区的副本。

1.  剩余未解析的数据必须保留在内部缓冲区中，以供下一个数据块使用。

**示例数据流**

数据将以块的形式到达。这里有一个示例序列 -

```js
// A full message - Type 1, Length 5, Value "hello"
const message1 = Buffer.from([0x01, 0x00, 0x05, 0x68, 0x65, 0x6c, 0x6c, 0x6f]);
 // A second message - Type 2, Length 8, Value "goodbye!"
const message2 = Buffer.from([0x02, 0x00, 0x08, 0x67, 0x6f, 0x6f, 0x64, 0x62, 0x79, 0x65, 0x21]);
 // Let's simulate a messy TCP stream by chunking the data weirdly.
const chunk1 = message1.subarray(0, 4); // Contains header and one byte of value
const chunk2 = Buffer.concat([message1.subarray(4), message2.subarray(0, 6)]); // Contains rest of msg1 and start of msg2
const chunk3 = message2.subarray(6); // Contains the rest of msg2
```

**目标**

当你将这些数据块通过你的`TlvParser`实例时，它应该发出两个`data`事件，按顺序产生这些对象 -

1.  `{ type: 1, value: <Buffer 68 65 6c 6c 6f> }`（值是"hello"）

1.  `{ type: 2, value: <Buffer 67 6f 6f 64 62 79 65 21> }`（值是"goodbye!"）

**需要考虑的事项**

+   你将如何管理你的内部缓冲区？`Buffer.concat`将是你的最佳朋友。

+   你的解析循环需要检查你是否有了足够的数据来读取头部（3 个字节），然后读取长度，然后检查你是否有了足够的数据来读取完整的值。

+   一旦一条消息成功解析，你如何从内部缓冲区中移除它以便解析下一条消息？`buf.subarray()`是这个工具。

+   为什么解析器输出值缓冲区的副本而不是其内部缓冲区的视图很重要？想想随着时间的推移内部缓冲区会发生什么。

### 挑战#4

你的视频处理服务正遭受内存碎片化。它不断分配和释放大（64KB）的缓冲区，运行几天后，由于内存不足错误而崩溃。你已经决定实现一个自定义的应用级缓冲池来减轻这种碎片化。

#### 你的任务

创建一个`BufferPool`类。这个类应该被设计用来管理一定数量的预分配的特定大小的缓冲区。

这个类必须具有以下功能 -

1.  **构造函数`(bufferSize, poolSize)`**：

    +   接收每个缓冲区的大小（例如，65536）和池中要保留的缓冲区数量（例如，100）。

    +   它应该预先分配所有这些缓冲区并将它们存储起来，可能在一个数组中。

1.  **方法`get()`**：

    +   如果池有可用的缓冲区，它应该返回一个。

    +   如果池为空，它应该记录一个警告并分配一个正确大小的新的临时缓冲区。这可以防止应用程序崩溃，但同时也表明池可能太小。

    +   它应该返回一个`Buffer`。

1.  **方法`release(buffer)`**：

    +   接收从池中之前获取的缓冲区。

    +   将缓冲区返回到池中，使其对下一个`get()`调用可用。

    +   它应该有一个检查来防止池的大小超过其初始大小（即，不要添加原本不在池中的缓冲区或当池为空时创建的额外缓冲区）。

1.  **属性`used`**：

    +   一个返回当前从池中借出的缓冲区数量的 getter。

**目标**

编写`BufferPool`类，然后编写一个小型模拟来测试它。模拟应该：

1.  创建一个池。

1.  从中获取几个缓冲区，检查`used`计数。

1.  将这些缓冲区释放回池中。

1.  通过尝试获取比池大小更多的缓冲区来测试“池为空”的条件。

1.  测试当池为空时创建的“额外”缓冲区的`release`逻辑。

**需要考虑的事项**

+   最好的数据结构来保存可用缓冲区是什么？一个具有`push()`和`pop()`操作的数组既简单又高效。

+   如何确保释放的缓冲区是有效的？你可以添加对其大小的检查，甚至以某种方式标记缓冲区，尽管这更高级。对于这个挑战，大小检查就足够了。

+   在一个现实世界的多线程应用程序（使用工作线程）中，你需要如何更改这个类以使其线程安全？（这是一个思想实验；你不需要为这个挑战实现它）。

+   当使用此池时，`try...finally`块是你的好朋友，可以确保即使在发生错误的情况下也能始终释放缓冲区。

### 挑战#5

你正在与一个使用奇怪二进制格式的旧硬件接口。它在同一数据包中混合了大端和小端字节顺序。`Buffer`的标准`read*BE()`和`read*LE()`方法很棒，但为了最大限度的清晰度和安全性，你决定使用`DataView`。

**协议规范**

数据包长度为 16 字节。

| 偏移量（字节） | 长度（字节） | 数据类型 | 顺序 | 描述 |
| --- | --- | --- | --- | --- |
| 0-1 | 2 | `UInt16` | **Big** | 数据包魔数（必须是`0xCAFE`） |
| 2-5 | 4 | `Int32` | **Little** | 设备 ID |
| 6-9 | 4 | `Float32` | **Big** | 电压读数 |
| 10 | 1 | `UInt8` | N/A | 状态码 |
| 11 | 1 | `UInt8` | N/A | 校验和 |
| 12-15 | 4 | `UInt32` | **Little** | 运行时间（秒） |

#### 你的任务

编写一个函数`parseLegacyPacket(buffer)`，它接受一个 16 字节的`Buffer`。在这个函数内部，你必须创建一个覆盖缓冲区底层`ArrayBuffer`的`DataView`。使用`DataView`方法（`getUint16`、`getInt32`、`getFloat32`等）根据规范解析数据包。记住，`DataView`方法接受一个可选的布尔参数来指定顺序（`true`为小端，`false`为大端）。

**样本数据**

```js
const legacyPacket = Buffer.from([
 0xca,
 0xfe, // Magic Number (BE)
 0xad,
 0xde,
 0x00,
 0x00, // Device ID: 57005 (LE)
 0x40,
 0xa0,
 0x00,
 0x00, // Voltage: 5.0 (BE)
 0x01, // Status: 1 (OK)
 0xb5, // Checksum
 0x80,
 0x51,
 0x01,
 0x00, // Uptime: 86400 (LE)
]);
```

**目标**

你的`parseLegacyPacket(legacyPacket)`函数应该返回一个看起来像这样的对象：

```js
{
 "magic": 60158,
 "deviceId": 57005,
 "voltage": 5,
 "status": 1,
 "checksum": 181,
 "uptime": 86400
}
```

**需要考虑的事项**

+   如何从一个`Buffer`获取底层的`ArrayBuffer`以创建`DataView`？每个`Buffer`实例都有一个`.buffer`属性。

+   注意`byteOffset`。如果`Buffer`是更大的`ArrayBuffer`的切片，`DataView`需要使用正确的偏移量来创建。对于这个挑战，你可以假设缓冲区不是切片，但了解`buf.byteOffset`属性是好的。

+   `DataView` 方法的第三个参数是字节序标志。`false`（或省略）是大端字节序。`true` 是小端字节序。你需要使用两者。

+   这是一个很好的练习，需要仔细、有系统地解析，因为每个字节及其解释都很重要。

### 挑战 #6（高级）

📌重要

这是一个高级的、可选的挑战。如果你之前没有使用过 Node.js 工作线程、共享内存或 `Atomics`，现在可以跳过这个挑战。在阅读完书中的相关章节后回来。我将在未来的章节中添加一个链接，作为提醒你完成这个挑战。

你有一个性能关键的应用程序，其中多个工作线程需要增加一个共享计数器。由于序列化开销，每次增加都需要在主线程之间传递消息将会太慢。你需要一种方式让所有线程可以直接且安全地访问和修改同一块内存。

#### 你的任务

编写一个脚本，演示使用 `SharedArrayBuffer` 和 `Atomics` 实现的线程安全计数器。

1.  **主脚本 (`main.js`)**

    +   创建一个足够大的 `SharedArrayBuffer` 来存储一个 32 位整数（4 字节）。

    +   在其上创建一个 `Int32Array` 视图。

    +   将计数器初始化为该内存位置的 0。

    +   创建两个 `Worker` 线程，并将 `SharedArrayBuffer` 传递给每个线程。

    +   每个工作线程将多次增加计数器（例如，1 百万次）。

    +   等待两个工作线程都发出完成信号。

    +   使用 `Atomics.load()` 从 `SharedArrayBuffer` 中读取最终值并打印出来。最终值应该是所有增加的总和（例如，200 万）。

1.  **工作脚本 (`worker.js`)**

    +   通过消息接收 `SharedArrayBuffer`。

    +   在共享缓冲区上创建自己的 `Int32Array` 视图。

    +   在一个紧密的循环中，使用 `Atomics.add()` 增加共享计数器。这是线程安全的关键。

    +   当循环完成后，向主线程发送一个 'done' 消息。

**目标**

主线程上的最终输出应该是 `Final counter value: 2000000`。如果你使用非原子操作如 `view[0]++`，由于竞争条件，你可能会得到小于 200 万的最终值，其中一个工作线程的读取-修改-写入周期会覆盖另一个的。

**需要考虑的事项**

+   这是唯一需要两个单独文件的挑战。

+   `SharedArrayBuffer` 是允许内存跨线程可见的核心组件。

+   为什么需要 `Atomics.add(view, 0, 1)` 而不是 `view[0]++`？研究在读取-修改-写入操作上下文中“竞争条件”是什么。

+   主线程如何知道两个工作线程都已完成？你可以使用 Promises 等待每个工作线程的 'done' 消息。`Promise.all` 是这个任务的理想工具。

+   这展示了在 Node.js 中在线程之间共享状态的绝对最低级和最高性能的方法，这是直接建立在我们所研究的内存原语之上的。
