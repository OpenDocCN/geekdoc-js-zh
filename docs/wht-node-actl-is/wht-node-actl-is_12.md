# 可写流

> 原文：[`www.thenodebook.com/streams/writable-streams`](https://www.thenodebook.com/streams/writable-streams)

你已经看到了可读流是如何工作的。你理解了它们如何维护内部缓冲区，如何在模式之间转换，以及如何将数据传递给消费者。现在我们需要转换视角，看看流方程的另一边：**一旦数据被生成后，数据去哪里了？**

这是**可写流**的领域。如果可读流是关于从源获取数据，那么可写流则是关于将数据*输入*到目的地。磁盘上的文件、网络套接字、HTTP 响应、压缩算法、数据库连接——任何你按块发送数据的地方，你都在使用某种形式的可写流。

向可写流写入并不简单。不仅仅是“用你的数据调用`write()`然后继续”。API 中内置了一个关键的反馈机制，如果你忽略它，最终你会看到你的进程消耗的内存比你预期的要多。这个反馈机制是**背压**，理解它是处理数据流的 Node.js 生产代码所必需的。

我们将从底层开始检查可写流。首先，我们将查看`Writable`类本身——它接受哪些选项，它发出哪些事件，以及`write()`方法的返回值实际上意味着什么。然后我们将深入探讨**背压**，了解它的存在原因，内部缓冲如何创建它，以及忽略它会发生什么。之后，我们将实现自定义的可写流，以便你确切了解数据流入目的地时发生了什么。最后，我们将查看在实际应用程序中写入可写流的正确模式。

## 可写流类

当你创建或接收一个可写流时，你正在处理一个扩展了`EventEmitter`的对象，就像可读流一样。现在这应该感觉熟悉了。Node.js 中的流通过事件进行通信，因为异步 I/O 操作不会立即返回值——它们通过事件来表示完成或失败。

可写流在概念上的任务是直接的：通过其`write()`方法接受数据块，并将这些块发送到某个底层目的地。目的地可以是任何东西。操作系统管理的文件描述符。TCP 套接字。内存中的数组。可写流并不关心。它提供接口和缓冲逻辑。底层目的地被抽象成一个内部方法`_write()`，子类实现它。

配置选项控制了在负载下可写流的行为。

`highWaterMark`选项与可读流的工作方式类似，但其含义略有不同。对于可写流，`highWaterMark`表示在开始信号回压之前，流将内部缓冲的最大字节数（或在`objectMode`中为对象数量）。默认值为`16384`字节，与可读流使用的相同的 16KB 默认值。

当你在可写流上调用`write()`时，这些数据并不一定会立即写入底层目标。相反，它们被添加到内部缓冲区。如果目标快速（例如写入到`/dev/null`或具有大量带宽的套接字），缓冲区将保持大部分为空，写入操作会很快完成。但如果目标较慢（例如在重负载 I/O 期间写入机械硬盘，或在拥塞的网络中发送数据），缓冲区开始填满。

当缓冲数据达到或超过`highWaterMark`时，`write()`方法返回`false`。这是**回压信号**。流表示“我正在缓冲太多数据。你需要减慢速度或停止写入，直到我告诉你我准备好了。”如果应用程序忽略这个信号并继续调用`write()`，内部缓冲区会继续增长，消耗越来越多的内存，直到进程耗尽。

可写流配置看起来像这样：

```js
import { Writable } from "stream";
 const writable = new Writable({
 highWaterMark: 8192, // 8KB buffer threshold
});
```

这创建了一个可写流，当其内部缓冲区达到 8KB 时将发出回压信号。请注意，当发出回压信号时，流不会停止接受写入——它只是返回`false`以指示你应该暂停。

`objectMode`选项，就像在可读流中一样，将流从处理字节转换为处理任意 JavaScript 对象。在`objectMode`中，`highWaterMark`表示缓冲的对象数量，而不是字节数。在`objectMode`中的默认值是 16 个对象。

```js
const objectWritable = new Writable({
 objectMode: true,
 highWaterMark: 50, // buffer up to 50 objects
});
```

当你构建数据处理管道时，其中每个块代表一个逻辑单元——一个数据库行、一个解析的日志条目、一个 JSON 文档——而不是字节块时，这很有用。

`decodeStrings`选项控制传递给`write()`方法的字符串是否在传递给`_write()`方法之前转换为`Buffer`对象。默认情况下，这是`true`。如果你将其设置为`false`，字符串将原样传递。大多数情况下你不需要触碰这个设置，但如果你正在实现一个想要将字符串与缓冲区区别对待的特定可写流，这很重要。

```js
const stringWritable = new Writable({
 decodeStrings: false, // keep strings as strings
});
```

此外，还有一个`defaultEncoding`选项，用于指定在字符串转换为缓冲区时使用的编码（如果`decodeStrings`为`true`）。默认值是`'utf8'`，这对于文本数据几乎总是你想要的。

最后，还有一个`emitClose`选项，用于控制流在销毁时是否发出`close`事件。默认值是`true`。除非你有特定的理由要抑制`close`事件，否则你应该保持这个设置不变。

要有效地使用可写流，您需要了解它们发出的事件以及它们所表示的含义。

## 可写流上的事件

可写流发出几个事件，以表示状态变化。每个事件都在流的特定生命周期点触发。

管理背压的最关键事件是**`drain`**。当可写流的内部缓冲区已满（意味着`write()`返回`false`）并且现在已排空至`highWaterMark`阈值以下时，此事件会被触发。`drain`事件是您继续写入的信号。

预期的使用模式：

```js
function writeData(writable, data) {
 if (!writable.write(data)) {
 writable.once("drain", () => {
 // Buffer drained, safe to write more
 continueWriting();
 });
 }
}
```

`write()`调用返回`false`，这意味着缓冲区已满。因此，我们附加一个一次性`drain`事件监听器并暂停我们的写入逻辑。当`drain`触发时，缓冲区又有空间了，我们可以继续。

如果您之前从未使用过流，或者没有阅读我之前关于流的章节，这可能会感觉有些奇怪。**阻塞会违背 Node.js 异步 I/O 模型的整体目的**。如果`write()`被阻塞，您的整个事件循环将冻结，等待写入完成。通过使用基于事件的信号，事件循环保持空闲，以便在流内部缓冲区排空时处理其他工作。

当您在流上调用`end()`并且所有缓冲数据都已成功写入底层目的地时，会触发**`finish`**事件。这是流完成其工作的信号。请注意，`finish`在`close`之前触发。流已完成写入，但它可能尚未关闭其底层资源。

```js
writable.on("finish", () => {
 console.log("All data written");
});
 writable.write("some data");
writable.end(); // signals end of writes
```

`end()`方法告诉流“我不会再写入任何数据。”在您调用`end()`之后，调用`write()`将抛出错误。流处理任何剩余的缓冲数据，当所有数据都已写入时，它发出`finish`。

**`close`**事件在流及其底层资源被关闭时触发。这发生在`finish`之后。并非所有流都会发出`close`，它们是否这样做取决于底层资源的实现。对于文件流，`close`在文件描述符关闭时触发。对于套接字流，`close`在套接字关闭时触发。

```js
writable.on("close", () => {
 console.log("Stream closed");
});
```

**`error`**事件在写入过程中出现问题时触发。可能是文件系统已满。可能是网络连接断开。可能是底层目的地由于某些内部原因抛出了错误。当发生错误时，流会发出带有错误对象的`error`。就像可读流一样，如果您没有附加错误处理程序，错误将被抛出，可能会使您的进程崩溃。

```js
writable.on("error", (err) => {
 console.error("Write error:", err);
});
```

当使用`pipe()`方法将可读流管道连接到该可写流时，会触发**`pipe`**事件。事件将源可读流作为参数传递。这主要用于日志记录或调试——了解哪些可读流当前已管道连接到该可写流。

```js
writable.on("pipe", (src) => {
 console.log("Something is piping into me");
});
```

类似地，当从该可写流中取消管道连接时，会触发**`unpipe`**事件。

这些事件构成了与可写流交互的 API 界面。但为了真正理解如何正确使用它们，我们需要深入了解控制流系统中流量控制的核心理念：**背压**。

## 理解背压

背压是那些听起来很抽象的概念之一，直到你看到没有它会发生什么。考虑一个具体的场景。

你正在编写一个程序，该程序读取一个大型文件并将其写入另一个位置。**简单的方法**看起来像这样：

```js
import { createReadStream, createWriteStream } from "fs";
 const readable = createReadStream("input.dat");
const writable = createWriteStream("output.dat");
 readable.on("data", (chunk) => {
 writable.write(chunk);
});
```

这段代码从`input.dat`读取数据块并将其写入`output.dat`。简单，对吧？但有一个**隐藏的问题**。如果可读流产生的数据比可写流消耗得快怎么办？

文件系统可以从源磁盘以 100MB/sec 的速度读取，但它只能以 50MB/sec 的速度将数据写入目标磁盘。可读流以 100MB/sec 的速度产生数据块，而你为每个数据块调用`write()`。可写流只能处理 50MB/sec，所以其他 50MB/sec 的数据**积累在其内部缓冲区中**。一秒后，缓冲区中有 50MB。两秒后，100MB。十秒后，500MB。缓冲区会不断增长，直到你的进程耗尽内存并崩溃。

这正是背压解决的问题。可写流通过从`write()`返回`false`向生产者发出“慢点，我跟不上”的信号。然后，生产者应该暂停，直到可写流发出`drain`事件，表示它已经赶上并准备好接收更多数据。

正确的版本：

```js
readable.on("data", (chunk) => {
 const canContinue = writable.write(chunk);
 if (!canContinue) {
 readable.pause();
 writable.once("drain", () => {
 readable.resume();
 });
 }
});
```

当`write()`返回`false`时，我们暂停可读流。它停止发出`data`事件。可写流内部缓冲区在将数据写入底层目标时排空。当缓冲区低于`highWaterMark`时，`drain`事件触发，我们恢复可读流。**数据流由消费者的容量调节，而不是生产者的速度。**

这种模式如此常见，以至于 Node.js 提供了一个内置的`pipe()`方法来自动处理这种精确的流控制。我们将在后面的子章节中深入探讨`pipe()`，但这表明背压管理是流模型的一个基本部分，重要到足以需要专门的辅助方法。

当你调用`write()`时，可写流内部实际上发生了什么？

当你调用`writable.write(chunk)`时，流会做几件事情。首先，它会检查是否已经在向底层目标写入数据。如果是，则将数据块添加到内部缓冲区。如果没有正在写入，则将数据块立即传递给`_write()`方法，该方法处理实际的 I/O 操作。

内部缓冲区是一个**写入请求的链表**。每个写入请求包含要写入的数据块、编码（如果是字符串）以及写入完成后要调用的回调函数。随着你反复调用`write()`，更多的写入请求被添加到这个列表中。

在将块添加到缓冲区（或传递给`_write()`）之后，流计算当前缓冲区大小。对于字节流，这是所有缓冲块长度的总和。对于`objectMode`流，这是缓冲块的数量。如果这个总数达到或超过`highWaterMark`，`write()`返回`false`。否则，它返回`true`。

简化的心智模型：

```js
class SimplifiedWritable {
 constructor(options) {
 this.highWaterMark = options.highWaterMark || 16384;
 this.buffer = [];
 this.bufferSize = 0;
 this.writing = false;
 }
 write(chunk) {
 this.buffer.push(chunk);
 this.bufferSize += chunk.length;
 if (!this.writing) {
 this._processBuffer();
 }
 return this.bufferSize < this.highWaterMark;
 }
 _processBuffer() {
 // writes chunks to destination
 }
}
```

这个简化的模型捕捉了核心思想。`write()`方法向缓冲区添加内容，并根据缓冲区是否低于阈值返回布尔值。

是什么决定了缓冲区排空的速率？这完全取决于`_write()`方法的实现和底层目标性能。如果你正在向快速的 SSD 写入，缓冲区会快速排空。如果你正在向慢速的网络连接写入，缓冲区会缓慢排空。可写流本身并不控制排空速率 - 它只测量它，并在缓冲区太满时发出信号。

这就是为什么`highWaterMark`是一个**调整参数**。如果你设置得太低，你将非常频繁地收到背压信号，即使目标可以处理更多数据。这可能会降低吞吐量，因为你一直在暂停和恢复。如果你设置得太高，你将在内存中缓冲大量数据，如果你有额外的内存，这可能没问题，但如果你在处理多个流的同时或在内存受限的环境中运行，可能会导致问题。

**默认的 16KB**对于大多数用例来说是一个合理的折中方案。它足够大，以至于你不会在典型的写入操作中不断遇到背压，但足够小，以至于如果目标较慢，你不会缓冲大量数据。

当你忽略背压时会发生什么？假设你向一个正在向慢速目标写入的 Writable 流写入了一百万个 1KB 的数据块。你调用`write()`一百万次而不检查返回值。每次调用都会向内部缓冲区添加 1KB。缓冲区增长到 1GB。现在你的进程在内存中就有**1GB 的数据**仅为此流的缓冲区等待写入。

如果你同时处理多个文件，或者处理多个并发 HTTP 请求，这种内存使用会成倍增加。你可能在所有流中都有 10GB 或更多的缓冲写入数据。在某个时刻，操作系统的内存分配器可能跟不上了，你的进程会因**内存不足错误**而崩溃。

这在生产系统中很常见，当开发者不尊重背压时就会发生。我在 Node.js 应用程序中调试过内存问题，其根本原因正是这一点：从快速源向慢速目标流式传输数据，而没有处理`write()`的返回值。

修复方法简单但需要自律。**检查返回值。暂停生产者。等待`drain`。恢复。** 这是一种一旦内化就会变得自然而然的模式。

## 可写流中的内部缓冲

内部缓冲的结构和管理决定了流如何处理内存使用和性能。这有助于你在流代码中推理内存使用和性能。

可写流维护一个`_writableState`对象，用于跟踪其内部状态。这个对象是**私有的**（由前面的下划线表示），但了解其中内容是有用的，因为它会影响流的行为。

一个关键属性是`bufferedRequestCount`，正如其名：当前缓冲的写请求的数量。每次调用`write()`时，如果流已经在写入，则新的块会变成一个缓冲请求，并且`bufferedRequestCount`增加。随着块被写入目的地，`bufferedRequestCount`减少。

另一个属性是`length`，它跟踪缓冲数据的总大小。对于字节流，这是所有缓冲块长度的总和。对于`objectMode`流，这是缓冲对象的计数。这是与`highWaterMark`比较的值，以确定`write()`是否应该返回`false`。

此外，还有一个`writing`标志，表示流是否正在执行写入操作。当调用`_write()`时，`writing`被设置为`true`。当传递给`_write()`的回调被调用（表示写入完成）时，`writing`被设置为`false`。在`writing`为`true`时，新的块会被缓冲而不是直接传递给`_write()`。

在较老的 Node.js 版本中，缓冲本身是一个写请求对象的**链表**。在新版本中，它实现为一个更有效的数据结构，但从概念上讲，它仍然是一个**队列：先进先出**。最老的缓冲块首先被写入，然后是下一个最老的，依此类推。

当调用`write()`时，流会检查`writing`标志。如果它是`false`，则将块立即传递给`_write()`并附带一个回调。当写入操作完成（或出错）时，`_write()`实现会调用回调。当回调运行时，流会检查是否有更多缓冲块。如果有，它会从缓冲区中拉取下一个块并再次调用`_write()`。这个过程会一直持续到缓冲区为空。

如果在这个过程中缓冲区大小降至`highWaterMark`以下，并且缓冲区大小之前在或高于`highWaterMark`，则流会发出`drain`事件。这是**背压已缓解**的信号。

一个更详细的心智模型：

```js
class DetailedWritable {
 constructor(options) {
 this.highWaterMark = options.highWaterMark || 16384;
 this.buffer = [];
 this.length = 0;
 this.writing = false;
 this.needDrain = false;
 }
 write(chunk) {
 this.buffer.push(chunk);
 this.length += chunk.length;
 const ret = this.length < this.highWaterMark;
 if (!ret) {
 this.needDrain = true;
 }
 if (!this.writing) {
 this._doWrite();
 }
 return ret;
 }
 _doWrite() {
 if (this.buffer.length === 0) {
 if (this.needDrain) {
 this.needDrain = false;
 this.emit("drain");
 }
 return;
 }
 const chunk = this.buffer.shift();
 this.length -= chunk.length;
 this.writing = true;
 this._write(chunk, (err) => {
 this.writing = false;
 if (err) {
 this.emit("error", err);
 } else {
 this._doWrite();
 }
 });
 }
}
```

这仍然是一个简化的版本，但它展示了核心流程。`write()`方法将块添加到缓冲区，如果缓冲区超过`highWaterMark`，则返回`false`。`_doWrite()`方法一次处理一个缓冲块，调用`_write()`并在回调执行后再处理下一个块。当缓冲区为空且`needDrain`为`true`时，会触发`drain`事件。

缓冲区是 **生产者**（调用 `write()` 的代码）和底层目的地（`_write()` 中的代码）之间的 **队列**。生产者可以尽可能快地向队列中添加内容，但如果目的地较慢，队列就会增长。反压机制（`write()` 返回 `false`，`drain` 事件）是反馈循环，告诉生产者在队列变得过大时减速。

流不会 **强制** 反压 - 它只是 **信号**。即使 `write()` 返回 `false`，你仍然继续调用 `write()`，流会继续缓冲。它不会抛出错误。它不会丢弃数据。它只是会继续增长缓冲区，直到内存耗尽。这个设计选择是有意为之的 - 它给应用程序提供了灵活性，以决定如何处理反压。一些应用程序可能具有硬实时约束，并选择丢弃数据而不是暂停。其他应用程序可能会选择更积极地缓冲，因为它们有额外的内存。但 **默认的正确行为是在 `write()` 返回 `false` 时暂停**。

缓冲区使用的内存是除了块本身使用的内存之外的。每个写入请求都是一个对象，它包含块、编码、回调和其他元数据。对于大量的小写入，这些对象的开销可能是相当大的。这就是为什么将小写入批处理到更大的块中可以提高性能的原因之一 - 更少的写入请求意味着更少的对象开销。

`cork()` 和 `uncork()` 方法是专门设计来优化在执行许多小写入时的缓冲。

当你调用 `writable.cork()` 时，流进入 **corked 状态**。在这个状态下，所有写入都会被缓冲，并且根本不会调用 `_write()`。这个想法是，你即将进行一系列小写入，你希望将它们全部缓冲起来，然后一次性写入。

在完成所有写入后，你调用 `writable.uncork()`。这将刷新缓冲的写入。如果流的 `_writev()` 方法已实现（允许一次性写入多个块），`uncork()` 将使用所有缓冲的块调用 `_writev()`。否则，它将为每个块重复调用 `_write()`。

这里有一个例子：

```js
writable.cork();
writable.write("line 1\n");
writable.write("line 2\n");
writable.write("line 3\n");
writable.uncork();
```

没有使用 `cork()`，每次 `write()` 调用可能会立即调用 `_write()`，导致三个独立的 I/O 操作。使用 `cork()` 后，所有三个写入都会被缓冲，当调用 `uncork()` 时，它们会一起写入（如果实现了 `_writev()`），或者在没有停顿的情况下顺序写入。

当你进行许多小写入时，这可以显著提高性能，因为它减少了系统调用的次数。在三次单独的 `write()` 系统调用中写入三个 8 字节块比在一次调用中写入一个 24 字节块要慢。

然而，`cork()`和`uncork()`在大多数应用程序代码中并不需要考虑。它们在实现生成许多小数据块的库或协议处理程序时最有用。对于典型应用程序代码，直接调用`write()`就足够了。

重要提示：`cork()`可以被多次调用，并且每次调用都会增加一个内部计数器。您需要以相同次数调用`uncork()`来实际刷新缓冲区。这允许在复杂的代码路径中进行**嵌套塞住**。

## 实现自定义可写流

理解如何使用可写流是一回事。理解它们内部的工作原理则是另一回事。**最有效的方法是实现自己的可写流**。

模式很简单。您扩展`Writable`类并实现`_write()`方法。此方法接收**三个参数**：要写入的数据块、编码（如果它是字符串）以及写入完成后要调用的回调。

下面是最简单的自定义可写流：

```js
import { Writable } from "stream";
 class NullWritable extends Writable {
 _write(chunk, encoding, callback) {
 callback(); // write completes immediately
 }
}
```

这实际上等同于`/dev/null`。它接受数据但不做任何处理。`_write()`方法只是立即调用回调，表示写入成功。

一个写入数组的可写流：

```js
class ArrayWritable extends Writable {
 constructor(options) {
 super(options);
 this.data = [];
 }
 _write(chunk, encoding, callback) {
 this.data.push(chunk);
 callback();
 }
}
```

现在，数据块累积在`data`数组中。每次写入只是将数据块推入并调用回调。

但如果写入操作是**异步的**呢？如果我们正在向数据库写入，或者通过网络发送数据呢？这就是回调变得重要的地方。**回调必须在异步操作完成后被调用**。

模拟的异步写入：

```js
class AsyncWritable extends Writable {
 _write(chunk, encoding, callback) {
 setTimeout(() => {
 console.log("Wrote:", chunk.toString());
 callback();
 }, 100);
 }
}
```

`setTimeout`模拟了一个耗时 100 毫秒的异步 I/O 操作。回调在`setTimeout`回调内部被调用，表示写入已完成。在`callback()`被调用之前，流不会处理下一个缓冲的数据块。**这就是流如何调整自己的速度以匹配目标速度的方式**。

如果在写入过程中发生错误，您将错误传递给回调：

```js
class ErrorWritable extends Writable {
 _write(chunk, encoding, callback) {
 if (chunk.toString().includes("bad")) {
 callback(new Error("Invalid data"));
 } else {
 callback();
 }
 }
}
```

当您将错误传递给回调时，流会发出一个`error`事件。任何缓冲的写入都会被丢弃，流进入**错误状态**。

`_writev()`方法是一个可选的优化，您可以使用它更有效地处理批量写入。如果实现了`_writev()`，并且有多个数据块被缓冲（或流被塞住），流将使用所有缓冲数据块的数组调用`_writev()`，而不是反复调用`_write()`。

`_writev()`的签名略有不同：

```js
class BatchWritable extends Writable {
 _writev(chunks, callback) {
 const allData = Buffer.concat(
 chunks.map((c) => c.chunk)
 );
 console.log("Batch write:", allData.length, "bytes");
 callback();
 }
}
```

`chunks`参数是一个对象数组，每个对象都有一个`chunk`属性（数据）和一个`encoding`属性。您可以一次性处理它们并完成时调用回调。

实现 `_writev()` 是可选的。如果你没有实现它，流将为每个缓冲块调用一次 `_write()`。但是，如果你的底层目标有一个批处理写入的 API（例如，具有多行的 SQL `INSERT` 或支持捆绑多个消息的网络协议），实现 `_writev()` 可以**显著提高性能**。

`_final()` 钩子在流结束时被调用（在调用 `end()` 之后并且所有缓冲写入都已处理），但在发出 `finish` 事件之前。它对于清理或最终操作很有用，例如关闭文件描述符或刷新缓冲区。

```js
class CleanupWritable extends Writable {
 _write(chunk, encoding, callback) {
 // write data
 callback();
 }
 _final(callback) {
 console.log("Finalizing...");
 // perform cleanup
 callback();
 }
}
```

`_final()` 回调必须在最终化完成后被调用，以表示最终化完成。在它被调用后，流发出 `finish` 事件。

一个更现实的自定义可写流，它以自定义格式写入日志文件：

```js
import { Writable } from "stream";
import { open, write as fsWrite } from "fs";
 class LogWritable extends Writable {
 constructor(filename, options) {
 super(options);
 this.filename = filename;
 this.fd = null;
 this._open();
 }
 _open() {
 open(this.filename, "a", (err, fd) => {
 if (err) {
 this.destroy(err);
 } else {
 this.fd = fd;
 this.emit("open", fd);
 }
 });
 }
 _write(chunk, encoding, callback) {
 if (!this.fd) {
 this.once("open", () => {
 this._write(chunk, encoding, callback);
 });
 return;
 }
 const line = `[${new Date().toISOString()}] ${chunk}\n`;
 fsWrite(this.fd, line, callback);
 }
 _final(callback) {
 if (this.fd) {
 require("fs").close(this.fd, callback);
 } else {
 callback();
 }
 }
}
```

此流在构造函数中打开一个用于追加的文件。当调用 `_write()` 时，它使用时间戳格式化块，并使用低级 `fs.write()` 函数将块写入文件。在 `_final()` 中，它关闭文件描述符。

注意 `_write()` 如何处理文件尚未打开的情况，通过等待 `open` 事件。这是当底层资源初始化是异步时的**常见模式**。

实现这样的自定义可写流让你**完全控制**数据去向和数据处理方式。你可以将数据写入数据库、外部 API、压缩流，或任何其他地方。可写接口足够灵活，可以适应任何目标。

## 正确写入可写流

在理解了可写流内部工作原理的基础上，以下是应用代码中正确写入它们的实用模式。

**最重要的规则**：始终检查 `write()` 的返回值。如果它返回 `false`，暂停你的数据源，并在继续之前等待 `drain`。

在从流中读取并写入另一个流的上下文中的模式：

```js
import { createReadStream, createWriteStream } from "fs";
 const reader = createReadStream("input.txt");
const writer = createWriteStream("output.txt");
 reader.on("data", (chunk) => {
 const ok = writer.write(chunk);
 if (!ok) {
 reader.pause();
 }
});
 writer.on("drain", () => {
 reader.resume();
});
 reader.on("end", () => {
 writer.end();
});
```

我们调用 `write()` 并捕获返回值。如果它是 `false`，我们暂停读取器。当写入器的 `drain` 事件触发时，我们恢复读取器。这确保了写入器的缓冲区永远不会无限制地增长。

当你完成写入后，你必须调用 `end()` 来表示不再写入更多数据。你可以选择性地将一个最终块传递给 `end()`：

```js
writer.end("final chunk");
```

这与以下操作等价：

```js
writer.write("final chunk");
writer.end();
```

在调用 `end()` 之后，流会处理任何剩余的缓冲写入，如果实现了 `_final()`，则调用 `_final()`，然后发出 `finish` 事件。在那个时刻，调用 `write()` 将会抛出错误。

错误是特定的**写入后结束错误**，其外观如下：

```js
writer.end();
writer.write("more data"); // throws ERR_STREAM_WRITE_AFTER_END
```

当你有异步代码写入流，并且代码的另一部分在异步写入完成之前调用 `end()` 时，这是一个**常见错误**。你需要确保在调用 `end()` 之前所有写入都已完成。

在应用程序代码中，`cork()` 和 `uncork()` 有特定的用途。如果你正在快速连续进行许多小写入，你可以使用 `cork()` 来缓冲它们，并使用 `uncork()` 来刷新：

```js
writer.cork();
for (let i = 0; i < 1000; i++) {
 writer.write(`line ${i}\n`);
}
writer.uncork();
```

但在实践中，你很少需要手动这样做。如果你使用 `pipe()` 或 `pipeline()`，背压会自动处理。如果你直接写入流，流中的缓冲已经提供了一些批处理。**`Cork` 主要用于生成结构化输出的库代码**，如 HTTP/2 帧编码器或数据库协议处理程序。

一个不可避免需要手动处理背压的场景是当你从**非流式源**生成数据时。例如，遍历一个数组并将每个元素写入：

```js
async function writeArray(writable, array) {
 for (const item of array) {
 const ok = writable.write(item);
 if (!ok) {
 await new Promise((resolve) => {
 writable.once("drain", resolve);
 });
 }
 }
 writable.end();
}
```

此函数将每个数组元素写入流。如果 `write()` 返回 `false`，它将在继续之前等待 `drain` 事件。这确保了即使数组很大且流很慢，流缓冲区也不会溢出。

另一种模式是使用具有异步迭代的可写流。Node.js 提供了一个 `stream.Writable.toWeb()` 方法，它创建一个 `WritableStream`（来自 WHATWG Streams 标准），这可以与异步迭代一起使用。但这是一个更高级的话题，我们将在现代网络 API 的上下文中进行讨论。

可写流通过 `write()` 返回值和 `drain` 事件内置了流控制。**尊重这种流控制是强制性的**。这是在轻负载下工作但重负载时崩溃的代码与在生产环境中可靠工作的代码之间的区别。

一个将所有内容结合起来的完整示例：我们将编写一个程序，读取一个大型 CSV 文件，解析每一行，转换数据，并将其写入数据库。我们将使用可写流并正确处理背压。

```js
import { createReadStream } from "fs";
import { Writable } from "stream";
import { pipeline } from "stream/promises";
 class DatabaseWriter extends Writable {
 constructor(db) {
 super({ objectMode: true });
 this.db = db;
 }
 async _write(row, encoding, callback) {
 try {
 await this.db.insert(row);
 callback();
 } catch (err) {
 callback(err);
 }
 }
}
 async function importCSV(filename, db) {
 const reader = createReadStream(filename);
 const parser = parseCSV(); // hypothetical CSV parser
 const writer = new DatabaseWriter(db);
 await pipeline(reader, parser, writer);
 console.log("Import complete");
}
```

此示例使用 `pipeline()`，它自动处理背压。`DatabaseWriter` 是一个自定义的可写流，它将每一行写入数据库。`_write()` 方法是异步的，这是允许的 - 你可以使用异步函数或从 `_write()` 返回承诺，Node.js 将等待它们解决后再处理下一个块。

注意，我们并没有手动检查 `write()` 的返回值或监听 `drain` 事件。这是因为 `pipeline()` 会为我们处理这些。这是**大多数流代码的推荐模式**：使用 `pipeline()` 或 `pipe()`，让 Node.js 处理背压，并专注于转换逻辑。

但当你不能使用 `pipeline()` - 当你处理多个来源或目的地，或者当你与非流式 API 集成时 - 你需要手动处理背压。当你这样做时，模式总是相同的：**检查 `write()`，当它返回 `false` 时暂停，当 `drain` 触发时恢复**。

## 缓冲区溢出的机制

通过追踪导致内存耗尽的确切事件序列，有助于你理解为什么存在背压预防机制。

考虑一个场景，你从快速源读取并写入慢速目标。你正在将 1GB 的文件从固态硬盘复制到网络共享，通过一个拥挤的链路。固态硬盘可以以 500 MB/sec 的速度提供数据。网络只能以 10 MB/sec 的速度发送。这是一个**50 倍的速度差异**。

你的代码看起来像这样：

```js
readable.on("data", (chunk) => {
 writable.write(chunk); // ignoring return value
});
```

可读流开始以固态硬盘能提供的速度传输 64KB 的块。每**128 微秒**，一个新的块到达（500 MB/sec 的 64KB）。每个块都传递给`write()`。可写流尝试通过网络发送每个块，但网络每**6.4 毫秒**只能处理大约一个 64KB 的块（10 MB/sec 的 64KB）。

在最初的 6.4 毫秒内，可读流传输了 50 个块。可写流发送了 1 个块。其他 49 个块被缓冲。这是 3.1MB 的缓冲数据。

在 64 毫秒后，可读流已传输了 500 个块。可写流已发送了 10 个块。还有 490 个块被缓冲。这是 30.6MB。

一秒后，缓冲区有 490MB。两秒后，980MB。在某个时刻，进程耗尽堆空间并崩溃。

这不是逐渐减速。**这个过程运行良好，直到突然崩溃**。事件循环一直响应，直到分配器无法为下一个缓冲区分配内存，V8 抛出内存不足异常。

相比之下，**尊重背压**的代码：

```js
readable.on("data", (chunk) => {
 const ok = writable.write(chunk);
 if (!ok) {
 readable.pause();
 }
});
 writable.on("drain", () => {
 readable.resume();
});
```

当缓冲区达到`highWaterMark`（默认为 16KB）时，`write()`返回`false`。可读流被暂停。它停止传输块。可写流继续通过网络发送缓冲的块。当缓冲区低于`highWaterMark`时，`drain`事件触发，可读流恢复。

缓冲区大小在 0 和`highWaterMark`之间波动。它永远不会无限制增长。内存使用量由`highWaterMark`加上可读流单个块的大小限制。对于默认设置，这是**32KB 的总缓冲空间**，无论文件有多大或目标有多慢。

这就是**背压的力量**。它将生产者的速度与消费者的速度解耦，同时保持内存使用的界限。

但这里有一个值得探讨的微妙之处。`highWaterMark`**不是一个硬限制**。缓冲区可以超过`highWaterMark`。`highWaterMark`控制的是`write()`何时返回`false`。如果你用一个 10MB 的块调用`write()`，并且缓冲区为空，该块将被缓冲，缓冲区大小现在是 10MB，远远超过了 16KB 的`highWaterMark`。但`write()`在这个调用上会返回`false`，表示背压。

这意味着实际的峰值内存使用量是 **`highWaterMark` 加上最大单个块的大小**。如果你以 64KB 的块进行流式传输，并且 `highWaterMark` 为 16KB，则峰值缓冲区约为 80KB。如果你以 1MB 的块进行流式传输，则峰值缓冲区约为 1MB。这就是为什么**选择合适的块大小很重要**，尤其是在内存受限的环境中。

有另一种场景，忽略背压会导致问题：**从多个生产者向单个可写流写入**。假设你有 10 个并发操作都在向同一个日志文件流写入。如果它们都不尊重背压，并且每个都以尽可能快的速度产生数据，缓冲区将增长以容纳所有 10 个流的输出。一个流的 16KB 缓冲区可能变成 160KB 或更多。在繁忙的服务器上，如果有数百或数千个并发操作，这可能会导致内存泄漏。

解决方案是相同的：尊重背压。每个生产者都会检查 `write()` 的返回值，并在返回 `false` 时暂停。可写流会发出一次 `drain` 事件，然后所有暂停的生产者都会恢复。缓冲区保持有界。

## 可写流中的错误处理

我们已经讨论了“快乐路径”——数据流动，尊重背压，流干净结束。但当事情出错时会发生什么呢？

可写流中的错误可能在多个点发生。底层目的地可能失败——磁盘可能已满，网络可能断开，可能发生权限错误。数据本身可能对目的地无效。当尝试操作时，流可能处于无效状态。

当在 `_write()`、`_writev()` 或 `_final()` 中发生错误时，错误会被传递给回调。流通过发出一个 `error` 事件来处理这个问题。在发出 `error` 事件后，流进入一个**错误状态**。任何缓冲的写入都会被丢弃，并且进一步的写入将抛出错误。

这看起来像：

```js
class FailingWritable extends Writable {
 _write(chunk, encoding, callback) {
 callback(new Error("Write failed"));
 }
}
 const writable = new FailingWritable();
 writable.on("error", (err) => {
 console.error("Stream error:", err.message);
});
 writable.write("test"); // triggers error event
writable.write("more"); // throws ERR_STREAM_DESTROYED
```

第一个 `write()` 触发 `error` 事件。之后，流被销毁，后续的写入将抛出异常。

如果你没有附加 `error` 事件监听器，错误将被抛出，可能会使你的进程崩溃。**你必须始终将错误处理程序附加到流上**，尤其是在生产代码中。未处理的流错误是 Node.js 进程意外崩溃的最常见原因之一。

在流管道中优雅地处理错误的模式，我们将在管道子章节中深入探讨。但就目前而言，要明白**错误处理是必不可少的**。你创建或接收的每个可写流都必须有一个错误处理程序。

另一个错误场景是在调用 `end()` 之后调用 `write()`。这是一个**编程错误**，而不是运行时错误。它表明你的代码逻辑中存在一个错误。当你调用 `end()` 时，你是在告诉流“没有更多的写入。”如果你随后调用 `write()`，流将抛出 `ERR_STREAM_WRITE_AFTER_END` 错误。

这通常发生在异步代码中，其中多个代码路径正在写入同一个流，其中一个路径调用 `end()`，而另一个路径仍有写入待处理。修复方法是协调你的代码，确保只有在所有写入都完成后才调用 `end()`。这可能涉及使用 `Promise.all()` 等待所有异步写入，或者使用计数器来跟踪待处理的写入。

问题的例子：

```js
async function buggyWrite(writable) {
 setTimeout(() => {
 writable.write("async write");
 }, 100);
 writable.end(); // called before async write
}
```

以及修复方法：

```js
async function correctWrite(writable) {
 await new Promise((resolve) => {
 setTimeout(() => {
 writable.write("async write", resolve);
 }, 100);
 });
 writable.end();
}
```

现在只有在异步写入完成后才会调用 `end()`。

此外，还有 `destroy()` 方法，它强制关闭流并可选择发出错误。调用 `writable.destroy(err)` 立即将流置于已销毁状态，丢弃缓冲写入，并发出 `error`（如果提供了 `err`）然后是 `close`。

```js
writable.destroy(new Error("Aborted"));
```

当你需要取消正在进行的流操作时，这很有用，比如当用户取消文件上传时。`destroy()` 不等待缓冲写入完成——它是一个 **立即的、强制性的关闭**。

`destroyed` 属性告诉你流是否已被销毁：

```js
if (!writable.destroyed) {
 writable.write("data");
}
```

使用这个检查可以防止尝试写入已被销毁的流的代码中的错误。

## 属性和内省

可写流公开了几个属性，让你可以内省其当前状态。这些对于调试、监控以及关于流控制的运行时决策很有用。

`writableLength` 属性告诉你当前有多少字节（或在 `objectMode` 中是对象）被缓冲：

```js
console.log(writable.writableLength);
```

如果这个值接近 `writableHighWaterMark`，你知道即将发出背压信号。你可能使用这个来实施软速率限制或向用户提供上传进度的反馈。

`writableHighWaterMark` 属性公开了 `highWaterMark` 值：

```js
console.log(writable.writableHighWaterMark);
```

这是确定 `write()` 返回 `false` 的阈值。它在流构建期间设置，通常不会改变，但你可以读取它来了解流的缓冲行为。

`writable` 属性是一个布尔值，指示是否安全调用 `write()`：

```js
if (writable.writable) {
 writable.write("data");
}
```

如果流已被销毁或结束，这是 `false`。在尝试写入之前这是一个快速检查。

`writableEnded` 属性告诉你是否已调用 `end()`：

```js
console.log(writable.writableEnded);
```

即使 `finish` 事件尚未触发，在调用 `end()` 之后这也是 `true` 的。这表明不再接受更多的写入。

`writableFinished` 属性告诉你是否已发出 `finish` 事件：

```js
console.log(writable.writableFinished);
```

在所有写入都已处理并且 `finish` 事件已触发后，这是 `true` 的。这表明流已经完成了其工作。

`writableCorked` 属性告诉你 `cork()` 被调用而没有相应 `uncork()` 的次数：

```js
writable.cork();
writable.cork();
console.log(writable.writableCorked); // 2
```

这主要用于调试 `cork()`/`uncork()` 的使用。

`writableObjectMode` 属性告诉你流是否处于 `objectMode`：

```js
console.log(writable.writableObjectMode);
```

这在构建期间设置，不会改变。当编写处理字节流和对象流的通用代码时很有用。

这些属性让你能够了解流的内部状态。在大多数应用程序代码中，你不需要它们。但在调试流问题或实现通用流工具时，它们是 **无价的**。

## 深入探讨：写入请求队列

理解在队列结构中如何内部管理写入请求有助于你推理性能和内存使用。

当你调用 `write()` 时，数据块及其相关元数据被封装在一个 **写入请求对象** 中。此对象包含：

+   数据块本身（`Buffer`、字符串或对象）

+   编码（如果它是字符串）

+   写入完成时调用的回调（可选）

+   队列中下一个写入请求的引用

这些写入请求对象形成一个 **链表**。链表的头部是当前正在处理写入请求。尾部是最最近添加的请求。当你调用 `write()` 时，一个新的写入请求被追加到尾部。

当 `_write()` 完成（当其回调被调用时），当前写入请求从列表的头部移除，下一个请求成为新的头部。如果有下一个请求，则再次调用 `_write()` 并使用该请求的数据块。如果没有下一个请求，队列就为空，流等待更多的 `write()` 调用。

这个队列结构对内存使用有影响。每个写入请求对象都有 **开销** - 指针、元数据、闭包。对于小写入，这种开销相对于数据块大小可能是显著的。如果你使用 1 字节的数据块调用 `write()` 一百万次，你将有一百万个写入请求对象，每个对象有 50-100 字节的开销。那只是队列结构就需要 50-100 MB 的内存，尽管实际数据只有 1 MB。

这就是为什么 **批量处理小写入可以提高性能**。与其进行一百万个 1 字节写入，不如进行 1000 个 1KB 写入。数据是相同的，但队列开销是 1/1000。

`cork()` 和 `uncork()` 方法与此队列交互。当你封口一个流时，`write()` 仍然创建写入请求对象并将它们追加到队列中，但不会调用 `_write()`。请求累积。当你解封时，如果实现了 `_writev()`，所有累积的请求将在一个调用中传递给 `_writev()`。否则，为每个请求重复调用 `_write()`。

封口写入序列：

```js
writable.cork();
writable.write("a"); // creates request, queues it
writable.write("b"); // creates request, queues it
writable.write("c"); // creates request, queues it
writable.uncork(); // calls _writev(["a", "b", "c"])
```

无 `cork()`：

```js
writable.write("a"); // creates request, calls _write("a")
writable.write("b"); // creates request, queues it
writable.write("c"); // creates request, queues it
// as each _write completes, the next is called
```

区别在于，在封口的情况下，如果 `_writev()` 被高效实现，**所有三个数据块都可以在一个 I/O 操作中写入**。在未封口的情况下，每个数据块分别写入，导致需要进行三个 I/O 操作。

对于某些目的地，如网络套接字或文件，显著减少 I/O 操作的数量可以显著提高吞吐量。对于其他目的地，如内存数组，这并不重要。这就是为什么 `_writev()` 是可选的——它是一种只有在目的地从批量处理中受益时才值得优化的优化。

## 可写入流的对象模式

我们主要讨论了字节流，但`objectMode`是构建数据处理管道的关键特性。在`objectMode`中，`Writable`流的行为以特定的方式改变。

在`objectMode`模式下，`highWaterMark`是以**对象计数**来衡量的，而不是以字节计数。默认值为 16 个对象。当你向一个`objectMode`流写入对象时，流会根据每个对象在内存中的大小，将缓冲区计数增加 1。

这意味着在`objectMode`中的`highWaterMark**不是一个内存限制**。它是一个计数限制。如果你写入 16 个对象，每个对象是一个 10MB 的缓冲区，那么流已经缓冲了 160MB 的数据，尽管`highWaterMark`是 16。

这是故意的。`objectMode`是为了那些每个数据块代表一个**逻辑工作单元**的场景而设计的，你希望限制正在传输的单元数量，而不是总字节数。例如，如果你正在处理数据库行，你可能希望一次缓冲 100 行，无论每行是 100 字节还是 10KB。

实现`objectMode`的`Writable`与字节流相同，只是你在选项中设置`objectMode: true`：

```js
class RowWriter extends Writable {
 constructor(db, options) {
 super({ ...options, objectMode: true });
 this.db = db;
 }
 async _write(row, encoding, callback) {
 try {
 await this.db.insert(row);
 callback();
 } catch (err) {
 callback(err);
 }
 }
}
```

每次调用`write()`都会传递一个行对象。`_write()`方法接收行并将其插入数据库。在`objectMode`中，`encoding`参数被忽略（它总是`'buffer'`），但它仍然被传递以保持签名的一致性。

一种常见的模式是将字节流转换为`objectMode`流，使用`Transform`。例如，解析 JSON 行：

```js
import { Transform } from "stream";
 class JSONLineParser extends Transform {
 constructor(options) {
 super({ ...options, objectMode: true });
 this.buffer = "";
 }
 _transform(chunk, encoding, callback) {
 this.buffer += chunk.toString();
 const lines = this.buffer.split("\n");
 this.buffer = lines.pop(); // incomplete line
 for (const line of lines) {
 if (line.trim()) {
 try {
 this.push(JSON.parse(line));
 } catch (err) {
 return callback(err);
 }
 }
 }
 callback();
 }
}
```

这个`Transform`读取字节数据块，将它们累积成行，并推送解析后的 JSON 对象。输出是一个`objectMode`流，尽管输入是字节流。

`objectMode`流对于构建可组合的数据管道至关重要，其中每个阶段处理逻辑记录而不是字节块。它们在**ETL**（提取、转换、加载）系统、日志处理、数据导入/导出以及处理结构化数据的任何地方都很常见。

## `_final()`钩子的详细说明

`_final()`方法值得更多关注，因为它经常被误解。它**不是一个析构函数**。当流被销毁时不会调用它。它在流正常结束时被调用，在所有写入完成后，但在`finish`事件发出之前。

`_final()`方法的目的是在流被认为完成之前执行任何清理或最终写入。例如，如果你正在写入一个压缩文件，`_final()`就是写入最终压缩脚注的地方。如果你正在累积数据以进行批量写入，`_final()`就是刷新那个批次的地方。

一个累积写入并在批次中刷新的`Writable`：

```js
class BatchingWritable extends Writable {
 constructor(batchSize, options) {
 super(options);
 this.batchSize = batchSize;
 this.batch = [];
 }
 _write(chunk, encoding, callback) {
 this.batch.push(chunk);
 if (this.batch.length >= this.batchSize) {
 this._flush(callback);
 } else {
 callback();
 }
 }
 _final(callback) {
 if (this.batch.length > 0) {
 this._flush(callback);
 } else {
 callback();
 }
 }
 _flush(callback) {
 const data = Buffer.concat(this.batch);
 this.batch = [];
 // write data to destination
 callback();
 }
}
```

`_write()`方法将数据块添加到批次中。当批次达到`batchSize`时，它会被刷新。`_final()`方法确保在流结束时，任何剩余的未完成批次都会被刷新。

**如果没有 `_final()`，当流结束时，部分批次将会丢失**。`finish` 事件会触发，但最后几个块没有被写入目标。这是自定义可写流中常见的**错误**，这些流执行批处理或缓冲。

`_final()` 回调必须被调用，就像 `_write()` 回调一样。如果你不调用它，`finish` 事件永远不会触发，流会挂起。如果你向回调传递一个错误，流会发出一个 `error` 事件而不是 `finish`。

异步版本：

```js
async _final(callback) {
 try {
 await this.flushAsync();
 callback();
 } catch (err) {
 callback(err);
 }
}
```

或者，由于 Node.js 支持从流方法返回承诺，你可以省略回调：

```js
async _final() {
 await this.flushAsync();
}
```

如果 `_final()` 返回一个承诺，Node.js 会等待它解析后再发出 `finish`，或者如果它拒绝，则发出 `error`。

## 高级自定义可写示例：速率限制写入器

一个更复杂的自定义可写流，它将写入速率限制到目标：这演示了几个高级概念：背压管理、时间控制和队列操作。

```js
import { Writable } from "stream";
 class RateLimitedWritable extends Writable {
 constructor(dest, bytesPerSecond, options) {
 super(options);
 this.dest = dest;
 this.bytesPerSecond = bytesPerSecond;
 this.tokens = bytesPerSecond;
 this.lastRefill = Date.now();
 }
 _write(chunk, encoding, callback) {
 this._refillTokens();
 if (this.tokens >= chunk.length) {
 this.tokens -= chunk.length;
 this.dest.write(chunk, encoding, callback);
 } else {
 const wait = ((chunk.length - this.tokens) / this.bytesPerSecond) * 1000;
 setTimeout(() => {
 this.tokens = 0;
 this.dest.write(chunk, encoding, callback);
 }, wait);
 }
 }
 _refillTokens() {
 const now = Date.now();
 const elapsed = (now - this.lastRefill) / 1000;
 this.tokens = Math.min(
 this.bytesPerSecond,
 this.tokens + elapsed * this.bytesPerSecond
 );
 this.lastRefill = now;
 }
}
```

这个流使用**令牌桶算法**来速率限制写入。它维护一个可用令牌（字节）的计数。每秒添加 `bytesPerSecond` 令牌。在写入时，如果有足够的令牌，写入会立即发生。如果没有，写入会延迟，直到有足够的时间积累所需的令牌。

`_refillTokens()` 方法在每次写入之前被调用，根据经过的时间添加令牌。`_write()` 方法检查是否有足够的令牌，如果没有，则使用 `setTimeout` 将写入操作安排在稍后进行。

这种模式可以适应各种速率限制场景：限制每秒对 API 的请求次数、节流日志写入、调整数据导出等。

注意，如果写入被延迟，此实现不会立即调用回调。回调会被传递给 `setTimeout`，最终传递给目标 `write()` 调用。这意味着可写流内部队列在等待时会被阻塞。这是**正确的行为** - 如果正在速率限制写入，流应该发出背压信号，这是自然发生的，因为回调只有在延迟写入完成后才会被调用。

## 多个写入器之间的背压

我们之前提到过这一点，但值得更深入地探讨。当多个生产者向单个可写流写入时，背压变得更加复杂。

每个生产者独立调用 `write()`。每个生产者都会看到返回值，指示缓冲区是否已满。但是 `drain` 事件会广播给所有监听者。当一个生产者因为 `write()` 返回 `false` 而暂停时，然后 `drain` 事件触发，所有暂停的生产者会同时恢复。

这可能导致**雷鸣般的群体问题**。假设有 100 个生产者都暂停等待 `drain`。当 `drain` 触发时，所有 100 个生产者都会恢复并立即调用 `write()`。刚刚排空的缓冲区会立即再次填满，所有 100 个生产者再次暂停。

流在排空和满载之间波动，没有取得任何进展。这是一个极端情况，但它说明了真实问题：在多个生产者之间协调背压是非平凡的。

一个解决方案是在更高层次使用队列。不是让 100 个生产者直接写入流，而是让他们将数据入队，然后让单个消费者从队列中读取并写入流。单个消费者处理背压，队列协调生产者。

另一个解决方案是使用信号量或类似的协调原语来限制可以同时写入的生产者数量。一次只允许 N 个生产者写入。当一个完成时，另一个获得机会。这防止了暴风雨般的群体。

在实践中，最简单的解决方案通常是**避免许多并发写入单个流**。如果你需要从多个来源聚合写入，考虑使用更高层次的抽象，比如内部协调写入的日志库，或者交错多个来源数据的流多路复用器。

**背压是针对流的信号，而不是针对生产者的信号**。如果你有多个生产者，你需要更高层次的协调来避免病态行为。

## 可写流的内存分析

在使用可写流的代码中调试内存问题需要系统性的方法。假设你的应用程序的内存使用量随时间增长，你怀疑它与流有关。你如何诊断它？

首先，检查你是否尊重背压。在你的`write()`调用中添加日志：

```js
const ok = writable.write(chunk);
if (!ok) {
 console.log("Backpressure! Buffer size:", writable.writableLength);
}
```

如果你看到“Backpressure!”消息，但你的代码没有暂停，那就是问题所在。你正在**忽略背压**。

如果你正确地暂停了，但内存仍在增长，请定期检查`writableLength`：

```js
setInterval(() => {
 console.log("Buffer size:", writable.writableLength);
}, 1000);
```

如果这个值持续增加，流的缓冲区正在增长，这意味着目标比生产者慢。这可能是有预期的（如果目标是合法慢速的），或者可能表明目标存在问题（如果它被阻塞或停滞）。

使用 Node.js 的内置**堆快照功能**来查看内存分配的位置：

```js
const v8 = require("v8");
const fs = require("fs");
 const snapshot = v8.writeHeapSnapshot();
console.log("Heap snapshot written to", snapshot);
```

在 Chrome DevTools 中加载快照以查看对象分配。寻找大量`Buffer`对象或写入请求对象。如果你看到与流相关的大量小对象，那么你已经找到了泄漏。

另一个有用的工具是`--trace-gc`标志，它记录垃圾回收事件：

```js
node --trace-gc app.js
```

如果你看到频繁的 GC 周期和高内存使用，尽管 GC 正在运行，这意味着你分配的速度比 GC 可以回收的速度快，这与无界缓冲区增长一致。

对于生产监控，跟踪`writable.writableLength`作为一个指标。如果它始终接近`writableHighWaterMark`，你经常遇到背压，这可能会表明你的管道存在瓶颈。

## 实用模式：组合多个可写对象

有时候你需要同时将相同的数据写入多个目的地。例如，写入文件和数据库，或者将数据发送到多个网络端点。你该如何构建这种结构？

一种方法是手动写入多个流：

```js
function writeToAll(writables, chunk) {
 const results = writables.map((w) => w.write(chunk));
 return results.every((r) => r === true);
}
 const ok = writeToAll([writable1, writable2], chunk);
if (!ok) {
 // at least one stream signaled backpressure
}
```

这可行，但处理背压是棘手的。如果一个流信号背压而其他流没有，你应该暂停吗？如果你等待所有流都排空，那么快速的流会被不必要地减慢。如果你不等待，慢速流的缓冲区会增长。

一个更好的方法是使用一个内部处理此功能的扇出流：

```js
class FanOutWritable extends Writable {
 constructor(destinations, options) {
 super(options);
 this.destinations = destinations;
 }
 _write(chunk, encoding, callback) {
 let pending = this.destinations.length;
 let error = null;
 const done = (err) => {
 if (err) error = err;
 if (--pending === 0) {
 callback(error);
 }
 };
 this.destinations.forEach((dest) => {
 dest.write(chunk, encoding, done);
 });
 }
}
```

这个流同时写入所有目的地，并在调用回调之前等待所有目的地完成。如果任何目的地出错，错误会被传递到回调。背压自然处理——回调只有在所有目的地完成时才会被调用，所以如果其中一个较慢，`FanOutWritable` 的缓冲区会填满，向其生产者发出背压信号。

这种模式对于将日志记录到多个输出、复制数据或广播事件很有用。

## 选择可写流的 `highWaterMark`

我们在本章中多次提到了 `highWaterMark`，但你如何选择正确的值？

**默认的 16KB** 对于大多数场景来说是一个合理的平衡。它足够大，可以避免典型的写入模式产生过多的背压信号，但足够小，以至于你不会缓冲不合理数量的数据。

如果你正在写入大块数据，考虑 **将 `highWaterMark` 增加以匹配**。如果你的块大小为 1MB，16KB 的 `highWaterMark` 意味着每次写入都会触发背压信号，这是低效的。将 `highWaterMark` 设置为 2MB 或 4MB 给流留出一些空间。

如果你在一个 **内存受限的环境** 中运行（如具有严格限制的 Docker 容器或嵌入式设备），考虑将 `highWaterMark` 减小以减少内存占用。将其设置为 4KB 或 8KB 意味着在任何给定时间缓冲的数据更少。

如果你正在同时处理许多流，将每个流的 `highWaterMark` 乘以并发流的数量，以估计总缓冲内存使用量。如果你有 1000 个并发的 HTTP 响应流，并且每个流都有一个 16KB 的 `highWaterMark`，那么仅用于流缓冲的潜在缓冲内存就有 16MB。如果这太多，降低 `highWaterMark`。

对于 `objectMode` 流，`highWaterMark` 控制的是 **对象数量**，而不是字节大小。根据你希望有多少对象在传输中选择一个值。对于数据库写入，100 或 1000 可能是合理的。对于文件解析，10 或 50 可能是合适的。没有通用的答案——这取决于你对象的大小和你的吞吐量需求。

一种技术是使 `highWaterMark` **可配置** 并根据观察到的性能进行调整。从默认值开始，测量吞吐量和内存使用情况，并根据需要进行调整。

记住`highWaterMark`是一个**阈值，而不是限制**。缓冲区可以超过`highWaterMark`，尤其是如果数据块很大。所以不要将`highWaterMark`视为严格的内存预算——将其视为一个信号点。
