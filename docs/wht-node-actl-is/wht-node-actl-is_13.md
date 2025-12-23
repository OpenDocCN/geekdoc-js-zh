# 转换与双工流

> 原文：[`www.thenodebook.com/streams/transform-streams`](https://www.thenodebook.com/streams/transform-streams)

`Readable`流产生数据。`Writable`流消费它。但有时你需要两者，或者你需要一个两个方向不相关的流，或者你需要输入和输出之间的转换。

`双工`流同时具有可读性和可写性，**两个独立的部分**并行操作。`转换`流是`双工`流的一个特殊版本，其中可写部分通过转换函数将数据传递到可读部分。这两种类型之间的区别会影响你构建数据处理管道的方式。

这两种流类型的工作方式不同。`双工`流具有独立的部分。`转换`流，在应用代码中更为常见，通过转换函数将可写输入连接到可读输出。我们将实现几个自定义的`转换`流来展示模式，然后讨论何时选择`双工`与`转换`。

## 双工流

`双工`流是基础。它是使`转换`流有意义的基石。`双工`流同时具有可读性和可写性。你可以在同一个对象上调用`read()`和`write()`。你可以附加`'data'`监听器并将块传递给`write()`。流具有`Readable`和`Writable`的所有属性和事件。

关键细节：**可读部分**和**可写部分**是**独立的**。你写入`双工`流的数据不会自动出现在可读部分。这两部分是独立的通道，恰好存在于同一个对象上。想象一下电话线——你可以对着它说话并通过它来听，但你说的内容不会回声给你。两个方向是独立的。

这种独立性存在是因为`双工`流模拟了双向通信通道。典型的例子是 TCP 套接字。当你有一个套接字连接时，你可以通过写入套接字向远程端点发送数据，也可以通过从它读取来接收数据。你发送的数据不是你接收的数据——它们是两个在相同连接上同时发生的独立通信流。

在类级别上，`双工`流具有特定的结构。`stream.Duplex`类扩展了`Readable`，同时也实现了`Writable`接口。内部，它为可读部分（`_readableState`）和可写部分（`_writableState`）维护了独立的状态。当你实现自定义的`双工`流时，你需要提供`_read()`和`_write()`方法。

最小化双工流实现：

```js
import { Duplex } from "stream";
 class MinimalDuplex extends Duplex {
 _read(size) {
 // produce data for readable side
 this.push("readable data");
 this.push(null);
 }
 _write(chunk, encoding, callback) {
 // consume data on writable side
 console.log("Received:", chunk.toString());
 callback();
 }
}
```

当可读部分需要数据时，会调用`_read()`方法。当有数据写入可写部分时，会调用`_write()`方法。这两个方法不相互交互。它们是完全独立的。

使用此流：

```js
const duplex = new MinimalDuplex();
 duplex.on("data", (chunk) => {
 console.log("Read:", chunk.toString());
});
 duplex.write("written data");
duplex.end();
```

当你运行这个程序时，你会从 `_write()` 端看到 "Received: written data"，从 `_read()` 端看到 "Read: readable data"。它们不是连接的。你并不是将 "written data" 转换为 "readable data"——它们是两个独立的流程。

`allowHalfOpen` 选项是 `Duplex` 特有的，它改变了流处理结束的方式。当你创建一个 `Duplex` 流时，你可以设置 `allowHalfOpen: false` 来改变当一方结束时会发生什么。

默认情况下，`allowHalfOpen` 是 `true`。这意味着可读端可以在可写端仍然打开时结束，反之亦然。你可以在可写端完成写入并调用 `end()`，但可读端会继续产生数据。或者可读端可以 `push(null)` 来表示 EOF，但你仍然可以向可写端写入。

网络套接字就是这样工作的。当一个 TCP 连接处于 **半关闭** 状态时，一个端点已经完成发送但仍然可以接收。只有当双方都完成发送后，连接才会完全关闭。

如果你设置 `allowHalfOpen: false`，流将强制执行当任何一方结束时，另一方也必须结束。如果可读端推送 `null`，可写端会自动结束。如果你在可写端调用 `end()`，可读端会自动推送 `null`。

```js
const duplex = new Duplex({
 allowHalfOpen: false,
 read() {
 // readable implementation
 },
 write(chunk, encoding, callback) {
 // writable implementation
 callback();
 },
});
```

当 `allowHalfOpen: false` 时，调用 `duplex.end()` 会导致可读端立即结束。当你模拟不支持半开放状态的东西时，可以使用这个选项，比如请求-响应协议，其中流应该在任一方向完成时完全关闭。

原始 `Duplex` 流的实际应用案例大多与 I/O 原语有关。Node 网络模块中的 `net.Socket` 类是一个 `Duplex` 流。当你创建一个 TCP 套接字时，你会得到一个 `Duplex`。可写端通过网络发送数据。可读端从网络接收数据。由于你正在与远程端点通信，这两端是独立的——你发送的内容并不等于你接收的内容。

另一个例子是子进程的 `stdin` 和 `stdout`。当你启动一个子进程时，它的 `stdin` 是可写的（你向进程发送数据），而它的 `stdout` 是可读的（你从进程接收数据）。这些被建模为一个 `Duplex` 流，其中两端与外部进程通信，而不是彼此之间。

应用代码很少从头开始实现 `Duplex` 流。`Transform` 流在数据转换中更为常见。但首先，一个稍微更现实的 `Duplex` 示例：

这个 `Duplex` 流维护一个内存缓冲区。写入的数据存储在一个内部数组中，从它读取的数据来自该数组：

```js
class BufferedDuplex extends Duplex {
 constructor(options) {
 super(options);
 this.buffer = [];
 }
 _write(chunk, encoding, callback) {
 this.buffer.push(chunk);
 callback();
 }
 _read(size) {
 if (this.buffer.length > 0) {
 this.push(this.buffer.shift());
 }
 }
}
```

现在，两端通过共享状态（`this.buffer` 数组）进行交互。当你写入时，数据块会被添加到缓冲区。当可读端需要数据时，数据块会被从缓冲区中取出。这是使用 `Duplex` 流实现的基本队列实现。

尽管存在共享状态，`_read()`和`_write()`方法并不互相调用。它们只是访问相同的数据结构。流的内部机制处理在可读侧需要数据时调用`_read()`，以及在可写侧有写入时调用`_write()`。

使用`Duplex`来实现队列或缓冲区是可行的，但这不是主要用例。通常，如果你正在构建在管道中转换或处理数据的程序，你想要一个`Transform`流，而不是`Duplex`。

关于`Duplex`流的一个额外细节：由于两侧独立，错误处理方式不同。因为`Duplex`有两个独立的侧面，一侧的错误不会自动传播到另一侧。如果在`_write()`中发生错误，流会发出一个`'error'`事件，但除非你明确地销毁它，否则可读侧会继续运行。同样，`_read()`中的错误也不会停止可写侧。

然而，当你对一个`Duplex`流调用`destroy()`时，两侧都会被销毁。这是正确的行为 - 销毁流意味着整个资源正在关闭，而不仅仅是单向。

```js
duplex.destroy(new Error("Fatal error"));
// Both readable and writable sides are now destroyed
```

当你处理清理或取消时，这很重要。如果你使用`Duplex`来模拟网络连接，并且连接断开，你将销毁流，这将关闭发送和接收。

## Transform Streams

`Transform`流是大多数开发者在构建数据处理程序时首先想到的。`Transform`流是一种特殊的`Duplex`流，其中可写输入通过一个转换函数连接到可读输出。数据从一侧流入，经过处理，然后从另一侧流出。

与两侧独立的原始`Duplex`流不同，`Transform`流在它们之间创建了一种**因果关系**。你写入可写侧的内容会直接影响可读侧输出的内容。你不仅仅是实现两个独立的通道 - 你实现的是一个函数，该函数接收输入块并生成输出块。

在 Node.js 的标准库中，`Transform`流的常见例子包括压缩和加密。`zlib.createGzip()`函数返回一个`Transform`流。你将未压缩的数据写入它，然后从它那里读取压缩数据。`crypto.createCipheriv()`函数返回一个`Transform`流。你将明文写入它，然后从它那里读取密文。转换在流内部进行。

`Transform`类在几个关键方面与`Duplex`不同。`Transform`扩展了`Duplex`，因此它具有`Duplex`的所有属性和方法。但不是实现`_read()`和`_write()`，而是实现一个不同的方法：`_transform()`。

`_transform()`方法具有以下签名：

```js
_transform(chunk, encoding, callback)
```

它接收来自可写侧的一个**块**，对其进行处理，并将零个或多个输出块推送到可读侧。当它完成处理时，它会调用**回调**来表示它已准备好接收下一个块。

一个简单的 `Transform`，将文本转换为大写：

```js
import { Transform } from "stream";
 class UppercaseTransform extends Transform {
 _transform(chunk, encoding, callback) {
 const upper = chunk.toString().toUpperCase();
 this.push(upper);
 callback();
 }
}
```

`_transform()` 方法接收一个数据块（默认情况下是一个 `Buffer`），将其转换为字符串，转换为大写，使用 `this.push()` 将结果推送到可读的一侧，然后调用回调以指示转换已完成。

使用这个流：

```js
const upper = new UppercaseTransform();
 upper.on("data", (chunk) => {
 console.log(chunk.toString());
});
 upper.write("hello");
upper.write("world");
upper.end();
```

输出："HELLO" 和 "WORLD"。你写入的每个数据块都会被转换，并在可读的一侧出现。

`Transform` 的 `_read()` 方法已经为你实现，与 `Duplex` 不同。你不需要重写它。`Transform` 基类处理从由你的 `_transform()` 方法填充的内部缓冲区中拉取数据。同样，`Transform` 的 `_write()` 方法也是实现为调用你的 `_transform()` 方法。你只需实现 **转换逻辑** - 流的管道处理由基类处理。

这使得 `Transform` 流比原始 `Duplex` 流更容易实现。你专注于“我对这个数据块做什么”，而不是“我如何管理两个独立的一侧”。

`_transform()` 中的回调参数执行两个操作。它表示你已完成当前数据块的处理，并允许你报告错误。

如果转换过程中发生错误，你将其传递给回调：

```js
_transform(chunk, encoding, callback) {
 try {
 const result = JSON.parse(chunk.toString());
 this.push(JSON.stringify(result));
 callback();
 } catch (err) {
 callback(err);
 }
}
```

如果你将错误传递给回调，流将发出 `'error'` 事件并停止处理。任何缓冲的数据都将被丢弃，流进入 **错误状态**。

你也可以在单个 `_transform()` 调用中使用 `this.push()` 多次。这被称为 **一对一转换**。对于每个输入数据块，你产生多个输出数据块。

这个 `Transform` 将输入分割成单独的行：

```js
class LineSplitter extends Transform {
 _transform(chunk, encoding, callback) {
 const lines = chunk.toString().split("\n");
 for (const line of lines) {
 if (line.length > 0) {
 this.push(line + "\n");
 }
 }
 callback();
 }
}
```

如果你写入 `"hello\nworld\n"`，转换器将推送两个数据块：`"hello\n"` 和 `"world\n"`。一个输入数据块变成了多个输出数据块。

你也可以不推送任何内容。如果你的转换决定丢弃一个数据块（过滤掉它），你只需调用回调而不推送：

```js
_transform(chunk, encoding, callback) {
 const text = chunk.toString();
 if (!text.startsWith("#")) {
 this.push(chunk);
 }
 callback();
}
```

这个转换器过滤掉以 `"#"` 开头的数据块。一些数据块通过，其他则被丢弃。

那么 **多对一转换** 呢？在这种情况下，你需要累积多个输入数据块才能产生输出。这在解析可能跨越数据块边界的结构化数据时很常见。你使用实例状态来缓冲不完整的数据。

这个 `Transform` 累积数据块，直到它看到分隔符，然后发出累积的数据：

```js
class DelimiterParser extends Transform {
 constructor(delimiter, options) {
 super(options);
 this.delimiter = delimiter;
 this.buffer = "";
 }
 _transform(chunk, encoding, callback) {
 this.buffer += chunk.toString();
 const parts = this.buffer.split(this.delimiter);
 this.buffer = parts.pop(); // last part is incomplete
 for (const part of parts) {
 this.push(part);
 }
 callback();
 }
}
```

这个转换器维护一个 `this.buffer`，用于累积传入的数据。每次调用 `_transform()` 时，它将新的数据块追加到缓冲区，根据分隔符进行分割，并推送完整的部分。最后一个部分（可能是不完整的）将保留在缓冲区中，以便下一次调用。

这是在 `Transform` 流中的一个基本模式：在 `_transform()` 调用之间 **维护状态**，以处理跨越多个数据块的数据结构。这是 **有状态转换**。

上述实现有一个问题。当流结束时，缓冲区中的任何剩余数据都会丢失。流结束而没有发出那个最后的未完成块。这就是`_flush()`的作用。

## `_flush()`方法

`Transform`流有一个你可以实现的方法：`_flush()`。这个方法在所有输入块都处理完毕后（在可写侧调用`end()`之后）被调用，但在可读侧推送`null`以发出 EOF 信号之前。这是你发出任何剩余数据的机会。

`_flush()`的签名是：

```js
_flush(callback)
```

它只接收一个回调，没有块。你可以调用`this.push()`来发出最终数据，然后调用回调来表示刷新完成。

添加了`_flush()`的定界符解析器：

```js
class DelimiterParser extends Transform {
 constructor(delimiter, options) {
 super(options);
 this.delimiter = delimiter;
 this.buffer = "";
 }
 _transform(chunk, encoding, callback) {
 this.buffer += chunk.toString();
 const parts = this.buffer.split(this.delimiter);
 this.buffer = parts.pop();
 for (const part of parts) {
 this.push(part);
 }
 callback();
 }
 _flush(callback) {
 if (this.buffer.length > 0) {
 this.push(this.buffer);
 }
 callback();
 }
}
```

现在当流结束时，`_flush()`被调用。如果缓冲区中还有剩余数据，它将被作为最后一个块推送。然后调用回调，流将推送`null`以在可读侧发出 EOF 信号。

没有实现`_flush()`，不以分隔符结束的数据会丢失。有了`_flush()`，它会被作为最后一个块发出。解析器、解码器和任何累积状态的`Transform`都需要这个。

`_flush()`回调的工作方式与`_transform()`回调相同。如果发生错误，你将其传递给回调：

```js
_flush(callback) {
 if (this.buffer.length > 0) {
 try {
 const parsed = this.parseBuffer(this.buffer);
 this.push(parsed);
 callback();
 } catch (err) {
 callback(err);
 }
 } else {
 callback();
 }
}
```

如果你将错误传递给回调，流将发出一个`'error'`事件而不是干净地结束。

关于`_flush()`的一个更多细节：它是**可选的**。如果你不实现它，流将直接结束而没有最后的处理步骤。不累积状态的转换（如大写转换）不需要这个。每个块都是独立的，所以当流结束时没有要刷新的内容。

但是对于任何跨块缓冲的转换（解析器、解码器、聚合器）- 你必须实现`_flush()`以避免数据丢失。

一个更完整的例子：解析**NDJSON**（换行分隔的 JSON），其中每一行都是一个单独的 JSON 文档。

```js
class NDJSONParser extends Transform {
 constructor(options) {
 super({ ...options, objectMode: true });
 this.buffer = "";
 }
 _transform(chunk, encoding, callback) {
 this.buffer += chunk.toString();
 const lines = this.buffer.split("\n");
 this.buffer = lines.pop();
 for (const line of lines) {
 if (line.trim().length > 0) {
 try {
 const obj = JSON.parse(line);
 this.push(obj);
 } catch (err) {
 return callback(err);
 }
 }
 }
 callback();
 }
 _flush(callback) {
 if (this.buffer.trim().length > 0) {
 try {
 const obj = JSON.parse(this.buffer);
 this.push(obj);
 callback();
 } catch (err) {
 callback(err);
 }
 } else {
 callback();
 }
 }
}
```

这个转换在`objectMode`模式下运行，这意味着它推送 JavaScript 对象而不是缓冲区。每一行都被解析为 JSON，然后生成的对象被推送到可读的一侧。如果一块的末尾有一行不完整，它将被缓冲，直到下一块到达。当流结束时，`_flush()`解析任何剩余的缓冲行。

如果`JSON.parse()`抛出错误，我们将错误传递给回调。这停止了流并发出一个错误事件。我们使用`return callback(err)`来提前退出 - 我们不希望在错误后继续处理。

这种模式（跨块缓冲，按分隔符分割，解析完整单元，刷新剩余数据）出现在大多数结构化数据的`Transform`流中。

## 实现自定义转换流

现在你已经理解了机制，我们将实现几个`Transform`流。这些包括**过滤**、**映射**、**分割**、**连接**和**状态解析**。

**过滤转换**会通过满足条件的块，并丢弃不满足条件的块。这个转换过滤掉了空行：

```js
class NonEmptyLines extends Transform {
 _transform(chunk, encoding, callback) {
 const text = chunk.toString();
 if (text.trim().length > 0) {
 this.push(chunk);
 }
 callback();
 }
}
```

简单。如果块（在去除空白后）有内容，则推入。否则，跳过。回调总是被调用以表示完成，即使我们没有推入。

**映射转换**将每个输入块转换为不同的输出块，通常在`objectMode`中。这个转换接受 JSON 对象并提取特定字段：

```js
class FieldExtractor extends Transform {
 constructor(fields, options) {
 super({ ...options, objectMode: true });
 this.fields = fields;
 }
 _transform(obj, encoding, callback) {
 const extracted = {};
 for (const field of this.fields) {
 if (obj[field] !== undefined) {
 extracted[field] = obj[field];
 }
 }
 this.push(extracted);
 callback();
 }
}
```

每个输入对象都映射到一个新对象，只包含指定的字段。一个对象输入，一个对象输出。这是一个**一对一转换**。

**分割转换**将输入拆分成更小的部分。我们看到了行分割器，但这里有一个字节级别的分割器，它将流分割成固定大小的块：

```js
class ChunkSplitter extends Transform {
 constructor(chunkSize, options) {
 super(options);
 this.chunkSize = chunkSize;
 this.buffer = Buffer.alloc(0);
 }
 _transform(chunk, encoding, callback) {
 this.buffer = Buffer.concat([this.buffer, chunk]);
 while (this.buffer.length >= this.chunkSize) {
 const piece = this.buffer.slice(0, this.chunkSize);
 this.buffer = this.buffer.slice(this.chunkSize);
 this.push(piece);
 }
 callback();
 }
 _flush(callback) {
 if (this.buffer.length > 0) {
 this.push(this.buffer);
 }
 callback();
 }
}
```

这个转换在缓冲区中累积传入的数据。当缓冲区达到`chunkSize`时，它切下一个块并将其推入。循环继续，直到缓冲区中剩余的数据少于`chunkSize`。当流结束时，`_flush()`将任何剩余的数据作为最后的部分块发出。

**连接转换**将多个输入块合并成一个输出块。这个转换将对象累积到数组中，并在达到一定数量时发出数组：

```js
class BatchAccumulator extends Transform {
 constructor(batchSize, options) {
 super({ ...options, objectMode: true });
 this.batchSize = batchSize;
 this.batch = [];
 }
 _transform(obj, encoding, callback) {
 this.batch.push(obj);
 if (this.batch.length >= this.batchSize) {
 this.push(this.batch);
 this.batch = [];
 }
 callback();
 }
 _flush(callback) {
 if (this.batch.length > 0) {
 this.push(this.batch);
 }
 callback();
 }
}
```

这是一个**多对一转换**。它累积`batchSize`个对象，然后推入数组。如果流以部分批次结束，`_flush()`将发出它。

**有状态解析转换**在块之间维护状态以解析结构化数据。我们看到了分隔符解析，但这里有一个更复杂的例子：一个**长度前缀的二进制消息解析器**。

在长度前缀协议中，每个消息都以一个 4 字节的长度字段（一个`uint32`）开头，指示后面有多少字节。为了解析它，我们需要读取长度，然后读取那么多字节，然后重复。

```js
class LengthPrefixedParser extends Transform {
 constructor(options) {
 super({ ...options, objectMode: true });
 this.buffer = Buffer.alloc(0);
 this.expectedLength = null;
 }
 _transform(chunk, encoding, callback) {
 this.buffer = Buffer.concat([this.buffer, chunk]);
 while (this.buffer.length >= 4) {
 if (this.expectedLength === null) {
 this.expectedLength = this.buffer.readUInt32BE(0);
 this.buffer = this.buffer.slice(4);
 }
 if (this.buffer.length >= this.expectedLength) {
 const message = this.buffer.slice(0, this.expectedLength);
 this.buffer = this.buffer.slice(this.expectedLength);
 this.expectedLength = null;
 this.push(message);
 } else {
 break;
 }
 }
 callback();
 }
}
```

这个转换使用一个**状态机**。`expectedLength`变量跟踪我们是否正在等待读取长度头或等待读取消息体。循环从缓冲区中读取尽可能多的完整消息，并将每个消息推入，然后调用回调。

这里没有`_flush()`。如果流以不完整的数据结束（部分长度头或部分消息体），则该数据将丢失。这是否正确取决于你的协议。一些协议将 EOF 处的部分数据视为错误。其他协议在`_flush()`中发出一个最终的不完整消息或发出错误。

你实现的大多数转换将是这些模式的变体。

## **分块边界问题**

跨块边界的**数据结构**在几乎每一个`Transform`实现中都会引起问题。这是流代码中许多微妙错误的来源。

当你处理字节流或文本流时，你接收到的块是**任意的**。流不知道也不关心你的数据结构。如果你正在解析由换行符分隔的 JSON 对象，换行符可能出现在块的中间，或者它可能正好落在块边界上，或者一个 JSON 对象可能被分割在两个块之间。

你不能假设每个块都是一个完整的单元。你必须处理**部分数据**。

我们在分隔符解析器和长度前缀解析器中看到了这一点。**缓冲**解决了这个问题。您维护一个内部缓冲区（通常是字符串或`Buffer`），累积传入的数据。您从缓冲区中处理完整的单元，并留下不完整的单元供下一次调用。

抽象形式的模式：

```js
class Parser extends Transform {
 constructor(options) {
 super(options);
 this.buffer = "";
 }
 _transform(chunk, encoding, callback) {
 this.buffer += chunk.toString();
 let unit;
 while ((unit = this.extractCompleteUnit(this.buffer))) {
 this.buffer = unit.remainder;
 this.push(unit.data);
 }
 callback();
 }
 _flush(callback) {
 if (this.buffer.length > 0) {
 // handle remaining data
 }
 callback();
 }
 extractCompleteUnit(buffer) {
 // return { data, remainder } or null
 }
}
```

`extractCompleteUnit()`方法尝试从缓冲区中解析一个完整的单元。如果成功，它返回解析的数据和剩余的缓冲区。如果没有足够的数据来解析一个完整的单元，它返回 null。循环继续提取单元，直到缓冲区为空或不完整。

这个模式正确地处理了任意块边界。块在哪里分割并不重要——解析器累积数据直到它有一个完整的单元，解析它，然后继续。

具体示例：从流中解析 CSV 行。CSV 文件是由换行符分隔的行，每行由逗号分隔的字段。一行可能被分割在块之间，如果字段被引号包围，它可能包含换行符。

一个简化的 CSV 解析器（不处理引号字段）：

```js
class SimpleCSVParser extends Transform {
 constructor(options) {
 super({ ...options, objectMode: true });
 this.buffer = "";
 }
 _transform(chunk, encoding, callback) {
 this.buffer += chunk.toString();
 const lines = this.buffer.split("\n");
 this.buffer = lines.pop();
 for (const line of lines) {
 const fields = line.split(",");
 this.push(fields);
 }
 callback();
 }
 _flush(callback) {
 if (this.buffer.length > 0) {
 const fields = this.buffer.split(",");
 this.push(fields);
 }
 callback();
 }
}
```

这正确地处理了行边界。如果一个行被分割在块之间，部分行将被缓冲，直到后续块中到达换行符。

但这并不能处理引号字段。如果一个字段是`"hello\nworld"`，引号内的换行符不应该分割行。正确处理这需要更复杂的有限状态机，该状态机跟踪我们是否处于引号内。

正确的`Transform`实现必须考虑到数据结构可能跨越块的可能性。缓冲区和状态机是您用来处理这些的工具。

## 在`Transform`中的推行为和背压

在`Transform`中调用`this.push()`比它看起来更微妙，因为`push`是可读端的接口，并且它尊重**背压**。

当你调用`this.push(chunk)`时，块被添加到可读端的内部缓冲区。如果缓冲区低于其`highWaterMark`，`push`返回`true`。如果缓冲区达到或超过`highWaterMark`，`push`返回`false`，表示背压。

你可以在`_transform()`中检查这个返回值：

```js
_transform(chunk, encoding, callback) {
 const transformed = this.transformData(chunk);
 const canContinue = this.push(transformed);
 if (!canContinue) {
 // readable side is full, but we have to process this chunk
 }
 callback();
}
```

`_transform()`中`push`的返回值通常不会影响你的逻辑。你仍然需要调用回调。`Transform`流通过在你不再调用`_transform()`之前不调用它来为你处理背压。你不需要自己实现暂停/恢复逻辑。

这与实现一个`Readable`流不同，在`Readable`流中，你检查`push`的返回值，如果它返回`false`，则停止调用`_read()`。在`Transform`中，基类处理这一点。你只需实现转换逻辑。

然而，如果你在一个单次`_transform()`调用中推送多个块（一对一到多一的转换），你可能想要检查`push`的返回值，并在背压信号时停止推送：

```js
_transform(chunk, encoding, callback) {
 const parts = this.splitIntoParts(chunk);
 for (const part of parts) {
 const canContinue = this.push(part);
 if (!canContinue) {
 // readable side is full, buffer the rest
 this.bufferedParts = parts.slice(parts.indexOf(part) + 1);
 break;
 }
 }
 callback();
}
```

但这增加了复杂性。大多数时候，你只是推送所有输出块，让流的缓冲处理背压。可读方面的缓冲区会增长，直到达到 `highWaterMark`，此时流停止调用 `_transform()`，直到缓冲区排空。

这种自动背压处理使得 `Transform` 流比原始 `Duplex` 流更容易使用。你不需要手动协调两边 - 基类为你处理。

## PassThrough

有一个内置的 `Transform` 流什么都不做：`stream.PassThrough`。这是一个 `Transform`，其中 `_transform()` 只是原样推送输入块。

```js
import { PassThrough } from "stream";
 const passthrough = new PassThrough();
```

用例：观察或拦截数据而不修改它。

一个用例是添加事件监听器。你可以在管道中插入一个 `PassThrough` 并将其 `'data'` 监听器附加到它，以观察通过而不影响管道的数据：

```js
import { pipeline } from "stream/promises";
 const passthrough = new PassThrough();
 passthrough.on("data", (chunk) => {
 console.log("Passing through:", chunk.length, "bytes");
});
 await pipeline(source, passthrough, destination);
```

另一个用例是实现一个 **tee** 或 **broadcast** 模式，其中你将流分割到多个目的地。你可以将流管道到多个 `PassThrough`，每个 `PassThrough` 可以被管道到不同的目的地。

`PassThrough` 也在测试中很有用。你可以创建一个 `PassThrough`，向其中写入测试数据，然后从中读取以验证你的流处理逻辑是否正确工作。

实现 `PassThrough`：

```js
class MyPassThrough extends Transform {
 _transform(chunk, encoding, callback) {
 callback(null, chunk);
 }
}
```

`callback(null, chunk)` 是推送块然后调用回调的简写。它等价于：

```js
this.push(chunk);
callback();
```

当你想要每个输入块正好推送一个块时，将块传递给回调的这种模式很常见。

## Transform vs Duplex - 何时使用每个

我们已经介绍了 `Duplex` 和 `Transform` 流。它们之间的选择对 API 设计很重要。

`Transform` 流适合数据管道，其中输入块成为输出块。输出取决于输入。压缩、加密、解析、格式化、过滤和映射都是这样工作的。

`Duplex` 流模拟双向通信通道，其中可读和可写方面是独立的。网络套接字、IPC 通道、代理连接和双向消息传递都是这样工作的。

你写入的内容会影响你读取的内容吗？如果是，使用 `Transform`。如果不是，使用 `Duplex`。

应用层代码主要使用 `Transform` 流。`Duplex` 流出现在系统层：网络和 IPC 模块，它们模拟通道而不是转换。

具体示例：

一个压缩数据的 **Transform**：

```js
import { createGzip } from "zlib";
 const gzip = createGzip();
input.pipe(gzip).pipe(output);
```

你写入 `gzip`（未压缩数据）的内容直接决定了你从中读取的内容（压缩数据）。这是一个转换。

代表 TCP 套接字的 **Duplex**：

```js
import { connect } from "net";
 const socket = connect(3000, "localhost");
socket.write("request");
socket.on("data", (chunk) => {
 console.log("response:", chunk);
});
```

你写入 `socket`（你的请求）的内容不会产生你从 `socket`（服务器的响应）读取的数据。它们是独立的。这是一个通道。

有一种微妙的情况，你可能需要实现一个`Duplex`而不是`Transform`：当你需要模拟具有独立输入和输出但两边共享状态的东西时。例如，一个使用相同的加密密钥加密输出数据和解密输入数据的流。两边是独立的（加密 A 不会产生解密 B），但它们共享配置。

实际上，你可能将这实现为两个独立的`Transform`流（一个用于加密，一个用于解密）而不是一个单一的`Duplex`，因为这更干净且更易于组合。但技术上这是一个有效的`Duplex`用例。

常规做法：如果你正在构建一个自然地适合与其他转换一起组成管道的东西，就将其做成一个`Transform`。如果你正在构建位于系统边缘、与外部实体通信的东西，就将其做成一个`Duplex`。

## 真实世界的转换示例

一些你可能在生产中实际使用的`Transform`流：

**1) JSON 行字符串化器**

