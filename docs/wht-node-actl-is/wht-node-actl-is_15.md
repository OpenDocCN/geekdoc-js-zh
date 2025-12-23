# 零拷贝、分散/收集 I/O 与高级技术

> 原文：[`www.thenodebook.com/streams/zero-copy-scatter-gather`](https://www.thenodebook.com/streams/zero-copy-scatter-gather)

🚨注意

本章涵盖了高级性能优化技术。如果您在阅读时感到不舒服，请随时跳过此部分，稍后再回来。这里的技术在您处理大量数据并且已经通过分析确定 I/O 是瓶颈时最为重要。

每次您在 Node.js 中复制文件时，您很可能会复制相同的数据四次。首先从磁盘到内核内存。然后从内核内存到您的 Node.js 进程内存。然后从您的进程内存回到内核内存，为目的地。最后从内核内存到目标磁盘。对于本应概念上是一个操作的事情，需要四次复制。

这很重要，因为复制是昂贵的。每次复制都涉及 CPU 时间、内存带宽和缓存污染。当您通过流管道移动数 GB 的数据时，不必要的复制成为瓶颈。您的磁盘可能能够达到 2GB/sec 的速度，但您只能获得 500MB/sec，因为您正在花费 CPU 周期在内存中移动字节，而不是真正做有用的工作。

本章中涵盖的性能技术——零拷贝、分散/收集 I/O、缓冲区池化——不是学术上的好奇心。它们是流管道达到硬件极限与浪费资源在簿记上的区别。我们将探讨数据实际上是如何通过系统移动的，复制发生在哪里，以及如何消除它们。然后我们将探讨批量 I/O 操作的技术以减少系统调用开销，以及管理缓冲区以最小化垃圾收集压力的策略。

最后，您将了解如何使流更快，为什么它们一开始就慢，以及哪些优化对您的负载真正重要。

## 什么是零拷贝？

“零拷贝”这个术语被随意使用，它并不像听起来那么神奇。我们需要精确地了解它实际上意味着什么。

传统的 I/O 涉及在不同内存区域之间复制数据。您的操作系统在内核空间（操作系统内核运行的地方）和用户空间（您的应用程序运行的地方）之间保持严格的分离。这种分离对系统稳定性很重要——它防止应用程序损坏内核内存或相互干扰。

当你在 Node.js 中读取文件时，通常会发生以下情况。操作系统从磁盘读取数据到内核缓冲区。这是第一次复制——从磁盘到内核内存。然后，因为你的 Node.js 进程不能直接访问内核内存，操作系统将那些数据从内核缓冲区复制到你的进程内存空间。这是第二次复制。当你将那些数据写入另一个文件时，过程相反：你的进程将数据写入用户空间的一个缓冲区，操作系统将其复制到内核缓冲区（第三次复制），然后写入磁盘（第四次复制）。

四次复制。对于一个简单的文件复制操作。

每次复制操作都涉及几个昂贵的步骤。CPU 必须执行指令从内存的一个位置读取并写入另一个位置。这消耗了本可以用于实际计算的 CPU 周期。它还污染了 CPU 缓存——当你通过缓存复制兆字节的数据时，你会驱逐程序其他部分需要的有效数据，导致后续的缓存未命中。

内存带宽也是有限的。现代系统有非常快的内存，但 CPU 和 RAM 之间每秒可以移动的字节数仍然有限。当你多次复制相同的数据时，你会在相同的字节上重复消耗该带宽。当移动大量数据时，这成为瓶颈。

考虑一个服务器正在提供 1GB 的视频文件。使用传统的 I/O，这 1GB 的数据需要复制四次，但每次通过 CPU 介导的复制都涉及内存总线上的读写操作。实际的带宽分配如下：

+   磁盘到内核缓冲区：DMA 写入（1GB）

+   内核到用户空间：CPU 读取 + 写入（2GB）

+   用户空间到内核：CPU 读取 + 写入（2GB）

+   内核到网络：DMA 读取（1GB）

这消耗了 6GB 的内存带宽来服务 1GB 的实际数据。在一个能够达到 50GB/sec 内存带宽的系统上，这个单独的文件传输仅内存操作就消耗了 120ms，还不包括磁盘或网络 I/O 延迟。

零复制是一种消除部分或全部这些中间复制的技术。“零”是理想化的——你无法真正实现零复制，因为数据必须从一个地方移动到另一个地方——但你可以在内核和用户空间之间避免复制，这是大多数 CPU 开销发生的地方。

如果你只是将数据从一个文件移动到另一个文件，而不检查或修改它，为什么还要将其复制到进程的内存中呢？内核已经有了这些数据。它可以直接从源文件的内核缓冲区移动到目标文件的内核缓冲区，完全绕过你的进程。

Linux 上的`sendfile()`系统调用正是如此。你告诉内核“从文件描述符 A 复制数据到文件描述符 B”，它完全在内核空间内完成这一操作。无需复制到用户空间。无需在进程中进行字节移动的 CPU 周期。数据以尽可能少的复制次数从源流向目的地。

实现方式因操作系统而异。Linux 有`sendfile()`和`splice()`。FreeBSD 和 macOS 有略有不同的语义的`sendfile()`。Windows 有`TransmitFile()`。这些都是操作系统级别的原语，应用程序可以使用它们在特定场景中实现零拷贝传输。

在具有 DMA（直接内存访问）的现代系统中，这甚至更好。DMA 控制器可以在不涉及 CPU 的情况下在设备和内存之间传输数据。磁盘控制器从磁盘读取数据并将其直接写入内存。网络卡从内存读取数据并通过电线发送。CPU 只需设置传输，然后在 DMA 控制器处理实际数据移动的同时做其他工作。

当零拷贝完美工作时，CPU 的工作减少到设置传输。数据通过 DMA 和内核到内核的复制移动，从未触及用户空间，从未消耗 CPU 周期进行 memcpy 操作。CPU 通过系统调用启动传输，内核编程 DMA 控制器，数据直接从磁盘到网络（或文件到文件）流动，从未进入 CPU 缓存。

这是一个理想的情况。在实践中，仍然存在一些副本。磁盘可能首先读取到内核缓冲区，然后内核将（或映射）该缓冲区到网络卡的 DMA 区域。但与四次复制（磁盘 → 内核 → 用户 → 内核 → 目的地）相比，两次内核级别的复制是一个巨大的改进。

零拷贝仅在不需要检查或修改数据时才有效。当你需要转换数据——解析它、压缩它、加密它——时，你需要访问它。你不能要求内核“在写入之前压缩这些数据”（好吧，在某些具有内核级压缩的专用情况下你可以这样做，但它并不广泛适用）。你需要将数据复制到你的进程，转换它，然后再复制出来。

零拷贝在数据未受触碰时提供最大的吞吐量。当你需要处理数据时，你将回到传统的 I/O，并带有所有复制开销。这就是为什么零拷贝最适用于代理场景：一个 HTTP 代理转发请求，一个文件服务器提供静态文件，一个反向代理路由流量。在这些情况下，你只是在修改数据的情况下从一个套接字或文件移动到另一个。

此外，还有一个实际限制：零拷贝仅适用于某些源-目标组合。Linux 的`sendfile()`要求源是一个常规文件（支持`mmap()`的东西），而不是管道或套接字。在 Linux 上，你不能完全使用`sendfile()`进行套接字到套接字的传输 - 输入必须是一个文件。对于套接字到套接字的零拷贝，你需要使用带有中间管道的`splice()`，这更复杂来实现。

理解这些限制有助于你推断出何时你的流会很快（零拷贝适用）以及何时它们会较慢（回退到传统 I/O）。你不能使每个流都实现零拷贝——这是不可能的。但你可以构建你的管道，以便在可用时，高流量数据路径使用零拷贝。

让我们看看这如何应用于 Node.js 流。

## Node.js 流的零拷贝

一个常见的误解是，当你将文件管道到套接字时，Node.js 会自动使用像`sendfile()`这样的零拷贝技术。它不会。

当你编写如下代码时：

```js
const readable = createReadStream("largefile.mp4");
const socket = getSocketSomehow();
 readable.pipe(socket);
```

Node.js**不**会自动使用`sendfile()`系统调用。标准的流`pipe()`方法通过用户空间缓冲区使用常规的`read()`和`write()`操作读取数据。数据从磁盘流入内核缓冲区，然后进入你的 Node.js 进程内存，然后回到内核缓冲区，最后到达套接字。这就是我们之前讨论的传统四拷贝路径。

尽管你可能在其他地方看到过这样的说法，但 Node.js 流并不使用内核级别的零拷贝。这里实际上是真的：

**libuv 确实有`uv_fs_sendfile()`**，但它用于文件到文件的操作，如`fs.copyFile()`，而不是用于流管道。当你调用`fs.copyFile()`时，libuv 可以使用内核的高效文件复制机制。在 Linux 上，这可能会使用`sendfile()`或基于写时复制的 reflinks（带有`COPYFILE_FICLONE`标志），如果文件系统支持的话。但这只是文件到文件，不是文件到套接字。

**Node.js 流使用 JavaScript 进行缓冲 I/O**。当你`pipe()`时，Node 会设置事件监听器。可读流会发出带有 Buffer 块的`data`事件。可写流的`write()`方法会对每个块进行调用。这一切都在 JavaScript 中发生，数据通过 V8 的堆传递。这里没有神奇的内核绕过。

为什么 Node.js 不使用`sendfile()`进行文件到套接字流传输？有几个原因：

1.  `sendfile()`在 Linux、macOS 和 FreeBSD 上的行为不同。Windows 使用`TransmitFile()`，语义不同。抽象成本累加。

1.  **HTTPS 使事情变得复杂**。零拷贝需要数据在内核中无改动地流动。传统的 TLS 需要在用户空间中加密每个字节。Linux 4.13+引入了内核 TLS（kTLS），它可以通过在内核中处理加密来启用 TLS 流量的零拷贝。然而，Node.js 目前没有使用 kTLS，所以 HTTPS 流量仍然需要用户空间加密。由于大多数生产流量都是 HTTPS，Node.js 中内核级别零拷贝的实际好处有限。

1.  Node 的流反压机制依赖于 JavaScript 回调和`drain`事件。将其与内核级别的`sendfile()`集成是复杂的。

1.  Node.js 在其早期阶段短暂地支持了`sendfile()`，但由于 bug 和跨平台问题，在从 libeio 到 libuv 的过渡后被移除。

因此，你实际上在 Node.js 中何时会得到零拷贝的好处？有几个场景：

**用于文件到文件拷贝的`fs.copyFile()`。** 这使用 libuv 的`uv_fs_copyfile()`，它可以使用`sendfile()`或写时复制 reflinks：

```js
import { copyFile, constants } from "fs/promises";
 // Try copy-on-write first, fall back to sendfile-based copy
await copyFile(src, dest, constants.COPYFILE_FICLONE);
```

在支持 reflinks 的文件系统（如 Btrfs、XFS、APFS）上，这会创建一个即时副本，直到其中一个副本被修改，才会共享物理块。这才是真正的零拷贝。

**原生插件。** 如果你绝对需要`sendfile()`来进行文件到套接字的传输，你可以编写一个原生插件来直接调用它。这需要你自己处理部分写入、背压和平台差异。对于大多数应用来说，这样做很少值得。

**使用`respondWithFile()`的 HTTP/2。** `http2`模块有`http2stream.respondWithFile()`和`respondWithFD()`方法，这些方法针对文件服务进行了优化。这些方法仍然通过用户空间进行，但比手动流式传输更高效。

实际影响：不要期望代码会自动实现零拷贝魔法。相反，专注于你可以控制的优化——缓冲区大小、避免代码中的不必要拷贝，以及使用`_writev()`进行批处理。这些可以给你带来真实、可衡量的收益。

流抽象优先考虑正确性、灵活性和跨平台兼容性，而不是最大原始吞吐量。对于大多数应用来说，这是正确的权衡。在内核级别零拷贝至关重要的场景（如通过 HTTP 向数千个并发客户端提供大文件）中，最好由像 nginx 或 CDN 这样的专用软件来处理，而不是 Node.js。

即使没有绕过内核，零拷贝的概念仍然有用。你可以最小化自己的代码中的拷贝，这是我们接下来要关注的。

## 内存映射

值得了解的另一种零拷贝方法是内存映射。你不需要将文件读入缓冲区，而是可以直接将文件映射到你的进程地址空间。文件的内容将作为一个内存区域出现，你可以通过读取该内存区域来访问它。

内存映射使用操作系统的虚拟内存系统。内核将文件的页面映射到你的进程地址空间，但实际上直到你访问这些页面，它不会将文件加载到物理内存中。当你从映射的页面读取时，操作系统会从磁盘加载该页面。当你写入它时，操作系统将该页面标记为脏，并在稍后将其刷新回磁盘。

这在零拷贝的意义上，是因为你没有将文件数据显式地拷贝到一个单独的缓冲区。你是通过内存映射直接访问文件数据的。你对映射内存所做的更改会反映在文件中。

Node.js 核心没有暴露内存映射。没有内置的 JavaScript API 用于`mmap()`。要在 Node.js 中使用内存映射，你需要第三方原生插件。原始的`node-mmap`和`mmap-io`包大部分已经不再维护，并且可能无法与现代 Node.js 版本兼容。如果你需要 mmap 功能，请寻找活跃维护的分支，或者考虑是否真的需要内存映射——通常使用带有适当偏移量的`fs.read()`就足够好了。

内存映射对于对大文件的随机访问很有用。如果你需要读取多吉字节文件中的特定偏移量，将其映射到内存中可以让你像处理巨大的字节数组一样处理它。无需搜索和读取数据块 - 只需索引到映射区域。

但内存映射也有权衡。操作系统确实会对内存映射文件应用预读，但优化方式与`read()`不同。使用`read()`时，内核确切地知道你事先需要多少数据。使用 mmap 时，内核必须通过页面错误来检测你的访问模式 - 你访问一个页面，内核加载它并可能预取附近的页面。这种反应式方法为初始访问添加了延迟，而`read()`可以避免。对于纯顺序流，读取流通常更快。

对于流处理，内存映射很少有意义。对于随机访问工作负载（如数据库），它可以工作得很好。只需注意权衡并测量特定用例的性能。

## 避免在您的代码中不必要的缓冲区复制

即使 Node.js 无法使用操作系统级别的零复制，你仍然可以在自己的代码中避免不必要的复制。让我们谈谈引入额外复制的常见模式以及如何消除它们。

一个常见的罪魁祸首是`Buffer.concat()`：

```js
const chunks = [];
readable.on("data", (chunk) => {
 chunks.push(chunk);
});
 readable.on("end", () => {
 const combined = Buffer.concat(chunks);
 processData(combined);
});
```

这种模式将数据块收集到一个数组中，然后连接它们。`Buffer.concat()`调用分配一个新的缓冲区并将所有数据块复制到其中。这是一个额外的复制。

如果你打算将数据作为连续的缓冲区进行处理，这个复制是不可避免的。但如果你可以单独处理数据块，则可以跳过连接：

```js
readable.on("data", (chunk) => {
 processChunk(chunk);
});
```

按照到达的顺序处理每个数据块。没有中间缓冲区，没有连接复制。

另一个反模式是将缓冲区转换为字符串然后再转换回来：

```js
const str = buffer.toString("utf8");
const processedStr = processString(str);
const newBuffer = Buffer.from(processedStr, "utf8");
```

每次转换都会分配新的内存并复制数据。如果你的处理可以直接在缓冲区上进行（使用`buffer.indexOf()`、`buffer.subarray()`等），请避免字符串转换。

当正确使用时，缓冲区切片是一种零复制操作：

```js
const slice = buffer.subarray(10, 50);
```

这不会复制数据。它从字节 10 到字节 50 创建原始缓冲区的视图。切片与原始缓冲区共享底层内存。修改切片会修改原始缓冲区。这是零复制切片。

注意：使用`subarray()`而不是`slice()`。`Buffer.slice()`具有相同的行为，但它已被弃用（DEP0158），因为它与`TypedArray.prototype.slice()`不一致，后者会创建一个副本。从 Node.js v25 开始，调用`slice()`会发出运行时弃用警告。请使用`subarray()`代替 - 它是推荐的 API，不会触发警告。

但如果你修改原始缓冲区的长度或重新分配它，切片就变得无效了。如果你从许多大缓冲区中切出微小的片段并保持这些切片存活，你将阻止大缓冲区被垃圾回收。小的切片持有对大缓冲区的引用，使其保持在内存中。

安全模式：切片进行临时处理，然后如果需要保留数据就进行复制：

```js
const slice = buffer.subarray(10, 50);
processTemporarily(slice);
 // If you need to keep it long-term:
const copy = Buffer.from(slice);
// Now the original buffer can be GC'd
```

另一个模式：除非你明确需要复制，否则请避免使用`Buffer.from(buffer)`。如果你只需要将缓冲区传递给另一个函数，请传递原始缓冲区：

```js
// Unnecessary copy:
const copy = Buffer.from(originalBuffer);
writeStream.write(copy);
 // Just use the original:
writeStream.write(originalBuffer);
```

可写流不会修改缓冲区（除非它是一个非常不寻常的流），因此没有必要复制它。

每个`Buffer.concat()`、`Buffer.from()`和`buffer.toString()`都可能分配和复制。只有在你的代码语义需要时才执行这些操作，而不是出于习惯。并且请记住，使用`subarray()`而不是`slice()`来获取零拷贝视图。

## 散列/收集 I/O

散列/收集 I/O 是一种在处理多个缓冲区时减少系统调用次数的技术。

在传统的 I/O 中，如果你想将三个缓冲区写入文件，你需要进行三次写入系统调用：

```js
fs.writeSync(fd, buffer1);
fs.writeSync(fd, buffer2);
fs.writeSync(fd, buffer3);
```

每个系统调用都有开销。CPU 从用户模式切换到内核模式，内核处理请求，然后 CPU 切换回用户模式。对于小写入，这个开销可能超过实际的 I/O 成本。

让我们量化这个开销。系统调用通常仅需要 50-200 纳秒来切换模式，这取决于 CPU 和操作系统。然后是内核处理请求的工作——验证参数、设置 I/O 等。对于简单的写入，这可能是另外 100-500 纳秒。因此，每个系统调用在发生任何实际数据移动之前的基础成本大约为 150-700 纳秒。

如果你正在写入三个 1KB 的缓冲区，那么就是三个系统调用，所以纯开销为 450-2100 纳秒。在快速的 SSD 上实际写入 3KB 可能只需要几百纳秒。系统调用开销可能超过 I/O 成本。

现在将这个概念放大。如果你正在编写 1000 个小缓冲区，你将花费 150-700 微秒仅仅用于系统调用开销。这可能听起来不多，但在一个每秒处理数百万次写入的高吞吐量系统中，这个开销累积起来就是实际的 CPU 时间。

散列/收集 I/O 允许你将多个缓冲区传递给单个系统调用。对于写入（收集），你从多个缓冲区收集数据并在一个操作中写入。对于读取（散列），你将传入数据散列到多个缓冲区中。

在 Linux 上，`writev()`系统调用处理收集写入。你传递一个缓冲区数组（技术上是一个指向缓冲区的 iovec 结构数组的数组），内核在一个操作中写入所有这些缓冲区。这比单独的写入调用要高效得多。

散列的对应操作是`readv()`，它将数据读取到多个缓冲区中。你指定一个缓冲区数组，内核按顺序填充它们。如果读取的数据多于第一个缓冲区能容纳的，它将继续写入第二个缓冲区，依此类推。当你知道传入数据的结构时，这很有用——例如，一个固定大小的头部后跟可变长度的有效负载。你可以在一个系统调用中将数据散列读取到头部缓冲区和有效负载缓冲区中。

Node.js 通过可写流上的 `_writev()` 方法公开收集写入，但流 API 中没有直接公开分散读取，因为流抽象事先不知道要将多少缓冲区分散到。然而，如果你需要，较低级别的 `fs.readv()` 和 `fs.writev()` 函数是可用的。

Node.js 的可写流通过 `_writev()` 方法公开这一点。如果你正在实现自定义可写流并重写 `_writev()`，Node 将批量写入，并使用一个包含块的数组调用你的 `_writev()` 而不是多次调用 `_write()`。

这是它的工作原理。当一个可写流被封堵或多个块快速写入时，Node 将它们缓冲。如果流实现了 `_writev()`，Node 会用所有缓冲的块调用它：

```js
class BatchWriter extends Writable {
 _writev(chunks, callback) {
 // chunks is an array: [{ chunk, encoding }, ...]
 const buffers = chunks.map(({ chunk }) => chunk);
 // Write all buffers in one syscall
 fs.writev(this.fd, buffers, (err) => {
 callback(err);
 });
 }
}
```

`chunks` 数组包含具有 `chunk` 和 `encoding` 属性的对象。你提取块并将它们传递给支持向量 I/O 的系统调用。

优点：对于 N 个块进行 N 次系统调用，你只需进行一次系统调用。对于写入许多小块（如带有许多小写入的 HTTP 响应）的流，这可以大大减少系统调用开销。

但有一个细微差别。Node 只在多个块被缓冲时调用 `_writev()`。如果块缓慢到达（一次一个，中间有间隔），Node 会为每个块调用 `_write()`。为了强制批处理，你可以封堵流：

```js
writable.cork();
writable.write(chunk1);
writable.write(chunk2);
writable.write(chunk3);
writable.uncork(); // Flushes as one _writev() call
```

封堵抑制 `_write()` 调用并缓冲一切。解堵刷新缓冲区，使用所有缓冲的块调用 `_writev()`。

当你知道你即将写入多个块时，这很有用。在写入之前封堵，写入之后解堵，即使 `_write()` 通常会被调用，你也能得到批处理。

对于分散读取（相反的方向），Node 没有直接公开 `readv()`，因为可读流按需拉取数据，并且不清楚要将多少缓冲区分散到。但是，如果你使用低级 `fs` API，你可以使用 `fs.readv()` 在一次系统调用中将数据读取到多个缓冲区中。

将多个 I/O 操作批量到一个系统调用中可以减少开销。分散/收集 I/O 是使用多个缓冲区执行此操作的机制。

## 实现用于最大吞吐量的 `_writev()`

这是一个实现 `_writev()` 以优化可写流的示例。

假设你正在向网络套接字写入，并希望最小化系统调用。套接字的默认行为为每个块调用 `write()`。如果块很小，你会进行许多系统调用。

通过实现 `_writev()`，你批量写入：

```js
class BatchedSocket extends Writable {
 constructor(socket, options) {
 super(options);
 this.socket = socket;
 }
 _write(chunk, encoding, callback) {
 this.socket.write(chunk, callback);
 }
 _writev(chunks, callback) {
 const buffers = chunks.map((c) => c.chunk);
 const combined = Buffer.concat(buffers);
 this.socket.write(combined, callback);
 }
}
```

这个 `_writev()` 连接缓冲区，这确实引入了复制。但如果替代方案是许多小系统调用，这仍然是一个净赢。

这里是权衡。没有 `_writev()`，你需要对 N 个块进行 N 次系统调用。系统调用很昂贵。使用可以连接的 `_writev()`，你只需进行一次复制（连接）和一次系统调用。如果系统调用开销超过复制开销，批处理就赢了。

但你可以做得更好。如果你的底层 I/O 机制支持真正的向量写入（如文件描述符上的 `writev()`），你可以避免连接：

```js
_writev(chunks, callback) {
 const buffers = chunks.map((c) => c.chunk);
 fs.writev(this.fd, buffers, (err) => {
 callback(err);
 });
}
```

现在你正在将多个缓冲区传递给向量 I/O 系统调用。内核将它们全部写入，而不需要你将它们复制到一个单独的缓冲区中。这是真正的聚集 I/O。

对于套接字，Node 的 net.Socket 在 JavaScript 级别不暴露向量写入，但 libuv（Node 的基础）在支持的地方内部使用它们。通过实现`_writev()`并让套接字的内部写入处理批处理，你可以从 libuv 的优化中受益。

关键是始终在目标支持批处理写入时实现`_writev()`。即使你必须连接缓冲区，这通常也比多个系统调用更快。

另一个模式：自适应批处理。如果数据块很大，批处理帮助不大。如果数据块很小，批处理非常重要。你可以跟踪数据块大小和动态地 cork/uncork：

```js
let pendingWrites = 0;
let isCorked = false;
 function writeWithBatching(chunk) {
 pendingWrites++;
 if (pendingWrites === 1 && chunk.length < 4096 && !isCorked) {
 writable.cork();
 isCorked = true;
 }
 writable.write(chunk, (err) => {
 pendingWrites = Math.max(0, pendingWrites - 1);
 if (pendingWrites === 0 && isCorked) {
 writable.uncork();
 isCorked = false;
 }
 });
}
```

这在第一次小写入时 cork，并在写入稳定后 uncork。使用写入回调确保即使在某些写入失败的情况下也能 uncork。`Math.max(0, ...)`防止计数器在发生意外情况时变为负数。

这是一个简化的启发式方法。在生产中，你会更仔细地跟踪时间和数据块大小，并为流关闭/错误事件添加清理。但这个想法是稳固的：批处理小写入，对于大写入则跳过批处理。

## 缓冲区池

每次你分配一个缓冲区，你都在请求 V8 分配内存。每次分配都会被垃圾收集器跟踪。当缓冲区不再被引用时，GC 会回收它们。频繁的分配会创建 GC 压力——垃圾收集器必须更频繁地运行，暂停你的应用程序以回收内存。

对于高吞吐量流，缓冲区分配开销可能成为瓶颈。如果你在处理大量数据时使用小块数据，你可能会分配数百万个缓冲区，每个缓冲区都会创建 GC 工作。

理解其机制在这里很重要。当你使用`Buffer.alloc(size)`分配缓冲区时，V8 不仅仅给你内存。它必须找到一个空闲的内存区域（如果内存碎片化可能会触发 GC），初始化元数据以跟踪分配，可能为零内存以保障安全，并将缓冲区链接到其跟踪结构中。

对于一个小缓冲区（比如 1KB），这个分配可能需要 500-2000 纳秒，具体取决于堆状态。如果你每秒处理 100,000 个数据块，每个数据块分配一个缓冲区，那么你每秒仅在分配开销上就会花费 50-200 毫秒——5-20%的 CPU 时间浪费在簿记上。

当这些缓冲区变得不可达时，垃圾收集器必须回收它们。V8 的 GC 有多个代（年轻代、老年代），对象根据年龄在代之间移动。存活多个 GC 周期的缓冲区会被提升到老年代，这更昂贵且难以收集。频繁的缓冲区分配和释放会破坏 GC，导致更长的暂停时间和更频繁的暂停。

缓冲区池化是一种重复使用缓冲区以避免重复分配的技术。而不是为每个数据块分配一个新的缓冲区，你预先分配一个缓冲区池并重复使用它们。

核心思想很简单：在启动时分配 N 个缓冲区，将它们保存在池中，并在需要时分发。当一个缓冲区不再需要时，将其返回到池中而不是让它被垃圾回收。这把重复分配转换成了简单的池管理——从数组中弹出和推入，这比 GC 操作快得多。

困难之处在于管理缓冲区的生命周期。只有当你确定缓冲区不再被其他任何地方引用时，才能重复使用缓冲区。如果你在某个其他代码仍然持有引用时将缓冲区返回到池中，那么当缓冲区被重复使用时，该代码将看到其数据被损坏。这是一个在 JavaScript 中而不是 C 中的使用后释放的漏洞。

安全的方法是只将具有明确、范围有限的缓冲区池化。例如，用于单个 I/O 操作的缓冲区：读取到缓冲区中，立即处理，然后返回池中。只要处理过程中不保留对缓冲区的引用，就可以安全地重复使用。

这里是最简单的形式：一个可重复使用的缓冲区：

```js
const reusableBuffer = Buffer.allocUnsafe(65536);
 readable.on("data", (chunk) => {
 chunk.copy(reusableBuffer, 0, 0, chunk.length);
 processBuffer(reusableBuffer.subarray(0, chunk.length));
});
```

你分配一个 64KB 的缓冲区。对于每个传入的数据块，你将其复制到可重复使用的缓冲区中，处理它，然后为下一个数据块重复使用该缓冲区。

这消除了每个数据块的分配。你只分配一次，重复使用多次，减少 GC 压力。

但有一个重要的细节：`Buffer.allocUnsafe()`。这分配了一个缓冲区而不对其内存进行零化。正常的`Buffer.alloc()`会零化缓冲区，这确保了没有来自先前使用的遗留数据。零化会消耗 CPU 周期。如果你即将覆盖整个缓冲区，零化是浪费的工作。

`Buffer.allocUnsafe()`跳过了零化。缓冲区包含在该内存区域之前的数据。这更快但更危险。如果你没有覆盖整个缓冲区，你可能会从先前的操作中泄露敏感数据。

安全模式：当你立即覆盖缓冲区时使用`Buffer.allocUnsafe()`，并将缓冲区切割到实际数据长度以避免暴露未初始化的内存：

```js
const buf = Buffer.allocUnsafe(1024);
const bytesRead = readDataInto(buf);
const safeSlice = buf.subarray(0, bytesRead);
```

切片只包含你写入的字节。缓冲区的其余部分（可能包含垃圾数据）不会被暴露。

为了更灵活的池化，维护一个缓冲区池并检查它们：

```js
class BufferPool {
 constructor(bufferSize, poolSize) {
 this.bufferSize = bufferSize;
 this.pool = [];
 for (let i = 0; i < poolSize; i++) {
 this.pool.push(Buffer.allocUnsafe(bufferSize));
 }
 }
 acquire() {
 return this.pool.pop() || Buffer.allocUnsafe(this.bufferSize);
 }
 release(buffer) {
 if (this.pool.length < 100) {
 this.pool.push(buffer);
 }
 }
}
```

你预先分配一个缓冲区池。当你需要缓冲区时，从池中取出一个。当你完成时，将其返回到池中。如果池为空，则分配一个新的缓冲区（回退到正常分配）。如果池太大（为了避免内存囤积），则丢弃返回的缓冲区而不是将它们池化。

**安全警告**：这个基本池在释放时不会将缓冲区清零。如果你的应用程序处理敏感数据（密码、令牌、PII），则之前的内容将保留在内存中，可能会泄露给后续使用相同缓冲区的用户。对于安全敏感的应用程序，在释放前将缓冲区清零（`buffer.fill(0)`）或者不要将包含敏感数据的缓冲区池化。

这大大减少了分配。你不再按块分配，而是预先分配一个小池，并重复使用这些缓冲区数千次。

在可读的流中使用它：

```js
const pool = new BufferPool(16384, 10);
 class PooledReadable extends Readable {
 _read(size) {
 const buffer = pool.acquire();
 readDataInto(buffer, (err, bytesRead) => {
 if (err) {
 pool.release(buffer);
 this.destroy(err);
 } else if (bytesRead === 0) {
 pool.release(buffer);
 this.push(null);
 } else {
 const data = Buffer.from(buffer.subarray(0, bytesRead));
 this.push(data);
 pool.release(buffer);
 }
 });
 }
}
```

你获取一个缓冲区，用它进行读取，使用 `Buffer.from()` 复制相关部分，推送那个副本，并将原始缓冲区释放回池。该缓冲区将被用于下一次读取。

这里要注意使用 `Buffer.from()` 的显式复制。`subarray()` 创建了一个与原始缓冲区共享内存的视图 - 它不会复制数据。如果你直接推送子数组并释放缓冲区，当池重新使用该缓冲区进行另一个读取时，下游消费者可能会看到损坏的数据。在释放池化缓冲区之前始终进行复制。

你仍然控制着分配发生的位置。你不再需要在每次 `_read()` 中分配一个缓冲区（V8 必须管理这些缓冲区），而是分配你读取的字节的小副本。池化缓冲区很大且可重复使用，从而减轻了分配器的压力。

对于真正的零复制池化，你需要直接将池缓冲区传递给下游，并确保在消费后释放它们。这意味着需要与消费者协调，这会变得很复杂。在实践中，当你可以控制数据流的两个端点时（如自定义协议实现），池化最有用。

缓冲区池化减少了分配，从而减少了 GC 压力。对于即将覆盖的缓冲区使用 `Buffer.allocUnsafe()`，并且在使用切片时要小心，以避免泄露未初始化的数据。

## 批量写入以提高效率

我们之前提到了使用 cork/uncork 来批量写入。现在我们将探讨何时以及如何策略性地使用它。

Corking a writable stream tells it to buffer writes instead of flushing them immediately. Uncorking flushes the buffered writes, ideally in a single batched operation (via `_writev()` if implemented).

优点：减少写操作的数量。代价：增加延迟（数据在缓冲区中等待直到 uncork）。

关键在于知道何时 cork。如果你即将写入一系列小块，则在爆发之前 cork，在爆发之后 uncork：

```js
writable.cork();
for (const item of items) {
 writable.write(processItem(item));
}
writable.uncork();
```

所有写入都会缓冲，然后一起刷新。如果实现了 `_writev()`，它们会在一个系统调用中写入。

但这种模式存在一个问题：如果 `processItem()` 抛出异常，`uncork()` 永远不会被调用，流保持 corked 状态。始终在 try/finally 中成对使用 cork 和 uncork：

```js
writable.cork();
try {
 // ... writes ...
} finally {
 writable.uncork();
}
```

`finally` 块确保即使发生错误也会 uncork。

一个细微之处：Node 允许多次调用 cork。每次 cork 都会增加一个内部计数器。每次 uncork 都会减少它。只有当计数器达到零时，流才会刷新：

```js
writable.cork(); // counter = 1
writable.cork(); // counter = 2
writable.uncork(); // counter = 1, no flush
writable.uncork(); // counter = 0, flush
```

这让你可以嵌套封堵区域。最内层的封堵/解封不会触发刷新；只有最外层的解封才会。

这对于复杂的控制流很有用，其中多个函数可能会封堵/解封：

```js
function writeHeader(writable) {
 writable.cork();
 writable.write(header);
 writable.uncork();
}
 function writeBody(writable) {
 writable.cork();
 for (const chunk of chunks) {
 writable.write(chunk);
 }
 writable.uncork();
}
 writable.cork();
writeHeader(writable); // Nested cork/uncork
writeBody(writable); // Nested cork/uncork
writable.uncork(); // Final flush
```

`writeHeader` 和 `writeBody` 都会封堵/解封，但它们嵌套在外部封堵中，因此它们的解封不会刷新。最后的解封刷新一切。

不应该封堵的情况：如果写入已经很大（兆字节），封堵没有帮助。缓冲的开销可能超过任何批处理的好处。封堵对于许多小写入最有用。

此外，不要无限期地封堵。如果你正在处理长时间运行的流并在开始时封堵，数据缓冲区会一直积累到末尾。这会消耗内存并增加延迟。只在写入爆发时封堵，而不是整个流的整个生命周期。

自适应封堵的模式：当写入频繁时封堵，当写入变慢时解封：

```js
let lastWrite = Date.now();
let corked = false;
let uncorkTimer = null;
 function adaptiveWrite(chunk) {
 const now = Date.now();
 if (now - lastWrite < 10 && !corked) {
 writable.cork();
 corked = true;
 }
 writable.write(chunk);
 lastWrite = now;
 if (uncorkTimer) clearTimeout(uncorkTimer);
 uncorkTimer = setTimeout(() => {
 if (corked) {
 writable.uncork();
 corked = false;
 }
 uncorkTimer = null;
 }, 10);
}
```

如果写入在彼此之间 10 毫秒内发生，则封堵。如果 10 毫秒内没有写入，则解封。这会将快速写入批处理在一起，当写入变慢时及时刷新。注意，我们在每次写入时都会清除并重置计时器——如果没有这样做，快速写入会积累许多挂起的计时器。

这是一个启发式方法。正确的阈值取决于你的工作负载。测量和调整。

## 避免在流中字符串连接的开销

当累积大量文本时，JavaScript 中的字符串连接可能会造成内存效率低下。现代引擎如 V8 使用“Cons 字符串”（也称为 rope）来优化连接——而不是立即复制，它们创建一个指向原始字符串的树结构。实际的复制发生在字符串被读取或扁平化时。

但这种优化有局限性。Cons 字符串为树结构增加了内存开销，并且扁平化发生得不可预测——当你访问一个字符、搜索字符串或将它传递给原生代码时。在流场景中，当你累积许多块时，这些延迟的复制最终还是会发生，而维护深层 Cons 字符串树的内存开销可能会导致问题。

在流中，有问题的模式看起来像这样：

```js
let text = "";
readable.on("data", (chunk) => {
 text += chunk.toString();
});
```

每个 `+=` 要么创建一个 Cons 字符串（延迟成本）要么触发之前 Cons 字符串的扁平化（即时成本）。对于包含许多块的文件，你最终会得到一个深度树，最终以昂贵的成本扁平化，或者接近 O(N²)行为的重复扁平化操作。

修复方法：使用数组收集块，然后一次性连接：

```js
const chunks = [];
readable.on("data", (chunk) => {
 chunks.push(chunk.toString());
});
 readable.on("end", () => {
 const text = chunks.join("");
 processText(text);
});
```

`Array.push()` 成本较低。`Array.join()` 一次分配并复制所有字符串。线性时间而不是二次方时间。

更好的是，如果你正在处理缓冲区，收集缓冲区并在最后连接：

```js
const buffers = [];
readable.on("data", (chunk) => {
 buffers.push(chunk);
});
 readable.on("end", () => {
 const combined = Buffer.concat(buffers);
 processBuffer(combined);
});
```

`Buffer.concat()` 一次分配并复制所有缓冲区。避免在实际上需要字符串之前使用 `toString()`。

如果你需要增量处理数据（而不是累积整个流），直接处理块：

```js
readable.on("data", (chunk) => {
 processChunk(chunk); // No accumulation
});
```

没有连接，没有累积，只是按块处理。

另一个反模式：将缓冲区转换为字符串进行简单操作：

```js
const str = buffer.toString();
if (str.includes("keyword")) {
 // ...
}
```

缓冲区有用于搜索的方法：

```js
if (buffer.indexOf("keyword") !== -1) {
 // ...
}
```

没有转换，没有字符串分配，只是缓冲区扫描。

字符串是不可变的，连接操作会创建新的字符串。对于流，尽可能减少转换和连接操作。当需要累积时，尽可能使用缓冲区，并在数组中收集块。

## 流的 `read(0)` 漏洞

一个不为人知但偶尔有用的技巧：在可读流上调用 `read(0)`。

通常，`read(size)` 从流的内部缓冲区中拉取 `size` 字节。但 `read(0)` 不会拉取任何数据。相反，它触发流的内部缓冲区检查，如果满足某些条件，可能会调用 `_read()`。

只有当 **两个** 条件都满足时，`_read()` 才会被调用：

1.  内部缓冲区低于 `highWaterMark`

1.  流当前不在 `_read()` 调用过程中

这在你在暂停模式下使用流并希望启动缓冲区填充时很有用：

```js
readable.pause();
 // Later, trigger buffer fill without consuming data:
readable.read(0);
```

调用 `read(0)` 告诉流“检查是否需要读取更多数据”，但实际上并没有消耗任何内容。如果缓冲区低且没有进行读取操作，则会调用 `_read()` 以开始填充缓冲区。

一个重要的注意事项：`_read()` 实现几乎总是异步的。它们在 I/O 完成后稍后调用 `this.push()`。所以 `read(0)` 仅仅是 *启动* 一个读取请求；它不会同步填充缓冲区。如果你需要数据可用，你必须等待异步操作完成：

```js
readable.pause();
setupResources();
readable.read(0); // Initiates async buffer fill
 // Wrong: buffer likely empty here, _read() hasn't completed yet
// readable.resume();
 // Right: wait for data to be available
readable.once('readable', () => {
 readable.resume(); // Now data is actually available
});
```

这是一个小众优化。大多数代码不需要它。但如果你正在编写低级流管道，并且需要精细控制何时调用 `_read()`，`read(0)` 就是这个工具。

对于调试：如果 `read(0)` 没有触发 `_read()`，这并不一定意味着有错误。它可能意味着缓冲区已经达到或超过 `highWaterMark`，或者之前的一个 `_read()` 操作仍在进行中。将 `readable.readableLength` 与流的 `highWaterMark` 进行比较，以了解当前状态。

将此视为一种高级技术。对于正常的流使用，你不会接触到 `read(0)`。但它属于流 API 的一部分，了解它的存在可以在深入了解内部时有所帮助。

## 避免中间转换

管道中的每个转换都会增加开销。数据通过转换的缓冲区，调用 `_transform()` 方法，然后再次缓冲输出。对于具有许多阶段的复杂管道，这种开销会累积。

如果你可以将转换组合起来，你就可以减少阶段并提高性能：

```js
// Slower: three transforms
pipeline(
 source,
 toUpperCase,
 removeWhitespace,
 trimLines,
 dest
);
 // Faster: one combined transform
pipeline(
 source,
 allInOne, // Does all three transformations
 dest
);
```

组合转换在一次遍历中完成相同的工作。减少缓冲，减少方法调用，减少开销。

代价是模块化。独立的转换更容易测试和重用。组合转换是一个单体。根据你的优先级选择：性能与可维护性。

如果性能很重要，并且你有许多简单转换的管道，考虑将它们组合。如果可维护性很重要，并且转换在多个管道中重复使用，则保持它们分开。

一种模式：懒组合。为了清晰起见，开始时使用单独的转换。如果分析显示管道开销很高，则组合热点路径：

```js
const combined = new Transform({
 transform(chunk, encoding, callback) {
 let result = toUpperCase(chunk);
 result = removeWhitespace(result);
 result = trimLines(result);
 this.push(result);
 callback();
 },
});
```

你已经封装了组合逻辑，管道少了一个阶段。

另一种优化：消除无操作转换。有时在特定条件下转换会不改变数据：

```js
if (shouldTransform) {
 pipeline(source, transform, dest);
} else {
 pipeline(source, dest);
}
```

如果转换不需要，就完全跳过它。不要包含一个“以防万一”的旁路转换。

每个管道阶段都有成本。当性能很重要时，尽量减少阶段。谨慎地组合转换以平衡速度和代码清晰度。

## 优化 readable.readableFlowing 以进行手动控制

`readable.readableFlowing`属性告诉你流是否处于流动模式（true）、暂停模式（false）或尚未设置（null）。

你可以用这个来手动控制流。假设你正在消费一个流，并且你想根据外部条件暂停：

```js
readable.on("data", (chunk) => {
 processChunk(chunk);
 if (shouldPause()) {
 readable.pause();
 }
});
 // Later, resume based on readableFlowing:
if (readable.readableFlowing === false) {
 readable.resume();
}
```

在调用`resume()`之前检查`readableFlowing`可以避免重复的恢复调用。如果流已经流动，`resume()`不会做任何事情，但先检查可以节省一个方法调用。

这属于微优化领域，但在处理数百万个块时紧密循环中，小的节省会累积起来。

另一种模式：根据流状态有条件地切换模式：

```js
if (readable.readableFlowing === null) {
 // Stream hasn't been started, use flowing mode
 readable.on("data", handler);
} else if (readable.readableFlowing === false) {
 // Stream is paused, resume it
 readable.resume();
}
```

这让你能够适应流的当前状态，而不做假设。

`readableFlowing`属性对于调试也很有用。如果一个流没有发出数据，检查`readableFlowing`。如果它是`false`，则流已暂停。如果它是`null`，则还没有触发流动模式的监听器。如果它是`true`，则流正在流动，问题在其他地方。

对于大多数代码，你不会直接操作`readableFlowing`。但在构建或调试流系统时，它是一个方便的内省工具。

## 性能分析

所有这些优化——零拷贝、缓冲区池、塞住/拔塞——只有在它们提高你的工作负载性能时才有意义。唯一知道的方法是测量。

从基线开始。在不进行优化的情况下运行你的流管道并测量吞吐量：

```js
const start = Date.now();
let bytes = 0;
 source.on("data", (chunk) => {
 bytes += chunk.length;
});
 source.on("end", () => {
 const duration = (Date.now() - start) / 1000;
 const throughput = bytes / duration / 1024 / 1024;
 console.log(`Baseline: ${throughput.toFixed(2)} MB/s`);
});
```

记录基线吞吐量。然后一次应用一个优化并重新测量。

例如，实现`_writev()`：

```js
class OptimizedWriter extends Writable {
 _writev(chunks, callback) {
 // ... batching logic ...
 }
}
```

使用优化后的写入器运行管道并测量吞吐量。如果它更高，则优化有帮助。如果相同或更低，则没有帮助（或者它不是瓶颈）。

也测量内存使用：

```js
setInterval(() => {
 const mem = process.memoryUsage();
 console.log(`Heap: ${(mem.heapUsed / 1024 / 1024).toFixed(2)} MB`);
}, 1000);
```

如果缓冲区池减少了堆的使用而没有损害吞吐量，那就是胜利。如果它减少的吞吐量比节省的内存多，那就没有意义。

使用 Node 的内置分析器找到热点：

```js
node --prof script.js
node --prof-process isolate-*.log > profile.txt
```

寻找在缓冲区操作、系统调用或转换方法中花费的时间。如果 50%的时间花在`Buffer.concat()`上，那就是你的瓶颈。优化这一点。

关键是要对你的实际工作负载进行性能分析，而不是合成基准测试。如果你正在处理 JSON，用 JSON 进行测试。如果你正在提供文件服务，用实际文件大小进行测试。有助于小块优化的优化可能对大块有害，反之亦然。

记住：过早优化是万恶之源。在测量之后优化，而不是在之前。许多“显而易见”的优化在实践中并没有帮助。

## 真实世界的性能模式

在完整的示例之前，我们需要确定每种优化技术何时在生产系统中提供实际价值。

**在高速率、低转换场景中，零拷贝概念很重要。** 如果你正在构建静态文件服务器、视频流服务或 HTTP 代理，最小化拷贝是至关重要的。然而，请记住，Node.js 流不会自动使用像`sendfile()`这样的内核级零拷贝。我们讨论过的优化 - 避免使用`Buffer.concat()`，使用`subarray()`进行视图，实现`_writev()` - 有助于减少用户空间中的拷贝。对于 CDN 规模（每秒 10,000+请求的大型文件）的真正内核级零拷贝，通常比 Node.js 更适合的是像 nginx 这样的专用服务器。

但如果你正在构建一个返回 JSON 响应的 API（通常小于 100KB），零拷贝就帮不上忙了。响应很小，它们是动态生成的（需要转换），并且不通过文件到套接字管道流动。在这里，传统的 I/O 是合适的，优化努力应该集中在其他地方 - 比如 JSON 序列化或数据库查询。

**散列/聚集 I/O（writev）在有许多小写操作时很重要。** HTTP 响应头就是一个经典的例子。你写下状态行，多个头部行，然后是主体 - 可能是数十个小块。没有 writev()，那就是数十个系统调用。有了 writev()，就只有一个或几个。对于高流量的 HTTP 服务器，这可以通过减少系统调用开销来提高响应延迟 10-30%。

但如果你的写操作已经很大（比如从文件中流式传输 64KB 块），writev()就帮不上忙了。每个块仍然是在一个系统调用中写入的。你节省的开销与实际的 I/O 成本相比微不足道。

**在分配率极端时，缓冲区池化是有益的。** 如果你正在处理具有许多小消息的二进制协议（例如网络数据包解析、物联网传感器数据、金融交易数据），你可能每秒分配数百万个小缓冲区。池化可以将 GC 暂停时间减少 50-80%，将一个难以维持 10,000 msg/sec 的系统转变为一个可以轻松处理 100,000 msg/sec 的系统。

但如果你的流处理以适中的速率分配缓冲区（比如，对于一个典型的 Web 应用，每秒 1,000 个），池化不会提供可测量的好处。垃圾收集器可以轻松处理那个速率，而池化管理复杂性是不合理的。

**Cork/uncork 优化有助于突发写入模式。** 如果你的应用程序处理批处理作业 - 从数据库中读取 1000 条记录，转换它们，并写入结果 - 在批处理周围进行 corking 可以带来明显的优势。你可能将写操作从 1000 减少到 10-50，从而大幅提高吞吐量。

但对于稳态流式传输（如跟踪日志文件并转发行），cork/uncork 没有帮助。数据持续流动，批处理不会减少总的 I/O 操作，它只是延迟了它们。在某些情况下，它甚至可能在没有提高吞吐量的情况下增加延迟。

这里的模式很清晰：首先测量你的工作负载特征，然后应用匹配这些特征的优化。不要为了你没有的场景进行优化。

决策框架：

1.  **在现实负载下分析你的应用程序。** 在 Linux 上使用 `perf`，在 macOS 上使用 Instruments，或在 Node 的内置分析器中进行分析。确定 CPU 时间实际上花在了哪里。如果你看到高 CPU 使用率但磁盘/网络利用率低，CPU 负载（复制、系统调用）是你的瓶颈。如果磁盘/网络饱和且 CPU 低，I/O 吞吐量是你的限制，而不是 CPU 负载。

1.  **测量分配率和垃圾回收影响。** 使用 `process.memoryUsage()` 并跟踪堆增长。使用 `--trace-gc` 来查看垃圾回收暂停时间。如果分配极端（多 MB/秒）且垃圾回收暂停时间长（`>10ms`），缓冲区池化可能有所帮助。如果分配适度且垃圾回收暂停时间短（`<1ms`），池化不会提供任何好处。

1.  **计算你的系统调用次数。** 在 Linux 上使用 `strace` 或在 macOS 上使用 `dtruss` 来查看你的应用程序执行了多少系统调用。如果你看到预期只有几个大写()调用时出现了成千上万的微小写()调用，writev() 或 cork/uncork 可以有所帮助。如果你看到主要是大 I/O 操作，批处理就不会很重要。

1.  **在有和没有优化的情况下进行基准测试。** 知道优化是否有帮助的唯一方法是进行测量。实现优化，基准测试吞吐量和延迟，并与基线进行比较。如果吞吐量提高了 20%，而延迟没有增加，那么就是胜利。如果没有变化或性能下降，则删除优化。

这种有系统的方法可以防止过早优化。你可能认为缓冲区池化会有帮助，但分析表明分配并不是瓶颈。你可能认为零拷贝是必要的，但你正在转换每一个字节，而且无论如何都不能使用它。先测量，然后优化。

## 实际示例：优化文件复制管道

这里是一个结合了这些技术的优化文件复制管道的示例。

我们将使用更大的缓冲区以提高吞吐量，使用 `_writev()` 进行批处理，以及高效的缓冲区处理来复制一个大文件：

```js
import { createReadStream, createWriteStream } from "fs";
import { pipeline } from "stream/promises";
import { Writable } from "stream";
 const pool = new BufferPool(65536, 10);
 class OptimizedWriter extends Writable {
 constructor(dest, options) {
 super(options);
 this.dest = dest;
 }
 _write(chunk, encoding, callback) {
 this.dest.write(chunk, callback);
 }
 _writev(chunks, callback) {
 const buffers = chunks.map((c) => c.chunk);
 const combined = Buffer.concat(buffers);
 this.dest.write(combined, callback);
 }
}
 async function optimizedCopy(src, dest) {
 const reader = createReadStream(src, {
 highWaterMark: 65536,
 });
 const writer = new OptimizedWriter(
 createWriteStream(dest, { highWaterMark: 65536 })
 );
 await pipeline(reader, writer);
}
 await optimizedCopy("input.dat", "output.dat");
```

此管道使用 64KB 缓冲区并实现了 `_writev()` 以将多个块批处理为单个写操作。请注意，64KB 实际上是 Node.js 22 的 `fs.createReadStream()` 和 `fs.createWriteStream()` 的默认值 - 我们在这里明确指出以增强清晰度，但对于文件流，你可以完全省略 `highWaterMark`。基本的 `stream.Readable` 类仍然默认为 16KB。

对于不需要处理数据的文件到文件复制，考虑直接使用 `fs.copyFile()`，这可以使用操作系统级别的优化：

```js
import { copyFile, constants } from "fs/promises";
 // Use copy-on-write if filesystem supports it
await copyFile(src, dest, constants.COPYFILE_FICLONE);
```

关键要点：结合技术。使用更大的缓冲区以实现高吞吐量，实现 `_writev()` 以进行批处理，并在不需要流处理时使用 `fs.copyFile()`。

## 测量和调试技术

确定你的优化是否起作用需要不仅仅是查看吞吐量数字。以下是一些深入挖掘的方法。

**系统调用跟踪** 可以显示你的优化如何影响系统调用。在 Linux 上，使用带时间戳的 `strace`：

```js
strace -c -f node your-script.js
```

`-c` 标志显示系统调用计数的摘要。如果你实现了 `_writev()` 但仍然看到数百个单独的 `write()` 调用，那么有问题 - 可能是流没有被塞住，或者块没有被缓冲。如果你看到数十个与你的数据块操作数相同的 `writev()` 调用，则优化正在起作用。

对于更多细节，跟踪特定的系统调用：

```js
strace -e write,writev -f node your-script.js
```

这显示了每个带有参数和返回值的写和 writev 调用。你可以验证 writev() 是否接收多个缓冲区，而不是一个。

使用 **perf** 进行 **CPU 分析** 可以显示 CPU 时间花费在哪里。在 Linux 上：

```js
perf record -F 99 -g node your-script.js
perf report
```

这每秒采样调用栈 99 次，并显示哪些函数消耗了最多的 CPU。如果你在 `memcpy` 或 `Buffer.concat` 中看到高 CPU，你有不必要的复制。如果你在系统调用入口/退出（x86 上的 `__kernel_vsyscall`）中看到高 CPU，你是系统调用受限 - 批处理可能有所帮助。

在配置文件中查找 `sendfile()` 函数。如果它存在并且消耗了显著的 CPU（这实际上是内核时间，但归因于 sendfile），则零拷贝正在激活。如果它不存在而你期望它存在，则没有使用零拷贝。

使用 **Chrome DevTools 或 V8 的堆分析器** 进行 **内存分析** 可以显示分配模式。使用 `--inspect` 连接到 Node 并打开 Chrome DevTools。在处理流前后进行堆快照。如果你在堆中看到数百万个小的 Buffer 对象，则分配过多 - 池化可能有所帮助。你还可以使用 `node --heap-prof your-script.js` 生成堆分析文件，然后在 DevTools 中加载它进行分析。

“分配时间线”视图显示了随时间变化的分配率。尖峰分配模式表明突发分配 - 可能是在某些处理阶段。这有助于确定应用池化的位置。

使用 **--trace-gc** 进行 **GC 跟踪** 可以显示垃圾回收活动：

```js
node --trace-gc your-script.js
```

每个垃圾回收事件都会打印时间和堆大小。寻找频繁的次要垃圾回收（年轻代）——这是高分配率的标志。如果缓冲区池减少了次要垃圾回收频率，那么它就是有效的。寻找主要垃圾回收（老年代）——这些更昂贵。如果池化减少了主要垃圾回收暂停时间，那么这是一个巨大的胜利。

**使用`--trace-gc-verbose`提供详细的垃圾回收统计信息：**

```js
node --trace-gc --trace-gc-verbose your-script.js
```

这显示了垃圾回收时间，回收了多少内存，有多少内存幸存，以及提升率。高提升率（对象从年轻代移动到老年代）表明对象生命周期长——对于应该短暂或池化的缓冲区来说并不理想。

**事件循环延迟测量**告诉你 I/O 操作是否阻塞事件循环。使用`perf_hooks`：

```js
import { monitorEventLoopDelay } from 'perf_hooks';
 const h = monitorEventLoopDelay({ resolution: 20 });
h.enable();
 // Run your stream pipeline
 setInterval(() => {
 // Note: all values are in nanoseconds, divide by 1e6 for milliseconds
 console.log(`p50: ${(h.percentile(50) / 1e6).toFixed(2)}ms`);
 console.log(`p99: ${(h.percentile(99) / 1e6).toFixed(2)}ms`);
 console.log(`max: ${(h.max / 1e6).toFixed(2)}ms`);
}, 1000);
```

如果处理流时 p99 延迟激增，你的代码正在阻塞事件循环。这可能是因为同步缓冲区操作（在大型缓冲区上使用 Buffer.concat()）或大量数据块压倒处理循环。分解工作或使用更小的块。

**可以通过`NODE_DEBUG`环境变量启用 Node.js 内部模块调试：**

```js
NODE_DEBUG=fs,net,stream node your-script.js
```

这启用了指定内置模块（例如 fs、net、stream）的调试输出。它显示了内部操作，如文件打开、套接字连接和流状态更改。

注意：`DEBUG=*`环境变量是用于流行的 npm `debug`包，而不是 Node.js 内部。对于实际的系统调用级别可见性，请使用前面显示的 Linux 上的`strace`或 macOS 上的`dtruss`。

结合这些技术可以给你一个完整的图景。系统调用跟踪显示 I/O 行为，CPU 分析显示计算成本，内存分析显示分配影响，垃圾回收跟踪显示内存管理开销，事件循环监控显示响应性。共同来说，它们告诉你你的优化是否真正有效，以及下一步应该关注哪里。

## 当使用这些技术时

并非每个流都需要这些优化。以下是它们值得的时候：

**在以下情况下最小化缓冲区复制：**

+   通过流管道处理大文件

+   构建高吞吐量数据处理系统

+   提供静态内容（尽管在 Node.js 中内核级别的零拷贝不是自动的）

**在支持 reflinks（Btrfs、XFS、APFS）的文件系统上使用`fs.copyFile()`与`COPYFILE_FICLONE`进行真正的零拷贝文件复制**。

**在以下情况下不要过度优化复制：**

+   你需要转换数据（复制是不可避免的）

+   数据量小（优化开销超过收益）

+   你在缓冲区操作上不是 CPU 受限（先分析！）

**在以下情况下使用分散/收集 I/O（`_writev()`）：**

+   编写许多小块内容

+   系统调用开销高（分析确认）

+   你的目标支持向量写入

**在以下情况下跳过：**

+   块已经很大（批处理没有帮助）

+   你正在写入一个没有受益的流（内存缓冲区）

**在以下情况下使用缓冲区池：**

+   分配数百万个缓冲区（垃圾回收压力高）

+   缓冲区大小统一（易于池化）

+   你控制缓冲区生命周期（可以安全重用）

**在以下情况下跳过：**

+   缓冲区大小可变（难以高效地池化）

+   分配不是瓶颈（分析显示低垃圾回收影响）

**在以下情况下使用塞住/解除塞住：**

+   写入小块的突发

+   你控制着突发边界（开始和结束）

+   突发期间的延迟是可以接受的

**在以下情况下跳过：**

+   写入已经自然批处理

+   延迟很重要（塞住延迟会导致刷新）

先测量，后优化。这些技术有帮助，但会增加复杂性。只在它们能明显提高性能的地方应用，而不是默认地应用到每个地方。
