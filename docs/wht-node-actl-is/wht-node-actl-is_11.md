# 可读流

> [`www.thenodebook.com/streams/readable-streams`](https://www.thenodebook.com/streams/readable-streams)

现在你理解了流存在的意义。你知道它们解决了处理大型数据集而不必将所有内容加载到内存中的问题。你已经看到了推送和拉模型之间的概念差异，并且知道 Node.js 流结合了这两种方法。现在的问题是：你如何在代码中实际使用可读流，更重要的是，它们在内部是如何工作的？

可读流是 Node.js 中流式处理的入口点。它们产生数据——来自文件、来自网络连接、来自内存结构、来自任何地方。理解可读流如何管理其内部状态、如何缓冲数据以及如何与消费者通信对于有效地使用流至关重要。

我们将有条不紊地构建这种理解。首先，我们将探索 `Readable` 流类本身——它的选项、它的契约、它的事件。然后，我们将讨论两种操作模式以及它们之间转换的触发因素。之后，我们将详细检查内部缓冲，因为这是内存管理发生的地方，也是 `highWaterMark` 真正起作用的地方。最后，我们将实现我们自己的可读流并探索所有消费它们的方式。到那时，你将有一个完整的心智模型，了解数据是如何从一个源通过可读流传输到消费者的。

## 可读流类

让我们从对象本身开始。当你从 Node 导入 `stream` 并访问 `stream.Readable` 时，你得到的是一个扩展了 `EventEmitter` 的类。这种继承关系非常重要。每个可读流本质上都是一个事件发射器，这意味着它可以发射诸如 `data`、`end`、`error` 和 `readable` 等事件。可读流的大部分行为都是通过这些事件来表达的。

在应用程序代码中直接创建可读流是不常见的。更常见的是，你从 Node.js API 如 `fs.createReadStream()` 或 `http.IncomingMessage` 接收可读流。但当你创建一个时，无论是通过扩展类还是使用 `new stream.Readable(options)`，你都会提供一个配置对象，该对象控制流的行为。

最重要的选项是 `highWaterMark`。这是一个表示流在停止从底层源拉取数据之前内部可以缓冲的最大字节数（如果你处于 `objectMode`，则是对象）。想象一下，这是流用于缓冲的内存预算。默认值是 65536 字节，即 64 千字节。这个默认值不是随机的数字——它代表了 Node 团队通过实验和生产使用所确定的内存使用和系统调用效率之间的平衡。

这为什么很重要？因为可读流不仅仅是直接从源到消费者的数据。它维护一个内部缓冲区。当消费者准备好数据时，它会从这个缓冲区中拉取。当缓冲区变低时，流会从底层源请求更多数据以填充它。`highWaterMark`控制流何时决定“我的缓冲区已经足够满了，我应该停止从源请求更多数据。”如果缓冲区包含等于或超过`highWaterMark`的字节，流将不会从源请求更多数据，直到缓冲区低于该阈值。

让我们看看它是什么样子：

```js
import { Readable } from "stream";
 const readable = new Readable({
 highWaterMark: 1024, // 1KB buffer
});
```

在这里，我们创建了一个具有 1KB 缓冲阈值的可读流。如果这个流从文件中读取数据，它将不会请求超过消费者消费速率的 1KB 数据。

另一个关键选项是`objectMode`。默认情况下，可读流与`Buffer`对象和字符串一起工作。但有时你想要流式传输任意 JavaScript 对象。将`objectMode: true`设置为流的行为。它不再缓冲字节，而是缓冲对象。`highWaterMark`不再表示字节数，而是表示对象数。在`objectMode`中，默认的`highWaterMark`是 16 个对象，而不是 64KB。

```js
const objectStream = new Readable({
 objectMode: true,
 highWaterMark: 100, // buffer up to 100 objects
});
```

当你构建处理结构化数据的管道时，这很有用。例如，如果你正在从数据库中读取行并将其通过转换阶段流式传输，`objectMode`使得每一行成为流中的单个单元，这在概念上比将行转换为缓冲区再转换回来要干净。

`encoding`选项是另一个配置细节。默认情况下，当你从可读流中读取时，它会发射`Buffer`对象。如果你设置了一个编码，如`'utf8'`，流会自动使用该编码将这些缓冲区转换为字符串。这纯粹是一种便利 - 你始终可以自己调用`buffer.toString('utf8')` - 但当你知道你总是在处理文本时，它可以使代码更简洁。

```js
const textStream = new Readable({
 encoding: "utf8",
});
```

现在这个流发射数据时，它将发射字符串，而不是缓冲区。

这些是基础选项。还有其他选项 - `read`（一个用于实现读取逻辑的函数）、`destroy`（一个清理函数）和`autoDestroy`（是否在流结束后自动销毁流） - 但`highWaterMark`、`objectMode`和`encoding`是配置最频繁的选项，它们对流的运行时行为影响最大。

## 事件

可读流主要通过事件与外界通信。让我们检查每个事件，了解它何时触发以及它的含义。

`data`事件是最直接的。当一个可读流处于流动模式时，每当它有可用数据时，它都会发射`data`事件。每个`data`事件携带一块数据 - 要么是一个`Buffer`，要么是一个字符串（如果设置了`encoding`），要么是一个对象（如果`objectMode`为真）。

```js
readable.on("data", (chunk) => {
 console.log(`Received ${chunk.length} bytes`);
});
```

当你将一个`data`事件监听器附加到一个可读流上时，你实际上是隐式地将流切换到流动模式。这很重要。监听`data`的行为会改变流的行为。一旦数据可用，就会将其推送到你的监听器。你不需要拉取。流会推送。

当流没有更多数据提供时，`end`事件会触发。底层源已被完全消耗。如果你正在读取文件，当到达文件末尾时，`end`事件会触发。如果你正在从 HTTP 响应中读取，当服务器完成发送响应体时，`end`事件会触发。此事件没有参数。它只是一个信号：“我完成了。”

```js
readable.on("end", () => {
 console.log("No more data");
});
```

当发生错误时，`error`事件会触发。可能你正在读取的文件在读取过程中被删除了。可能网络连接断开了。可能底层源由于某种原因抛出了错误。当发生错误时，流会发出一个带有错误对象的`error`事件。如果你没有附加`error`事件监听器，Node.js 会抛出错误，这可能会导致你的应用程序崩溃。这就是为什么你应该**始终**将错误处理程序附加到流上的原因。

```js
readable.on("error", (err) => {
 console.error("Stream error:", err);
});
```

`readable`事件更为微妙。当流中有数据可供读取时，它会触发。这个事件主要与流处于暂停模式相关（我们在上一章中提到了**流动**和**暂停**模式，不用担心，我们很快会再次澄清模式）。`readable`事件是一个信号，表示“我的内部缓冲区中有数据。如果你调用`read()`，你会得到一些东西。”

```js
readable.on("readable", () => {
 let chunk;
 while ((chunk = readable.read()) !== null) {
 console.log(`Read ${chunk.length} bytes`);
 }
});
```

下面是发生的事情。`readable`事件触发。在处理程序内部，我们循环调用`readable.read()`，从内部缓冲区中拉取块，直到`read()`返回`null`，这意味着缓冲区为空。这是一种基于拉的消费模式，与`data`事件的基于推的模式相反。

此外，当流及其任何底层资源被关闭时，也会触发一个`close`事件。这与`end`事件不同。`end`事件意味着“没有更多数据”，但资源可能仍然打开。`close`事件意味着“资源已被释放”。并非所有流都会发出`close`事件，在许多情况下你不需要监听它，但如果你需要知道清理何时完成，它就在那里。

这些事件构成了可读流的 API 表面。无论你是消费它们还是实现它们，你与可读流的交互都将围绕这些事件及其语义展开。

## 流动模式与暂停模式（回顾）

现在让我们来探讨我在上一章中简要解释的概念：操作模式。在任何给定时间，每个可读流都处于两种模式之一：**流动模式**或**暂停模式**。模式决定了数据如何从流的内部缓冲区移动到您的代码中。

在 **暂停模式** 下，数据不会自动流动。流会填充其内部缓冲区直到 `highWaterMark`，但它不会将数据推送到你那里。你必须显式调用 `readable.read()` 来从缓冲区中拉取数据。暂停模式是在创建新的可读流时的默认状态。

在 **流动模式** 下，数据自动流动。一旦内部缓冲区中有数据可用，流就会发出带有数据块的事件。你不需要调用 `read()`。数据会自动推送到你那里。

为什么有两个模式？因为不同的消费模式从不同的控制流程中受益。有时你希望流尽可能快地将数据推送到你那里，而你将通过暂停和恢复流来处理背压。其他时候，你希望对数据何时被拉取有精细的控制，正好在你准备好读取更多数据时进行读取。暂停模式为你提供了这种控制。流动模式在准备好以尽可能快的速度处理数据时，优化了简单性和吞吐量。

让我们看看你是如何在不同模式之间切换的。当创建一个可读流时，它从暂停模式开始。你可以通过以下任何一种方式切换到流动模式：

+   添加 `data` 事件监听器

+   调用 `resume()` 方法

+   调用 `pipe()` 方法将流管道连接到可写流

相反，你可以通过调用 `pause()` 方法（但只有当没有 `pipe()` 目标时）从流动模式切换回暂停模式。

这里有一个微妙之处。如果你已经使用 `pipe()` 将可读流管道连接到可写流，调用 `pause()` 实际上并不会暂停流。`pipe()` 机制内部管理流控制，并且它将根据可写流的背压信号继续运行。这是设计上的考虑 - `pipe()` 是一个更高级别的抽象，为你处理背压，而手动 `pause()`/`resume()` 调用会干扰这一点。

让我们看看暂停模式下的数据消费：

```js
const readable = getReadableStream();
 readable.on("readable", () => {
 let chunk;
 while ((chunk = readable.read()) !== null) {
 processChunk(chunk);
 }
});
 readable.on("end", () => {
 console.log("Stream ended");
});
```

在这里，流保持在暂停模式。`readable` 事件告诉我们数据可用。我们反复调用 `read()` 直到它返回 `null`。我们控制着何时拉取数据。

现在，让我们看看流动模式下的数据消费：

```js
const readable = getReadableStream();
 readable.on("data", (chunk) => {
 processChunk(chunk);
});
 readable.on("end", () => {
 console.log("Stream ended");
});
```

一旦我们添加了 `data` 监听器，流就会切换到流动模式。数据会自动推送到我们这里。如果 `processChunk()` 很慢，数据将在内存中缓冲，等待处理，除非我们通过暂停来实现背压。

这是如何在流动模式下实现背压的：

```js
readable.on("data", (chunk) => {
 const canContinue = processChunk(chunk);
 if (!canContinue) {
 readable.pause();
 // Later, when processing catches up:
 // readable.resume();
 }
});
```

当 `processChunk()` 指示它跟不上的情况下，我们暂停流。这停止了 `data` 事件的流动。稍后，当处理赶上（可能在回调或已解决的承诺中）时，我们调用 `resume()` 来重新启动流动。

消费流还有第三种、较少见的方法：在暂停模式下，没有 `readable` 监听器的 `read(size)` 方法。你可以在任何时候直接调用 `readable.read(size)` 来从内部缓冲区拉取特定数量的字节。如果缓冲区没有那么多字节，`read()` 返回可用的任何内容，或者如果缓冲区为空，则返回 `null`。

```js
const chunk = readable.read(100);
if (chunk !== null) {
 console.log(`Read ${chunk.length} bytes`);
}
```

这让你能够精确控制每次拉取多少数据，这在实现具有固定大小头或结构的协议时可能很有用。

关键在于这些模式反映了不同的内存和并发管理策略。暂停模式赋予你控制权，并使背压变得明确。流动模式在你处理能力能够跟上数据速率时，提供了简单性和性能。理解何时以及如何使用每种模式是掌握流的一部分。

## 内部缓冲

让我们进一步探讨在可读流内部真正发生的事情。当你创建一个可读流并开始从中读取数据时，数据不会直接从底层源（文件、套接字、生成器）传送到你的消费代码。它通过流维护的内部缓冲区传递。

内部缓冲是一个块队列。当流从底层源拉取数据时，这些块会被添加到缓冲区。当你从流中消费数据（在暂停模式下通过调用 `read()` 或在流动模式下接收 `data` 事件）时，块会从缓冲区中移除。当源生产速度超过消费者消费速度时，缓冲区会增长；当消费者赶上时，缓冲区会缩小。

缓冲区不是一个单一的 `Buffer` 对象。实际上，它是一个块数组（之前是一个链表，但为了更好的性能进行了更改）。每个块都保留在其原始分配的 `Buffer` 中，而数组只是跟踪序列。虽然数组需要偶尔调整大小，但 JavaScript 的数组实现处理得非常高效，更好的缓存局部性和更简单的迭代通常超过了偶尔重新分配的成本。

你可以使用 `_readableState` 属性来检查缓冲区的当前状态。这个属性在技术上属于内部属性（下划线前缀表示），但它对调试和理解正在发生的事情非常有用。

```js
const state = readable._readableState;
console.log(`Buffer length: ${state.length} bytes`);
console.log(`Buffer count: ${state.buffer.length} chunks`);
console.log(`highWaterMark: ${state.highWaterMark} bytes`);
```

`state.length` 告诉你当前缓冲了多少字节。`state.buffer` 是数组本身，而 `state.buffer.length` 告诉你数组中有多少块。`state.highWaterMark` 是我们配置的阈值，如果没有配置，则为默认的 64KB。

现在，这是关键机制。当缓冲区的总长度低于 `highWaterMark` 且流需要更多数据（无论是由于消费者正在读取还是因为流处于流动模式）时，流会调用一个内部方法，称为 `_read()`。此方法负责从底层源获取更多数据并将其推入缓冲区。如果你正在实现自定义 Readable 流，你需要提供 `_read()` 实现如果你正在使用内置流，如 `fs.createReadStream()`，Node 内部提供了 `_read()` 实现。

当 `_read()` 被调用时，它会被告知：“缓冲区有空间。请获取更多数据。” `_read()` 的实现应该从源获取数据并使用 `push()` 方法将其推入缓冲区。以下是一个简化的版本：

```js
class MyReadable extends Readable {
 _read(size) {
 const chunk = this.getDataFromSomeTypeOfSource(size);
 if (chunk) {
 this.push(chunk); // Adds to internal buffer
 } else {
 this.push(null); // Signals end of data
 }
 }
}
```

`_read(size)` 方法接收一个大小提示 - 通常为 `highWaterMark` 值 - 建议它应该获取多少数据。这只是一个提示。你可以推送更多或更少的数据。流会进行适配。但尊重这个提示有助于优化 I/O 效率。

当你调用 `this.push(chunk)` 时，会发生几件事情。首先，块被添加到内部缓冲区。其次，如果流处于流动模式，块可能会立即作为 `data` 事件发出（如果有一个消费者准备好，则完全绕过缓冲区）。第三，如果流处于暂停模式，可能会发出一个 `readable` 事件来表示数据可用。

重要的是，`push()` 返回一个布尔值。如果 `push()` 返回 `false`，则表示缓冲区已达到或超过 `highWaterMark`（具体来说，当 `state.length >= state.highWaterMark` 时），并且流正在请求源停止产生数据。作为回应，你的 `_read()` 实现应该停止从源获取数据。流将在缓冲区再次低于 `highWaterMark` 时稍后再次调用 `_read()`。

以下是一个更完整的示例：

```js
class FileReader extends Readable {
 constructor(fd, options) {
 super(options);
 this.fd = fd;
 }
 _read(size) {
 const buffer = Buffer.allocUnsafe(size);
 fs.read(this.fd, buffer, 0, size, null, (err, bytesRead) => {
 if (err) {
 this.destroy(err);
 } else if (bytesRead === 0) {
 this.push(null); // EOF
 } else {
 this.push(buffer.slice(0, bytesRead));
 }
 });
 }
}
```

这是一个简化的文件读取器。当 `_read()` 被调用时，它从文件描述符读取 `size` 字节并将它们推入流。如果 `fs.read()` 返回零字节，则表示已到达文件末尾，因此我们推送 `null` 来表示 EOF。如果发生错误，我们使用错误销毁流。

Node.js 流实现保证在调用 `push()` 之前不会再次调用 `_read()`，因此尽管 `fs.read()` 是异步的，我们也不需要跟踪一个标志来防止重叠调用。流的内部状态机会自动处理这一点。背压也会自动处理 - 只有当缓冲区低于 `highWaterMark` 时才会再次调用 `_read()`，确保即使与异步数据源一起也能尊重背压。

缓冲区的行为在 `objectMode` 和字节模式之间也有所不同。在字节模式下，缓冲区跟踪缓冲的总字节数，并将其与基于字节的 `highWaterMark` 进行比较。在 `objectMode` 中，缓冲区跟踪缓冲的对象数量，并将其与基于对象计数的 `highWaterMark` 进行比较。内部使用相同的结构，但会计方式不同。

一个额外的细节。流不仅在您调用 `read()` 或 `data` 事件触发时清空缓冲区。还有一个内部跟踪的“读取状态”的概念。如果流正在积极读取（意味着 `_read()` 已被调用且尚未推送新数据或推送 `null`），则流不会在当前读取完成之前再次调用 `_read()`。这防止了重复读取，并防止源被并发读取请求淹没。

所有这些缓冲机制都是为了平滑源数据速率和消费者消费速率之间的不匹配。如果源以突发方式产生数据（例如，从接收数据包的网络套接字读取），缓冲区会累积这些突发，以便消费者看到稳定的流。如果消费者偶尔暂停（例如，等待数据库写入完成），缓冲区会保留数据，直到消费者再次准备好。`highWaterMark` 控制这个缓冲区的大小，这直接控制了内存使用和吞吐量之间的权衡。

## 实现自定义可读流

现在我们已经了解了内部机制，让我们来实现自己的可读流。这比消费流要少见得多，但对于构建库、创建自定义数据源或深入理解流的行为非常重要。

标准方法是扩展 `Readable` 类并实现 `_read()` 方法。让我们从一个简单的例子开始：一个从 1 到 N 发出数字的流。

```js
import { Readable } from "stream";
 class CounterStream extends Readable {
 constructor(max, options) {
 super(options);
 this.max = max;
 this.current = 1;
 }
 _read() {
 if (this.current <= this.max) {
 this.push(String(this.current));
 this.current++;
 } else {
 this.push(null);
 }
 }
}
```

这个流将每个数字作为字符串推送。当计数器超过 `max` 时，它会推送 `null` 来表示结束。注意我们并没有检查 `push()` 的返回值。因为我们正在同步产生数据，并且流在需要更多数据时调用 `_read()`，所以流控已经由流内部逻辑处理。如果缓冲区满了，流不会再次调用 `_read()`，直到它被清空。

让我们消费这个流：

```js
const counter = new CounterStream(5);
 counter.on("data", (chunk) => {
 console.log(`Received: ${chunk}`);
});
 counter.on("end", () => {
 console.log("Counter ended");
});
```

输出：

```js
Received: 1
Received: 2
Received: 3
Received: 4
Received: 5
Counter ended
```

现在我们来实现一个更现实的东西：一个从文本文件中读取行的流。当处理大型日志文件或 CSV 文件时，这是一种常见的模式。

ℹ️注意

如果您不理解下面与 'fs' API 相关的代码，请不要担心。我们将在下一章深入探讨文件。

```js
import { Readable } from "stream";
import fs from "fs";
 class LineStream extends Readable {
 constructor(filePath, options) {
 super(options);
 this.fd = fs.openSync(filePath, "r");
 this.buffer = "";
 this.position = 0;
 }
 _read() {
 const chunk = Buffer.alloc(1024);
 const bytesRead = fs.readSync(this.fd, chunk, 0, 1024, this.position);
 if (bytesRead === 0) {
 if (this.buffer.length > 0) {
 this.push(this.buffer);
 }
 this.push(null);
 return;
 }
 this.position += bytesRead;
 this.buffer += chunk.slice(0, bytesRead).toString();
 let lineEnd;
 while ((lineEnd = this.buffer.indexOf("\n")) !== -1) {
 const line = this.buffer.slice(0, lineEnd);
 this.buffer = this.buffer.slice(lineEnd + 1);
 if (!this.push(line)) {
 return;
 }
 }
 }
 _destroy(err, callback) {
 if (this.fd !== undefined) {
 fs.close(this.fd, callback);
 } else {
 callback(err);
 }
 }
}
```

这个流从文件中读取块，将它们累积在一个内部字符串缓冲区中，并将完整的行推送到流中。当它遇到换行符时，它会推送该行（不带换行符）并继续。如果文件结束时缓冲区中还有剩余数据，它会将其作为最后一行推送。

注意 `_destroy()` 方法。这是一个在流销毁时被调用的清理钩子。默认情况下，可读流有 `autoDestroy: true`，这意味着在流结束后（在 `push(null)` 之后）将自动调用 `_destroy()`。我们使用它来关闭文件描述符，确保我们不会泄露文件句柄。我们在关闭之前检查 `this.fd` 是否已定义，以安全地处理边缘情况。

还要注意，在 `while` 循环内部，我们检查 `push()` 的返回值。如果 `push()` 返回 `false`，我们提前从 `_read()` 返回，停止进一步的推送。这尊重了背压。如果消费者暂停或缓冲区填满，我们不会推送更多行，直到再次调用 `_read()`。

正如我们在上一章中看到的，对于许多用例，创建可读流有一个更简单的方法：`stream.Readable.from()`。这个实用函数可以从可迭代对象或异步可迭代对象创建可读流。如果你有一个数组、生成器或异步生成器，你可以用一行代码将其转换为可读流。

```js
import { Readable } from "stream";
 async function* generateNumbers() {
 for (let i = 1; i <= 5; i++) {
 await new Promise((resolve) => setTimeout(resolve, 100));
 yield i;
 }
}
 const stream = Readable.from(generateNumbers());
 stream.on("data", (num) => {
 console.log(`Received: ${num}`);
});
```

这非常方便。`Readable.from()` 方法处理所有繁重的工作。它调用异步生成器的 `next()` 方法，等待承诺解析，将值推入流中，并重复此过程直到生成器完成。如果你正在从结构化数据构建可读流或实现简单的自定义数据源，`Readable.from()` 可以消除手动扩展 `Readable` 的需求。

在实现可读流时，还有一个需要考虑的因素是 **错误处理**。如果在从源获取数据时发生错误，你应该用该错误销毁流。这会停止流，发出 `error` 事件，并清理资源。

```js
_read() {
 this.fetchData((err, data) => {
 if (err) {
 this.destroy(err); // Emits 'error' event
 } else if (data === null) {
 this.push(null); // End of stream
 } else {
 this.push(data);
 }
 });
}
```

调用 `this.destroy(err)` 将流转换为已销毁状态。不会进行更多 `_read()` 调用，并且会发出带有错误对象的 `error` 事件。如果你实现了 `_destroy()`，它将被调用以清理资源。

## 消费模式

我们在本章中看到了一些消费的片段。现在，让我们系统地介绍所有消费可读流的方法，并提供何时使用每种方法的建议。

**基于事件的消费（流动模式）** 是最直接的方式。附加 `data` 和 `end` 监听器，流就会将数据推送到你。

```js
readable.on("data", (chunk) => {
 processChunk(chunk);
});
 readable.on("end", () => {
 console.log("Done");
});
 readable.on("error", (err) => {
 console.error("Error:", err);
});
```

当你的处理速度快时，这种模式简单且性能良好。然而，如果 `processChunk()` 慢或异步，你需要手动通过调用 `pause()` 和 `resume()` 来实现背压，这增加了复杂性。

**异步迭代消费** 是最现代、最直接且最直观的方法。可读流是异步可迭代对象，因此你可以使用 `for await...of` 来消费它们。

```js
try {
 for await (const chunk of readable) {
 await processChunk(chunk);
 }
 console.log("Done");
} catch (err) {
 console.error("Error:", err);
}
```

这种模式自动处理背压。如果 `processChunk()` 返回一个承诺，循环将等待它解析后再拉取下一个块。这意味着流不会推送更多数据，直到你准备好。这是干净、易于推理的，并且对于大多数用例都是推荐的。

**显式 `read()` 消费（暂停模式）** 给你细粒度的控制。当你准备好数据时，你调用 `read()`。

```js
readable.on("readable", () => {
 let chunk;
 while ((chunk = readable.read()) !== null) {
 processChunk(chunk);
 }
});
 readable.on("end", () => {
 console.log("Done");
});
```

你还可以调用 `read(size)` 来拉取特定数量的字节，这在需要读取固定大小头或结构的二进制协议解析中很有用。

```js
const header = readable.read(4);
if (header !== null) {
 const bodyLength = header.readUInt32BE(0);
 const body = readable.read(bodyLength);
 if (body !== null) {
 processMessage(header, body);
 }
}
```

这种模式功能强大但冗长且容易出错。你必须自己管理状态机，处理数据尚未足够可用的情况。

**使用 `pipe()`** 将可读流连接到可写流，并自动处理背压。不用担心，我们有一个专门的子章节介绍管道和可写流。

```js
readable.pipe(writable);
 readable.on("error", (err) => {
 console.error("Read error:", err);
});
 writable.on("error", (err) => {
 console.error("Write error:", err);
});
```

`pipe()` 方法监听可读流上的 `data` 事件，并在可写流上调用 `write()`。如果 `write()` 返回 `false`（表示可写流的缓冲区已满），`pipe()` 在可读流上调用 `pause()`。当可写流发出 `drain` 事件（表示其缓冲区再次有空间）时，`pipe()` 在可读流上调用 `resume()`。这就是 `pipe()` 那么方便的原因，因为它自动处理背压。

然而，`pipe()` 有局限性。错误处理很尴尬，因为错误不会自动传播，所以你必须将错误监听器附加到两个流上。此外，如果在管道传输过程中发生错误，清理可能会很棘手。流可能没有被正确关闭或销毁。

**使用 `stream.pipeline()`** 是 `pipe()` 的现代、健壮的替代方案。它将多个流连接到管道中，并自动处理错误和清理。

```js
import { pipeline } from "stream/promises";
 try {
 await pipeline(readable, writable);
 console.log("Pipeline succeeded");
} catch (err) {
 console.error("Pipeline failed:", err);
}
```

来自 `stream/promises` 的 `pipeline()` 函数返回一个在管道成功完成时解析或如果任何流发出错误时拒绝的承诺。当发生错误时，`pipeline()` 自动销毁管道中的所有流，确保资源被清理。这使得它在生产代码中组合流时成为推荐的方式。

你也可以将转换函数传递给 `pipeline()`，我们将在后续章节中探讨，当我们介绍转换流时。

每种消费模式都有其适用场景。对于简单、快速的处理，基于事件的消费就足够了。对于具有干净背压处理的异步处理，异步迭代是理想的。对于二进制协议解析或细粒度控制，显式 `read()` 是必要的。对于将流连接起来，`pipeline()` 是最安全、最健壮的选择。

## 模式转换和状态管理

我们已经讨论了流动和暂停模式，但让我们明确一下转换发生的时间以及内部状态是什么样的。这可能导致数据丢失或背压不被尊重的 bug。

当创建一个可读流时，它处于暂停模式，其内部状态有一个标志 `state.flowing` 设置为 `null`。这既不是暂停也不是流动 - 它是一个初始状态，其中流尚未开始。

第一次添加 `data` 监听器时，`state.flowing` 变为 `true`，流切换到流动模式。如果内部缓冲区有数据，数据将立即开始流动；如果没有数据，则数据一旦可用，流就会开始流动。

如果你调用 `pause()`，`state.flowing` 变为 `false`。流停止发出 `data` 事件。然而，内部缓冲区会继续填充到 `highWaterMark`。源将继续产生数据，直到缓冲区满，此时 `_read()` 停止被调用。

如果你调用 `resume()`，`state.flowing` 再次变为 `true`。流从缓冲区开始发出 `data` 事件，如果缓冲区排空到低于 `highWaterMark`，则调用 `_read()` 从源获取更多数据。

如果你移除了所有 `data` 监听器（并且流没有被管道传输到任何地方），`state.flowing` 仍然保持 `true`。这是一个细微差别：流在结构上保持在流动模式，但没有监听器附加，`data` 事件没有地方去。流将继续排空其缓冲区并调用 `_read()`，但发出的数据实际上消失了。如果你稍后附加一个新的 `data` 监听器，流将立即开始向它发出事件（你不会收到在没有任何监听器附加时已发出的数据）。要实际暂停流并停止处理数据，你必须显式调用 `pause()`，这将 `state.flowing` 设置为 `false`。

当你动态添加和删除监听器时，这种区别很重要。简单地移除 `data` 监听器并不会暂停流——它只是移除了事件的目的地。流将继续从其源消费数据。如果你想要暂时停止数据处理（可能是应用回压或等待某些条件），你需要显式 `pause()` 流。

暂停模式（`false`）需要显式调用 `pause()` 或在流被管道传输且目的地应用回压时发生。

你可能会问，为什么？因为如果你正在动态添加和删除监听器，或者如果你正在构建包装流的中间件，你需要理解这些状态转换，以避免意外丢失数据或无法控制流处理数据的时间。

这里有一个片段来观察这些转换：

```js
const readable = getReadableStream();
 console.log(`Initial flowing: ${readable.readableFlowing}`); // null
 readable.on("data", (chunk) => {
 console.log(`Received ${chunk.length} bytes`);
});
 console.log(`After data listener: ${readable.readableFlowing}`); // true
 readable.removeAllListeners("data");
console.log(`After removing listeners: ${readable.readableFlowing}`); // true (still!)
 readable.pause();
console.log(`After pause: ${readable.readableFlowing}`); // false
 readable.resume();
console.log(`After resume: ${readable.readableFlowing}`); // true
```

`readableFlowing` 属性是公开的，可以安全读取。它反映了当前状态：`null`、`false` 或 `true`。

另一个状态考虑因素是 `ended` 标志。一旦流发出 `end` 信号，将不再发出更多数据。如果你尝试从已结束的流中读取，`read()` 将返回 `null`。流将保持这种结束状态，直到它被销毁。即使以某种方式新数据变得可用（在表现良好的流中不应发生这种情况），已结束的流也不会发出它。

也会跟踪 `destroyed` 标志。一旦流被销毁，它将不再发出任何事件（除了 `close`），尝试读取或写入将失败。流释放了其资源，实际上已经死亡。

理解这些状态标志和转换有助于调试像“为什么我的流没有发出数据？”或“为什么我的流卡住了？”这样的问题。通常，流处于你意想不到的状态——当你认为它在流动时被暂停，当你认为还有更多数据时它结束了，或者当你认为它仍然活跃时它被销毁了。

## 实践中的背压

让我们具体看看背压（backpressure）的概念。我们之前已经从抽象的角度讨论过它，但现在让我们看看在实际代码中它是如何表现的，以及如果你忽略了它会发生什么。

假设你正在读取一个大文件，并通过向 API 发起 HTTP 请求来处理每个块。每个请求需要 100 毫秒。以下是忽略背压的简单代码：

```js
const readable = fs.createReadStream("large-file.txt");
 readable.on("data", async (chunk) => {
 await fetch("https://api.example.com/process", {
 method: "POST",
 body: chunk,
 });
});
 readable.on("end", () => {
 console.log("Done");
});
```

这里发生了什么？流尽可能快地读取文件并发出`data`事件。每个事件都会触发异步处理程序，该处理程序启动一个 HTTP 请求。但是处理程序不会阻塞流。流继续发出`data`事件，创建越来越多的并发 HTTP 请求。如果文件很大且块很小，你可能会结束成千上万的飞行请求，耗尽内存和网络资源。

这是一个背压失败。消费者（HTTP 请求逻辑）比生产者（文件读取）慢，但没有机制来减缓生产者的速度。

这是如何通过显式暂停和恢复来修复它的：

```js
const readable = fs.createReadStream("large-file.txt");
 readable.on("data", async (chunk) => {
 readable.pause();
 await fetch("https://thenodebook.com/process", {
 method: "POST",
 body: chunk,
 });
 readable.resume();
});
 readable.on("end", () => {
 console.log("Done");
});
```

现在，一旦发出`data`事件，我们就暂停流。这阻止了进一步的`data`事件。我们异步处理块。一旦 HTTP 请求完成，我们恢复流，允许下一个`data`事件触发。这序列化了处理，确保一次只有一个请求在飞行。文件读取速率与 HTTP 请求速率相匹配。

但说实话，这个模式是**尴尬**且不好的。暂停/恢复调用使逻辑变得混乱，如果发生错误，你可能会忘记恢复，导致流卡住。

更干净的方法是异步迭代：

```js
const readable = fs.createReadStream("large-file.txt");
 for await (const chunk of readable) {
 await fetch("https://thenodebook.com/process", {
 method: "POST",
 body: chunk,
 });
}
 console.log("Done");
```

这通过自动实现相同的背压行为。`for await...of`循环不会在当前迭代完成之前拉取下一个块。如果`fetch()`需要时间，流会等待。生产者的速率与消费者的速率相匹配，没有显式的暂停或恢复调用。

现在，如果你想要受控的并发——比如说，一次最多 5 个请求？仅仅异步迭代无法提供这一点。你需要实现一个并发限制器，这超出了本章的范围，但在生产中这是一个常见的模式，我相当确信你足够聪明可以自己实现它。如果不能，不要担心，像`p-limit`或`async`这样的库提供了这个功能的工具。

关键点是，除非你使用强制执行它的机制，否则背压不是自动的。使用`data`监听器的事件驱动消费默认不强制执行背压。你必须手动使用暂停/恢复来添加它。异步迭代通过设计强制执行背压。使用`pipe()`或`pipeline()`通过监控可写流的状

## 在对象模式下读取

对象模式略微改变了可读流的语义。不再是推送`Buffer`对象或字符串，而是推送任意的 JavaScript 值。`highWaterMark`不再是字节数，而是对象数。让我们看看这在实践中是什么样子。

这是一个发出数据库行的可读流：

```js
import { Readable } from "stream";
 class RowStream extends Readable {
 constructor(db, query, options) {
 super({ ...options, objectMode: true });
 this.db = db;
 this.query = query;
 this.offset = 0;
 }
 async _read() {
 try {
 const rows = await this.db.query(this.query, { offset: this.offset, limit: 100 });
 for (const row of rows) this.push(row);
 rows.length > 0 ? (this.offset += rows.length) : this.push(null);
 } catch (err) {
 this.destroy(err);
 }
 }
}
```

该流以 100 行为一批查询数据库。每一行作为一个 JavaScript 对象被推送。流的内部缓冲区处理背压 - 当它填满时，`_read()`不会再次被调用，直到有空间，自然地暂停数据库查询。消费者看到的是行对象的流：

```js
const stream = new RowStream(db, "SELECT * FROM users");
 for await (const row of stream) {
 console.log(`User: ${row.name}, Email: ${row.email}`);
}
```

当你处理不自然映射到字节的有序数据时，对象模式非常强大。你不必将行序列化为 JSON，然后将 JSON 作为`Buffer`推送，并在消费者那里再次解析它，你只需直接推送对象。这更高效、更简洁。

📌重要

由于`highWaterMark`是对象数，而不是字节数，流的内存使用完全取决于你推送的对象的大小。如果你推送了 16 个对象，每个对象是 10MB，你将缓冲 160MB，即使`highWaterMark`只是 16。Node.js 没有方法来测量任意 JavaScript 对象的字节数，所以它依赖于对象数。这意味着你必须作为程序员，**仔细计算**基于预期对象大小的适当的`highWaterMark`值。

## 边界情况和调试

可读流有几个边界情况，如果你不知道它们，可能会让你陷入困境。我将介绍几个。

**空流。** 可读流可以立即结束而不推送任何数据。如果你在`_read()`中调用`push(null)`而从未推送实际数据，流将发出`end`事件而从未发出`data`事件。这是有效的，有时是故意的（例如，读取空文件），但它可能会让期望至少有一个`data`事件的消费者感到惊讶。

**未处理的`error`事件。** 如果可读流发出`error`事件而没有监听器，Node.js 将抛出错误，可能会使你的应用程序/进程崩溃。始终附加一个`error`监听器，即使它只是记录错误。

**销毁流。** 一旦流被销毁，就无法再从中读取数据。如果你提前销毁流（例如，在仍有缓冲数据时调用 `destroy()`），这些数据就会丢失。如果你需要清理资源但仍然想优雅地发出缓冲数据，请调用 `push(null)` 来发出结束信号，而不是立即销毁。

**混合消费模式。** 如果你将 `readable` 监听器和 `data` 监听器都附加到同一流上，行为可能会令人困惑。附加 `readable` 监听器会阻止流进入流动模式，即使之后附加了 `data` 监听器。`readable` 监听器具有优先权，并且 `data` 事件不会按预期触发。每个流坚持一种消费模式。

**误解 `read(size)`。** 调用 `read(size)` 并不保证你会得到确切的 `size` 字节。你会得到缓冲区中可用的最多 `size` 字节。如果缓冲区中少于 `size` 字节，你会得到那里的所有内容。如果缓冲区为空，你会得到 `null`。不要假设 `read(size)` 会阻塞或等待 - 它是非阻塞的，并立即返回。

**忽略 `readable.destroyed`。** 如果你正在实现与流交互的自定义逻辑，在调用 `read()` 或 `push()` 等方法之前，始终检查 `readable.destroyed`。在已销毁的流上操作可能导致错误或意外行为。

对于调试，`_readableState` 属性非常有价值。它暴露了流的内部状态：

```js
console.log(readable._readableState);
```

这会记录一个具有 `buffer`、`length`、`highWaterMark`、`flowing`、`ended`、`endEmitted`、`reading`、`destroyed` 等属性的对象。如果流表现异常，检查这个状态可以揭示出问题所在。

你还可以通过设置 `NODE_DEBUG` 环境变量来为流启用调试日志：

```js
NODE_DEBUG=stream node your-script.js
```

这会将详细的内部流事件记录到标准错误输出（stderr），显示何时调用 `_read()`，何时推送数据，何时发出事件，等等。它很详细，但对于理解操作的确切顺序非常有用。

## 内存影响和选择 highWaterMark

我们已经多次提到 `highWaterMark`，但让我们讨论如何选择一个好的值。对于大多数用例，默认的 64KB 是一个合理的折衷方案，但并不是所有场景都最优。

如果你正在流式传输大型文件，其中源和消费者的速度大致平衡，增加 `highWaterMark` 可以提高吞吐量。更大的缓冲区意味着从源读取的系统调用更少。例如，如果你正在从快速的 SSD 读取文件，将 `highWaterMark` 增加到 128KB 或 256KB 可以减少重复 `fs.read()` 调用的开销。

```js
const readable = fs.createReadStream("large-file.bin", {
 highWaterMark: 128 * 1024, // 128KB
});
```

然而，更大的缓冲区意味着更多的内存使用。如果您正在同时处理数千个流（例如，在高流量的 Web 服务器中，每个请求都从文件中读取），所有这些缓冲区的累积内存使用量可能会非常显著。在这种情况下，您可能希望降低 `highWaterMark` 以减少每个流的内存占用。

```js
const readable = fs.createReadStream("file.txt", {
 highWaterMark: 4 * 1024, // 4KB
});
```

同时也要考虑延迟问题。如果您正在流式传输实时数据（例如，实时视频流），较小的 `highWaterMark` 意味着更低的延迟。消费者可以更快地获取数据，因为流不需要等待填充大缓冲区后再推送数据。相反，如果延迟不是问题，而吞吐量至关重要，较大的 `highWaterMark` 可以减少开销。

在 `objectMode` 中，权衡类似，但单位是对象而不是字节。如果每个对象都很小，16 的 `highWaterMark` 可能太低，导致频繁的 `_read()` 调用并降低吞吐量。如果每个对象都很大，16 可能太高，消耗过多的内存。

没有一种适合所有情况的答案。请分析您的应用程序。使用不同的 `highWaterMark` 值来测量内存使用量和吞吐量。默认值通常就足够了，但对于性能关键的应用程序，调整设置可能会有所帮助。

## `Readable.from()` 和异步迭代器

在我们结束这一章之前，让我们回顾一下 `Readable.from()` 并了解为什么它如此强大。这个实用工具弥合了异步迭代器（生成器、数组、异步生成器）和可读流之间的差距，让您能够在不手动实现 `_read()` 的情况下利用流 API。

假设您有一个异步生成器，它从 API 中获取分页数据：

```js
async function* fetchPages(url) {
 let page = 1;
 while (true) {
 const response = await fetch(`${url}?page=${page}`);
 const data = await response.json();
 if (data.items.length === 0) break;
 for (const item of data.items) {
 yield item;
 }
 page++;
 }
}
```

这个生成器获取项目页面并产生每个项目。您可以将它转换成一个可读流：

```js
const stream = Readable.from(fetchPages("https://api.example.com/items"));
 stream.pipe(someWritable);
```

`Readable.from()` 方法处理所有繁重的工作。它调用生成器的 `next()` 方法，等待承诺解决，将产生的值推入流中，并重复。如果生成器抛出错误，流会发出 `error` 事件。如果生成器完成，流会推送 `null` 以表示结束。

这种模式具有极高的可组合性。如果您有一个返回异步迭代器的函数，您只需一行代码就可以将其连接到流生态系统。您将获得流的所有好处——背压、基于事件的消费、连接到可写流、与 `pipeline()` 兼容——而无需编写任何特定于流的代码。

您也可以传递常规迭代器，而不仅仅是异步迭代器：

```js
const stream = Readable.from([1, 2, 3, 4, 5]);
 for await (const num of stream) {
 console.log(num);
}
```

这将从数组创建一个可读流。数组的每个元素都成为流中的一个块。这是一种简单地将同步数据转换为流以进行测试或与基于流的 API 集成的方法。

这里的教训是，可读流不仅仅是关于读取文件或套接字。它们是产生值序列的一般抽象，无论这些值来自 I/O、计算还是数据结构的迭代。`Readable.from()`使这种抽象对任何产生可迭代或异步可迭代的代码都变得可访问，降低了在代码库中采用流的门槛。

## 总结

**啊哈……终于！** 我们已经覆盖了很多内容。让我们快速回顾一下我们所学的内容。

可读流是一个数据的生产者。它从底层源（文件、套接字、生成器、数据库查询）中拉取数据，并将这些数据提供给消费者。流维护一个内部缓冲区——一个数据块数组——位于源和消费者之间，以平滑速率不匹配并提供反向压力。

流在两种模式之一中操作：暂停或流动。在暂停模式下，必须显式使用`read()`来拉取数据。在流动模式下，数据通过`data`事件自动推送。你通过事件监听器和`pause()`、`resume()`、`pipe()`等方法调用控制模式。

`highWaterMark`选项控制缓冲区大小的阈值。当缓冲区长度超过`highWaterMark`时，流停止调用`_read()`，对源应用反向压力。当缓冲区低于`highWaterMark`时，流再次调用`_read()`，请求更多数据。

要实现自定义可读流，你扩展`Readable`类并实现`_read()`。在`_read()`内部，你从你的源获取数据并调用`push(chunk)`将其添加到缓冲区。当没有更多数据时，你调用`push(null)`来表示结束。如果发生错误，你调用`destroy(err)`。

要消费可读流，你有几种选择：基于事件的消费使用`data`和`end`监听器，异步迭代使用`for await...of`，在暂停模式下显式调用`read()`，使用`pipe()`进行管道传输，或者使用`stream.pipeline()`进行健壮的管道。每种模式在简单性、反向压力处理和控制方面都有权衡。

可读流通过事件来发出状态变化：`data`用于数据块，`end`用于完成，`error`用于错误，`readable`用于数据可用性，`close`用于资源清理。你的代码对这些事件做出反应，以处理数据和处理边缘情况。

反向压力对于有限的内存使用至关重要。如果你的消费者比生产者慢，你必须使用一种强制自动应用反向压力的消费模式（异步迭代、`pipeline()`），或者手动实现它（暂停/恢复）。

对象模式将流的语义从基于字节更改为基于对象，允许你推送任意 JavaScript 值，并将`highWaterMark`视为对象计数而不是字节计数。

这种心智模型——源（source）、缓冲区（buffer）、`highWaterMark`、模式（modes）、事件（events）、背压（backpressure）——是处理 Node.js 中所有流的基础，而不仅仅是可读流（Readable streams）。可写流（Writable）、转换流（Transform）、和双工流（Duplex）都是基于这些相同的概念，并添加了它们自己的特定行为和契约。深入理解可读流意味着你已经掌握了流式处理范式的一半。另一半——写入数据、转换数据、和组合流——自然地建立在你在本部分学到的内容之上。