这个转换接受 JavaScript 对象并输出换行符分隔的 JSON：

```js
class JSONLineStringifier extends Transform {
 constructor(options) {
 super({ ...options, writableObjectMode: true });
 }
 _transform(obj, encoding, callback) {
 try {
 const json = JSON.stringify(obj);
 this.push(json + "\n");
 callback();
 } catch (err) {
 callback(err);
 }
 }
}
```

这里设置的`writableObjectMode: true`：可写端接受对象，但可读端发出字符串（或缓冲区）。你可以混合模式——可写端在`objectMode`中，可读端在字节模式中，反之亦然。

**2) 行数计数器**

这个转换计算行数并在结束时发出一个总结对象：

```js
class LineCounter extends Transform {
 constructor(options) {
 super({ ...options, readableObjectMode: true });
 this.lineCount = 0;
 this.byteCount = 0;
 }
 _transform(chunk, encoding, callback) {
 this.byteCount += chunk.length;
 const lines = chunk.toString().split("\n").length - 1;
 this.lineCount += lines;
 callback();
 }
 _flush(callback) {
 this.push({
 lines: this.lineCount,
 bytes: this.byteCount,
 });
 callback();
 }
}
```

这个转换在`_transform()`期间不推送任何内容，只是累积统计数据。在`_flush()`中，它推送一个单独的总结对象。这是一个有效的模式——转换不需要为每个输入块产生输出。

**3) 速率限制器**

这个转换延迟块以强制最大吞吐量：

```js
class RateLimiter extends Transform {
 constructor(bytesPerSecond, options) {
 super(options);
 this.bytesPerSecond = bytesPerSecond;
 this.tokens = bytesPerSecond;
 this.lastRefill = Date.now();
 }
 _transform(chunk, encoding, callback) {
 this._refillTokens();
 const wait =
 Math.max(0, chunk.length - this.tokens) / this.bytesPerSecond;
 setTimeout(() => {
 this.tokens = Math.max(0, this.tokens - chunk.length);
 this.push(chunk);
 callback();
 }, wait * 1000);
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

这个转换使用令牌桶来限制吞吐量。如果当前块没有足够的令牌，它将延迟回调，直到有足够的时间过去。这是一个用于限制数据流以匹配下游容量的有用模式。

**4) 去重器**

在 objectMode 中，这个转换会根据键移除重复的对象：

```js
class Deduplicator extends Transform {
 constructor(keyField, options) {
 super({ ...options, objectMode: true });
 this.keyField = keyField;
 this.seen = new Set();
 }
 _transform(obj, encoding, callback) {
 const key = obj[this.keyField];
 if (!this.seen.has(key)) {
 this.seen.add(key);
 this.push(obj);
 }
 callback();
 }
}
```

这维护了一个它已看到的键的集合。如果一个对象的键是新的，它就会被推送。否则，它会被丢弃。这是一个有状态的过滤转换。

这些示例展示了转换流的通用性。它们可以聚合、过滤、格式化、节流、去重等。任何接受块流并产生块流的操作都是转换的候选。

## 错误处理和清理

转换流从可读和可写继承错误处理。如果在`_transform()`或`_flush()`中发生错误，你将其传递给回调，并且流会发出一个'error'事件。

```js
_transform(chunk, encoding, callback) {
 try {
 const result = this.process(chunk);
 this.push(result);
 callback();
 } catch (err) {
 callback(err);
 }
}
```

如果`this.process()`抛出异常，错误将被捕获并传递给回调。流会发出'error'事件，处理停止。

你也可以为`_transform()`和`_flush()`使用异步函数：

```js
async _transform(chunk, encoding, callback) {
 try {
 const result = await this.processAsync(chunk);
 this.push(result);
 callback();
 } catch (err) {
 callback(err);
 }
}
```

或者省略回调并返回一个 Promise：

```js
async _transform(chunk, encoding) {
 const result = await this.processAsync(chunk);
 this.push(result);
}
```

如果 Promise 被拒绝，Node.js 将其视为错误，并使用拒绝原因调用回调。

对于清理，你可以实现`_destroy()`，当流被销毁时会调用它：

```js
_destroy(err, callback) {
 this.cleanup();
 callback(err);
}
```

如果你的变换分配了需要在流销毁时释放的资源（文件句柄、数据库连接、计时器），这将很有用。

总是给创建或使用的变换流附加一个'error'监听器：

```js
transform.on("error", (err) => {
 console.error("Transform error:", err);
});
```

没有错误监听器，错误会导致你的进程崩溃。

## 对变换的 ObjectMode 考虑

我们多次提到了 objectMode。在 Transform 流中，你可以在可写端和可读端之间混合模式。

默认情况下，两端都是字节模式。但你可以设置：

+   `writableObjectMode: true` - 可写端接受对象，可读端发出缓冲区/字符串

+   `readableObjectMode: true` - 可写端接受缓冲区/字符串，可读端发出对象

+   `objectMode: true` - 两端都在 objectMode

示例：一个将字节解析为对象的变换：

```js
class JSONParser extends Transform {
 constructor(options) {
 super({ ...options, readableObjectMode: true });
 }
 _transform(chunk, encoding, callback) {
 try {
 const obj = JSON.parse(chunk.toString());
 this.push(obj);
 callback();
 } catch (err) {
 callback(err);
 }
 }
}
```

可写端处于字节模式（接受缓冲区），可读端处于 objectMode（发出对象）。

相反，一个将对象转换为 JSON 的变换：

```js
class JSONStringifier extends Transform {
 constructor(options) {
 super({ ...options, writableObjectMode: true });
 }
 _transform(obj, encoding, callback) {
 try {
 const json = JSON.stringify(obj);
 this.push(json);
 callback();
 } catch (err) {
 callback(err);
 }
 }
}
```

可写端处于 objectMode（接受对象），可读端处于字节模式（发出字符串/缓冲区）。

这种灵活性让你可以构建无缝地在字节流和对象流之间转换的管道。你可以有一个从文件读取的字节流，一个将字节解析为对象的变换，一个处理对象的变换，以及一个在写入目的地之前将对象序列化为字节的其他变换。

## 简化的变换创建

对于简单的变换，你不需要创建一个类。你可以通过`transform`和`flush`函数内联传递选项：

```js
import { Transform } from "stream";
 const uppercase = new Transform({
 transform(chunk, encoding, callback) {
 this.push(chunk.toString().toUpperCase());
 callback();
 },
});
```

这对于一次性变换来说很方便。你在选项对象中传递一个`transform`函数和可选的`flush`函数。Node.js 创建 Transform 并调用你的函数作为`_transform()`和`_flush()`。

你也可以直接使用带有变换函数的`stream.pipeline()`：

```js
import { pipeline } from "stream/promises";
 await pipeline(
 source,
 async function* (source) {
 for await (const chunk of source) {
 yield chunk.toString().toUpperCase();
 }
 },
 destination
);
```

这个异步生成器变成了一个 Transform。每个`yield`推送一个块。这对于简单的变换来说更加简洁，并且与异步迭代自然地结合。

对于复杂的状态化变换，使用一个类。对于管道中的简单一次性变换，使用内联选项或生成器。

## 性能考虑

`Transform`流添加了一层抽象，这会有性能成本。每个块都会通过`Transform`的内部机制 - 缓冲、事件发射、回调调用。对于高吞吐量应用，这个开销可能很重要。

如果你每秒处理数百万个小块，创建`Transform`实例和为每个块调用`_transform()`的开销可能是可测量的。在这种情况下，考虑**批处理**。一次处理一个对象，而是处理对象数组。这减少了`Transform`调用的次数。

例如，而不是：

```js
for await (const obj of stream) {
 process(obj);
}
```

使用以下方式批处理：

```js
for await (const batch of batchedStream) {
 for (const obj of batch) {
 process(obj);
 }
}
```

我们之前实现的`BatchAccumulator`变换就是这样做的。

另一个性能考虑因素是缓冲区复制。如果你的`Transform`反复调用`Buffer.concat()`来累积数据，你会在每个数据块上分配和复制缓冲区。对于大量数据，这会很慢。考虑使用更高效的数据结构，如缓冲区的链表或来自`'bl'` npm 包的`BufferList`。

对于不需要累积状态的转换，确保你没有意外地缓冲。如果你的`_transform()`立即推送每个数据块，转换是高效的。如果你在推送之前在数组或缓冲区中累积数据块，你正在增加内存压力和延迟。

在优化之前测量性能。使用 Node.js 的内置分析器或`clinic.js`来识别瓶颈。许多转换已经足够快，但高吞吐量管道需要关注这些细节。

## 测试自定义转换

当你实现自定义`Transform`时，你需要测试它。以下是一些可靠测试转换的模式。

**通过读写测试：**

```js
import { Readable, Writable } from "stream";
import { pipeline } from "stream/promises";
 const input = Readable.from(["hello", "world"]);
const output = [];
 const collector = new Writable({
 write(chunk, encoding, callback) {
 output.push(chunk.toString());
 callback();
 },
});
 await pipeline(input, myTransform, collector);
 assert.deepEqual(output, ["HELLO", "WORLD"]);
```

你创建一个具有已知数据的`Readable`源，将其通过你的`Transform`，在`Writable`中收集输出，并断言结果。

**测试边缘情况：**空输入、单个数据块、许多小数据块、大数据块、EOF 处的未完整数据，以及无效数据。为每种情况编写一个测试。

**测试背压：**

创建一个慢速的`Writable`目标并验证`Transform`是否尊重背压：

```js
const slow = new Writable({
 write(chunk, encoding, callback) {
 setTimeout(callback, 100);
 },
});
 const start = Date.now();
await pipeline(fastSource, myTransform, slow);
const elapsed = Date.now() - start;
 assert(elapsed > expectedMinimum);
```

如果`Transform`不尊重背压，它将比预期快得多，因为它没有等待慢速目标。
