# 现代异步管道与错误处理

> [原文链接](https://www.thenodebook.com/streams/modern-pipelines-error-handling)

你现在已经了解了单个流的工作方式——`Readable`产生数据，`Writable`消费它，`Transform`在中间处理它。每种流类型都有自己的缓冲、自己的背压机制、自己的事件生命周期。在实际应用中，你很少单独处理流。你将它们连接起来，创建**管道**，其中数据从源通过多个转换阶段流向最终目的地。

这就是事情变得复杂的地方。概念本身是好的——从一条流到另一条流的管道操作足够直接——但实际正确地执行它？这就是头疼开始的地方。因为当你连接流时，你正在处理**多个错误源**、**多个背压信号**和**多个资源清理场景**。如果你的管道中的任何流失败，其他流会发生什么？如果中途发生背压，它是否正确传播？当管道完成时，所有资源是否都得到了适当的清理？

本章旨在回答这些问题。我们将检查原始的`pipe()`方法，了解它存在的原因以及为什么它对于生产代码来说不足够。然后我们将深入研究`stream.pipeline()`，这是现代、推荐的用于组合流管道并具有适当错误处理和清理的方法。之后，我们将探讨特定于流式的错误处理模式——因为管道中的错误与同步代码中的错误行为不同。我们将探讨使用异步迭代作为替代管道方法，最后我们将介绍用于构建可重用、灵活管道段的先进组合模式。

本章介绍了如何正确连接流——具有适当的错误传播、资源清理和跨所有阶段的背压处理。

## pipe() 方法

你已经从可读和可写章节中学习了`pipe()`的工作方式。作为一个快速回顾：`pipe()`将可读流连接到可写流，通过在`write()`返回`false`时调用`pause()`以及在可写流发出`drain`时调用`resume()`来自动处理背压。我们在可写流章节讨论背压时广泛介绍了这种模式。

此方法返回目标流，允许你链式调用管道：

```js
readable.pipe(transform1).pipe(transform2).pipe(writable);
```

这创建了一个四阶段的管道：可读 -> 转换 1 -> 转换 2 -> 可写。数据按顺序通过每个阶段流动，**背压从可写目标反向传播到可读源**——这是我们详细探讨的可写流章节中的模式。如果`writable`发出背压信号，整个链路将暂停；当`writable`发出`drain`时，恢复信号将向前传播。

一个具体的例子——压缩日志文件：

```js
import { createReadStream, createWriteStream } from "fs";
import { createGzip } from "zlib";
 createReadStream("app.log")
 .pipe(createGzip())
 .pipe(createWriteStream("app.log.gz"));
```

三个流，两个 `pipe()` 调用。文件读取器产生数据块，gzip 转换器压缩它们，文件写入器保存压缩数据。内存使用保持有界，因为每个阶段都尊重背压信号。

但 `pipe()` 有一个问题：**错误处理**。

下面是管道中发生错误时的情况。遇到错误的流会发出一个 `error` 事件。但这个错误**不会传播**到管道中的其他流。您已经从可读和可写章节中了解到，每个流都需要自己的错误处理器，否则错误会崩溃您的进程。但在管道中，这变得尤为痛苦。

```js
const reader = createReadStream("input.txt");
const transform = createGzip();
const writer = createWriteStream("output.gz");
 reader.pipe(transform).pipe(writer);
 reader.on("error", (err) => {
 console.error("Reader error:", err);
});
 transform.on("error", (err) => {
 console.error("Transform error:", err);
});
 writer.on("error", (err) => {
 console.error("Writer error:", err);
});
```

您需要三个单独的错误处理器。错过任何一个，您的进程就会崩溃。这很繁琐，容易出错，坦白说，对于应该简单直接的事情来说，这有点荒谬。

但问题更加严重。当管道中间发生错误时，其他流**不会自动停止**。假设在处理数据块时转换器抛出错误。转换器发出 `error` 事件并停止处理。但读取器继续读取并尝试写入转换器，此时转换器处于错误状态。写入器正在等待永远不会到来的数据，并且可能永远不会发出 `finish` 事件，因为管道从未干净地完成。

您最终会得到**悬空资源**。未关闭的文件句柄。未清理的网络连接。未释放的内存缓冲区。管道处于部分失败状态，清理它需要手动对每个流调用 `destroy()`：

```js
reader.on("error", (err) => {
 reader.destroy();
 transform.destroy();
 writer.destroy();
 console.error("Pipeline failed:", err);
});
 transform.on("error", (err) => {
 reader.destroy();
 transform.destroy();
 writer.destroy();
 console.error("Pipeline failed:", err);
});
 writer.on("error", (err) => {
 reader.destroy();
 transform.destroy();
 writer.destroy();
 console.error("Pipeline failed:", err);
});
```

这很冗长，重复，且脆弱。如果您将新的流添加到管道中，您必须更新所有错误处理器以销毁新的流。

`pipe()` 的另一个限制是：您无法轻松地知道整个管道何时完成。可读流在完成读取时发出 `end` 事件。可写流在完成写入时发出 `finish` 事件。但您应该听哪一个？如果您中间有多个转换器呢？每个转换器都会发出自己的 `end` 事件。最终目标发出 `finish` 事件。您必须跟踪**正确的流上的正确事件**，这取决于管道的结构。

对于简单的双流场景，`pipe()` 方法可以很好地工作。但对于具有多个阶段和适当错误处理要求的实际生产级管道，`pipe()` 方法就不够用了。这就是为什么引入了 `stream.pipeline()` 的原因。

## `unpipe()` 方法

在介绍 `pipeline()` 之前，值得介绍一下 `unpipe()` - 虽然您很少会用到它。此方法断开一个管道流：

```js
const writer = writable();
readable.pipe(writer);
 // Later, disconnect
readable.unpipe(writer);
```

当您调用 `unpipe()` 时，可读流停止向指定的可写流发送数据。如果您不带参数调用 `unpipe()`，它将断开与所有目标流的关系：

```js
readable.unpipe();
```

你为什么要使用这个？主要是在需要根据运行时条件重定向流输出的动态路由场景中。例如，你可能正在从套接字读取，最初将数据管道传输到文件，但后来决定根据传入的数据将数据管道传输到不同的目的地：

```js
socket.on("data", (chunk) => {
 if (shouldRedirect(chunk)) {
 socket.unpipe(fileWriter);
 socket.pipe(differentWriter);
 }
});
```

但在实践中？我几乎从未需要`unpipe()`。大多数管道都是静态的 - 你在设置时定义流程，然后让它运行到完成。动态路由更适合用高级抽象来处理，比如路由流或条件转换。

关于`unpipe()`的主要事情是要知道它存在，并且当你取消管道一个流时，目标不会自动结束。当`unpipe()`被调用时，它会从源流中移除目标流的监听器。源流的状态取决于是否有任何消费者保持连接 - 如果所有管道目标都被移除且没有`data`事件监听器，流会切换回暂停模式。如果有其他消费者存在（其他管道目标或数据监听器），源流会继续发出`data`事件。如果你想关闭目标，你需要手动调用它的`end()`。

## stream.pipeline()

`stream.pipeline()`是组合流的现代方法。这个函数被添加到 Node.js 中，专门用来解决`pipe()`的错误处理和清理问题。以下是基本用法：

```js
import { pipeline } from "stream";
 pipeline(readable, transform, writable, (err) => {
 if (err) {
 console.error("Pipeline failed:", err);
 } else {
 console.log("Pipeline succeeded");
 }
});
```

而不是链式调用`pipe()`，你将所有流作为参数传递给`pipeline()`，然后是一个在管道完成或出错时被调用的回调函数。这是签名：

```js
pipeline(stream1, stream2, ..., streamN, callback)
```

`pipeline()`做了`pipe()`没有做的三件事：

1.  **自动错误传播**：如果管道中的任何流发出错误，`pipeline()`会停止管道并调用回调函数传递该错误。你不需要在每个流上单独的错误处理器。

1.  **自动清理**：当发生错误（或当管道完成时），`pipeline()`会在管道中的所有流上调用`destroy()`。文件句柄被关闭，缓冲区被释放，连接被拆除。

1.  **单个完成回调**：一个回调处理所有事情。

让我们看看这在实践中是什么样子：

```js
import { pipeline } from "stream";
import { createReadStream, createWriteStream } from "fs";
import { createGzip } from "zlib";
 pipeline(
 createReadStream("input.txt"),
 createGzip(),
 createWriteStream("output.gz"),
 (err) => {
 if (err) {
 console.error("Compression failed:", err);
 } else {
 console.log("Compression succeeded");
 }
 }
);
```

如果这些流中的任何一个发生错误 - 文件读取失败，gzip 遇到损坏的数据，文件写入遇到磁盘满错误 - 回调函数会带错误被调用，并且所有三个流都被销毁。如果一切顺利，回调函数会带`err`作为`undefined`被调用。

这比等效的带有手动错误处理的`pipe()`代码要简单。没有单独的错误监听器，没有手动销毁调用，不需要跟踪哪个流要监听完成。

在`stream/promises`模块中，也有`pipeline()`的基于 Promise 的版本：

```js
import { pipeline } from "stream/promises";
import { createReadStream, createWriteStream } from "fs";
import { createGzip } from "zlib";
 try {
 await pipeline(
 createReadStream("input.txt"),
 createGzip(),
 createWriteStream("output.gz")
 );
 console.log("Compression succeeded");
} catch (err) {
 console.error("Compression failed:", err);
}
```

基于 Promise 的版本在管道完成或任何流发生错误时返回一个解析的 Promise。这自然地与 async/await 代码配合使用。你将`pipeline()`调用包裹在 try/catch 中，错误处理就像任何其他 Promise 拒绝一样。

这是现代 Node.js 代码的**推荐模式**。使用`stream/promises`和`async/await`进行干净、可读的管道组合。

## `pipeline()`的内部工作原理

理解内部机制有助于您推理行为并调试问题。当您调用`pipeline(s1, s2, s3, callback)`时，该函数本质上：

1.  使用您在早期章节中学到的相同`pipe()`机制连接流——与我们在可写流章节中涵盖的相同自动背压处理。

1.  为所有流附加错误监听器以进行协调的错误处理

1.  当发生任何错误或完成时，对所有流调用`destroy()`。

1.  一次调用回调，传递错误或未定义。

与手动`pipe()`的主要区别是**错误协调和自动清理**。您获得相同的背压处理（我们在可写流章节中广泛讨论过），但带有生产级的错误管理。

这里有一个高度简化的概念模型，展示了基本行为。**重要**：这是一个教学简化，用于说明概念，而不是实际的实现。实际的 Node.js `pipeline()`实现（基于`pump`库）要复杂得多，并处理了许多这里没有显示的边缘情况——包括异步迭代器、生成器、复杂错误场景、一次性的回调保证和适当的流类型检测：

```js
function simplifiedPipeline(...args) {
 const callback = args.pop();
 const streams = args;
 // Connect streams with pipe()
 for (let i = 0; i < streams.length - 1; i++) {
 streams[i].pipe(streams[i + 1]);
 }
 // Track completion
 const lastStream = streams[streams.length - 1];
 lastStream.on("finish", () => {
 destroyAll(streams);
 callback();
 });
 // Handle errors
 for (const stream of streams) {
 stream.on("error", (err) => {
 destroyAll(streams);
 callback(err);
 });
 }
}
 function destroyAll(streams) {
 for (const stream of streams) {
 stream.destroy();
 }
}
```

这是一个概念模型，旨在帮助您理解行为——Node.js 中实际的`pipeline()`实现要复杂得多，它有自己的流连接逻辑和全面的边缘情况处理（一次性的回调调用、销毁流处理、复杂错误场景等）。将其视为“它做什么”而不是“它是如何做的”。

`pipeline()`处理您应该知道的边缘情况：一个在已销毁后发出错误的流。这可能在自定义流中的异步操作中发生，其中错误发生在流名义上完成之后。实际的`pipeline()`实现确保即使多个流同时出错，回调也只被调用一次。

## 使用`pipeline()`与转换函数

您甚至不需要`Transform`流实例——您可以传递**异步生成器函数**，`pipeline()`会将它们视为转换。

```js
import { pipeline } from "stream/promises";
import { createReadStream, createWriteStream } from "fs";
 await pipeline(
 createReadStream("input.txt"),
 async function* (source) {
 for await (const chunk of source) {
 yield chunk.toString().toUpperCase();
 }
 },
 createWriteStream("output.txt")
);
```

中间的异步生成器自动转换为`Transform`流。对于来自源的数据块，生成器将其转换（在这种情况下，转换为大写）并产生结果。产生的值成为输出流中的块。

这对于简单的转换效果很好。您不需要创建自定义的`Transform`类，而是可以内联编写生成器函数。它看起来像是一个循环：**对于每个输入块，生成一个输出块**。

您还可以使用返回异步迭代器的常规异步函数：

```js
async function uppercase(source) {
 for await (const chunk of source) {
 yield chunk.toString().toUpperCase();
 }
}
 await pipeline(
 createReadStream("input.txt"),
 uppercase,
 createWriteStream("output.txt")
);
```

这之所以有效，是因为`pipeline()`识别异步迭代器，并在内部自动将它们包装在转换流中。

您甚至可以链式多个生成器转换：

```js
await pipeline(
 createReadStream("log.txt"),
 async function* (source) {
 let buffer = "";
 for await (const chunk of source) {
 buffer += chunk.toString();
 const lines = buffer.split("\n");
 buffer = lines.pop();
 for (const line of lines) {
 yield line + "\n";
 }
 }
 if (buffer) yield buffer;
 },
 async function* (source) {
 for await (const line of source) {
 if (!line.startsWith("#")) {
 yield line;
 }
 }
 },
 createWriteStream("filtered.txt")
);
```

第一个生成器将缓冲区转换为字符串并将它们分割成行，使用缓冲区处理块边界。第二个生成器过滤掉以“#”开头的行。每个生成器都是一个管道阶段，`pipeline()`负责管道连接。

这是现代 Node.js 中构建流管道的推荐方式。对于简单的转换，使用内联生成器。对于复杂的具有状态的转换，创建一个`Transform`类。根据需要混合匹配。

**关于生成器函数的重要说明**：只有使用`yield`产生的值才会发送到输出流。如果生成器使用`return`来产生最终值，该值**不会**被发送到管道——它只能被直接消耗生成器的代码访问。在管道转换中，始终使用`yield`来输出数据块。

快速旁白：如果您来自 Python，这就像 itertools，但实际上是内置在语言中的。结束旁白。

## 流管道中的错误处理

在管道中处理错误变得复杂，因为它们引入了单流代码中不存在的错误场景。您已经从可读和可写章节中了解到，流为各种失败情况发出`error`事件（文件未找到、磁盘已满、网络断开等）。

在管道中，这些错误可能来自**多个来源同时**：

+   **源流**可能无法读取（文件不存在、权限被拒绝、网络断开）

+   **转换流**可能遇到无效数据（解析错误、验证失败）

+   **目标流**可能无法写入（磁盘已满、管道损坏、远程端点关闭）

这些错误中的每一个都会在遇到它的流上表现为一个`error`事件。使用`pipe()`，您必须单独处理每个错误。使用`pipeline()`，所有错误都会被捕获并传递到您的回调或承诺拒绝。

当管道中途发生错误时，部分数据会怎样？假设您正在读取一个 100MB 的文件，对其进行转换，并写入结果。在 50MB 时，转换遇到损坏的数据和错误。前 50MB 发生了什么？

答案取决于目标流的行为。如果它正在写入文件，则文件现在包含 50MB 的**部分输出**。文件存在，但不完整且可能无效。`pipeline()`**不会回滚**部分写入——它不能。数据已经写入底层资源。

这意味着您需要在应用程序逻辑中处理部分数据。一种模式是写入一个临时文件，只有在成功后将其重命名为最终名称：

```js
import { rename, unlink } from "fs/promises";
 const tempFile = "output.tmp";
const finalFile = "output.dat";
 try {
 await pipeline(source, transform, createWriteStream(tempFile));
 await rename(tempFile, finalFile);
} catch (err) {
 await unlink(tempFile); // Clean up partial file
 throw err;
}
```

如果管道成功，临时文件将被重命名为最终名称。如果失败，临时文件将被删除。这确保了要么完整的输出存在，要么什么都不存在——没有部分文件。

另一种模式是在向数据库写入时使用事务。在一个事务中写入所有行，只有当管道成功完成时才提交：

```js
const tx = await db.beginTransaction();
 try {
 await pipeline(
 source,
 transform,
 new DatabaseWriter(tx)
 );
 await tx.commit();
} catch (err) {
 await tx.rollback();
 throw err;
}
```

`pipeline()`只处理**流级别清理**——在流上调用`destroy()`。它不处理**域级别清理**（删除部分文件、回滚事务）。**这是你的责任**。

错误传播是这里有趣的地方。当管道中的流出错时，`pipeline()`会立即对其他所有流调用`destroy()`。这导致每个流发出`close`事件，并取消任何挂起的操作。这是正确的——如果一个阶段失败，整个管道应该停止。

但如果你想要对不同流中的错误进行不同的处理呢？例如，如果源失败，你想要记录“读取错误”，但如果目标失败，你想要记录“写入错误”。使用`pipeline()`，你只能在回调中获取一个错误——即第一个发生的错误。

如果你需要区分错误来源，你可以检查错误对象的属性或在自定义流中使用错误包装：

```js
class SourceStream extends Readable {
 _read() {
 const err = new Error("Read failed");
 err.code = "ERR_SOURCE_READ";
 this.destroy(err);
 }
}
 pipeline(source, transform, destination, (err) => {
 if (err && err.code === "ERR_SOURCE_READ") {
 console.error("Source read error:", err);
 } else if (err) {
 console.error("Other error:", err);
 }
});
```

通过使用`code`属性标记错误，你可以在错误处理程序中区分它们。

另一种模式是使用`stream.finished()`来检测特定流何时完成或出错，即使在更大的管道中：

```js
import { pipeline, finished } from "stream";
 const transform = createSomeTransform();
 finished(transform, (err) => {
 if (err) {
 console.error("Transform specifically failed:", err);
 }
});
 pipeline(source, transform, destination, (err) => {
 if (err) {
 console.error("Overall pipeline failed:", err);
 }
});
```

`finished()`实用工具将监听器附加到流上，并在流结束时、出错或被销毁时调用回调。这让你可以监控管道中的单个流。

## `stream.finished()`

`stream.finished()`值得仔细研究。

`finished()`函数接受一个流和一个回调，并在流完成时（无论是成功还是出错）调用回调：

```js
import { finished } from "stream";
 finished(someStream, (err) => {
 if (err) {
 console.error("Stream errored:", err);
 } else {
 console.log("Stream finished successfully");
 }
});
```

“完成”是什么意思？对于`Readable`，这意味着流已结束（推送了`null`或被销毁）。对于`Writable`，这意味着流已完成写入并发出`finish`（或被销毁）。对于`Duplex`或`Transform`，这意味着两边都已完成。

这比直接将监听器附加到`end`或`finish`更安全，因为`finished()`还监听`error`、`close`和`destroy`事件，并处理确定流是否真正完成的复杂逻辑。

此外，还有一个基于 Promise 的版本：

```js
import { finished } from "stream/promises";
 try {
 await finished(someStream);
 console.log("Stream finished");
} catch (err) {
 console.error("Stream errored:", err);
}
```

当流成功完成时，Promise 会解决；如果流出错，则会拒绝。

**关于事件监听器的说明**：`stream.finished()`故意在回调调用或 Promise 解决后留下悬空的事件监听器（尤其是`error`、`end`、`finish`和`close`）。这种设计选择允许`finished()`捕获来自不正确流实现的意外错误事件，防止崩溃。对于大多数具有短暂生命周期的流的用例，这不是一个问题，因为流将被垃圾回收。然而，对于对内存敏感的应用程序或长期存在的流，你可以使用`cleanup`选项自动删除这些监听器：

```js
import { finished } from "stream/promises";
 await finished(someStream, { cleanup: true }); // Removes listeners after completion
```

为什么使用 `finished()` 而不是仅仅监听 `end` 或 `finish`？因为流可以通过多种方式结束。流可能会自然地发出 `end`，或者由于错误而被销毁，或者通过 `destroy()` 显式销毁。`finished()` 工具处理所有这些情况，并给你一个表示“这个流无论如何都完成了”的单个回调或承诺。

这在需要知道一个复杂管道中的特定流何时完成时很有用，即使整体管道仍在运行。例如，如果你将源流传输到多个目的地（如 tee 或广播模式），你可以使用 `finished()` 来知道每个目的地是否已消耗完所有数据：

```js
const dest1 = createWriteStream("output1.txt");
const dest2 = createWriteStream("output2.txt");
 source.pipe(dest1);
source.pipe(dest2);
 await Promise.all([
 finished(dest1),
 finished(dest2),
]);
 console.log("Both destinations finished");
```

这在继续之前等待两个目标都完成写入。

## 管道中的错误恢复

并非所有错误都是致命的——一些值得重试。一些错误是**瞬态的**，可以重试。其他错误则表明存在根本问题，需要管道失败。

例如，如果你从网络源读取数据并且连接断开，这可能是瞬时的。重试连接可能会成功。但是，如果你读取文件并得到 `EACCES`（权限被拒绝），重试是没有帮助的——文件的权限不会神奇地改变。

第一步是**分类错误**。这是操作错误（预期的失败条件）还是程序员错误（代码中的错误）？它是瞬态的还是永久的？

对于瞬态错误，你可以在管道周围实现重试逻辑：

```js
async function pipelineWithRetry(maxRetries) {
 for (let attempt = 0; attempt <= maxRetries; attempt++) {
 try {
 await pipeline(source(), transform, destination);
 return; // Success
 } catch (err) {
 if (isTransientError(err) && attempt < maxRetries) {
 console.log(`Attempt ${attempt + 1} failed, retrying...`);
 await delay(1000 * Math.pow(2, attempt)); // Exponential backoff
 } else {
 throw err; // Give up
 }
 }
 }
}
 function isTransientError(err) {
 return err.code === "ECONNRESET" || err.code === "ETIMEDOUT"; // Network errors
}
```

这个函数尝试管道最多 `maxRetries` 次。如果发生瞬态错误，它将等待（使用指数退避）并重试。如果发生非瞬态错误，或者所有重试都耗尽，它将抛出错误。

注意，`source()` 是一个创建新源流的函数。一旦流出错并被销毁，就不能再重用该流。每次重试都需要新的流实例。

另一种恢复模式是回退。如果一个数据源失败，尝试一个替代源：

```js
try {
 await pipeline(primarySource, transform, destination);
} catch (err) {
 console.warn("Primary source failed, trying fallback");
 await pipeline(fallbackSource, transform, destination);
}
```

这对于冗余数据源很有用，例如首先尝试 CDN，如果 CDN 不可用，则回退到原始服务器。

对于目标失败，你可能想要尝试将写入重试到不同的位置：

```js
try {
 await pipeline(source, transform, primaryDest);
} catch (err) {
 console.warn("Primary destination failed, trying backup");
 await pipeline(source(), transform, backupDest);
}
```

再次注意，`source()` 是一个创建新源的函数。在第一个管道失败后，原始源被销毁，因此你需要一个新的源来进行重试。

**关键原则**：事先决定哪些错误是可恢复的，并在管道级别实现你的重试或回退逻辑，而不是在单个流级别。流不知道你的应用程序的重试策略——你必须在外部协调它。

## 部分数据关注

部分数据需要关注，因为它很容易出错。当管道在中间失败时，已经写入目标的数据将保留在那里。管道不会自动清理它。

这是对**数据完整性**的问题。如果你正在写入数据库导出，并且管道在 60%处失败，你将有一个 60%完成的文件。如果你稍后重试并成功，你可能会得到重复的数据（前 60%两次）或者你可能用完整的文件覆盖了部分文件，这取决于你如何打开输出文件。

这里是**处理此问题的策略**：

**1. 写入到临时位置并原子重命名**

```js
const temp = "output.tmp";
const final = "output.dat";
 try {
 await pipeline(source, transform, createWriteStream(temp));
 await rename(temp, final);
} catch (err) {
 await unlink(temp).catch(() => {}); // Clean up, ignore errors
 throw err;
}
```

这是文件输出的**最安全模式**。最终文件只有在管道成功完成后才存在。

**2. 使用追加模式和幂等操作**

如果你的输出支持追加（例如日志文件），并且你的操作是幂等的（写两次相同的数据无害），你可以从头开始重试并追加：

```js
await pipeline(source, transform, createWriteStream("output.log", { flags: "a" }));
```

如果管道失败并且你重试，第二次尝试会追加更多数据。如果你在下游有重复检测，这没问题。

**3. 使用事务性目标**

数据库、消息队列和一些云存储系统支持事务。在事务中写入，仅在成功时提交：

```js
const tx = await db.beginTransaction();
try {
 await pipeline(source, transform, new DatabaseWriter(tx));
 await tx.commit();
} catch (err) {
 await tx.rollback();
 throw err;
}
```

目标不会在管道成功之前持久化任何数据。

**4. 写入完成标记**

对于无法使用事务的场景，写入一个标记文件以指示成功完成：

```js
await pipeline(source, transform, createWriteStream("output.dat"));
await writeFile("output.dat.complete", "");
```

在处理输出文件之前，检查标记。如果标记不存在，文件不完整，应丢弃或重试。

你选择的模式取决于你的目标的能力和你的一致性要求。关键是明确部分失败时会发生什么，并设计你的管道来处理它。

## 销毁流

你已经在可读和可写章节中学习了`stream.destroy()`。作为提醒，对任何流调用`destroy()`会将它转换为销毁状态，发出`close`事件，并且如果传递了一个错误，则可选地发出`error`。

管道特有的特定之处在于，当你销毁`pipeline()`中的**任何**流时，管道函数会自动销毁所有其他流，并带错误调用回调：

```js
const source = createReadStream("input.txt");
const dest = createWriteStream("output.txt");
 pipeline(source, dest, (err) => {
 if (err) {
 console.error("Pipeline stopped:", err.message);
 }
});
 // Later, cancel the pipeline (e.g., after user action)
setTimeout(() => {
 source.destroy(new Error("Cancelled by user"));
}, 100);
```

当你调用`source.destroy()`时，源流停止读取，发出`close`事件，如果你传递了一个错误，则发出`error`事件。`pipeline()`函数看到错误并销毁管道中的所有其他流（在这种情况下，`dest`）。回调会带错误被调用。

这种自动清理是`pipeline()`相对于手动`pipe()`链的另一个优点。使用`pipe()`，你必须跟踪所有流并手动销毁它们。

这对于实现取消很有用。如果用户取消操作，你销毁管道的源流。管道优雅地停止，清理资源，并且你的回调被调用以处理取消。

你也可以不带错误地销毁：

```js
source.destroy();
```

在这种情况下，流被销毁了，但没有发出`error`事件。`pipeline()`回调仍然被调用，但`err`是`null`。这对于在不将其视为失败的情况下停止管道很有用。

一个重要的细节是：`destroy()` 是幂等的。在同一个流上多次调用它，在第一次调用之后不会有任何作用。流只销毁一次，后续的销毁调用会被忽略。

另一件事——当你销毁一个流时，缓冲数据就消失了。如果一个 `Writable` 有尚未刷新的缓冲写入，它们就会丢失。如果一个 `Readable` 有尚未消费的缓冲数据，它也会丢失。销毁意味着 **“立即停止并丢弃任何状态，”** 而不是“优雅地完成挂起的操作。”

如果你需要优雅地关闭（在关闭前完成缓冲数据的写入），请使用 `end()` 而不是 `destroy()`：

```js
writable.end(); // Finish writing buffered data, then close
```

但 `end()` 只在可写流上有效。对于可读流，没有优雅的停止方式——你只能消耗所有数据或者销毁。

## 异步迭代管道

在可读流章节中，我们探讨了使用 `for await...of` 来消费具有自动背压处理的流。这种模式也是构建流处理逻辑的 `pipe()` 和 `pipeline()` 的替代方案。

作为复习：当你使用 `for await...of` 迭代可读流时，迭代协议会自动实现背压。循环不会拉取下一个块，直到当前迭代完成。如果你的处理是异步的，流会等待：

```js
for await (const chunk of readableStream) {
 await processAsync(chunk); // Stream waits for this
}
```

我们在这里不会重复完整的背压机制——有关迭代协议如何管理流控制的完整解释，请参阅可读流章节中的“异步迭代中的背压”部分。

本章相关的是特别使用这种模式进行 **管道构建** 而不是 `pipeline()`。你可以通过从源读取、转换每个块并写入目标来构建管道：

```js
const source = createReadStream("input.txt");
const dest = createWriteStream("output.txt");
 for await (const chunk of source) {
 const transformed = chunk.toString().toUpperCase();
 const ok = dest.write(transformed);
 if (!ok) {
 await new Promise((resolve) => dest.once("drain", resolve));
 }
}
 dest.end();
```

这是一个手动管道。你从源拉取数据，进行转换，并写入目标，具有显式的背压处理（如果 `write()` 返回 false，则等待 `drain`）。这种模式让你对数据流有精细的控制，但需要你手动管理背压——忘记 `drain` 检查会导致内存无限制增长，正如我们在可写流章节中讨论的那样。

一种更干净的模式是使用 `stream.Readable.from()` 与异步生成器：

```js
async function* transform(source) {
 for await (const chunk of source) {
 yield chunk.toString().toUpperCase();
 }
}
 const source = createReadStream("input.txt");
const transformed = Readable.from(transform(source));
const dest = createWriteStream("output.txt");
 await pipeline(transformed, dest);
```

异步生成器会自动包装在一个可读流中，`pipeline()` 处理管道连接。这结合了 `for await...of` 的清晰性和 `pipeline()` 的健壮性。

你可以链式调用多个生成器转换：

```js
async function* toUppercase(source) {
 for await (const chunk of source) {
 yield chunk.toString().toUpperCase();
 }
}
 async function* filterEmpty(source) {
 for await (const line of source) {
 if (line.trim().length > 0) {
 yield line;
 }
 }
}
 await pipeline(
 source,
 toUppercase,
 filterEmpty,
 dest
);
```

每个生成器是一个管道阶段。这是可读的。每个阶段是一个简单的 `for await` 循环，产生转换后的块。组合由 `pipeline()` 处理。

这种模式对于 **`objectMode` 管道**特别有用，其中每个块都是一个结构化对象：

```js
async function* parseJSON(source) {
 for await (const line of source) {
 yield JSON.parse(line);
 }
}
 async function* extractField(source, field) {
 for await (const obj of source) {
 yield obj[field];
 }
}
 await pipeline(
 source,
 parseJSON,
 (source) => extractField(source, "name"),
 dest
);
```

每个阶段都是一个从源到异步可迭代对象的函数。`pipeline()` 函数将它们连接起来。这是函数式管道组合，通常比创建转换类更清晰。

## 管道中的异步迭代背压

在可读流章节中，我们介绍了 `for await...of` 如何与背压一起工作。在管道中使用此模式时需要记住的关键点：

1.  **等待异步工作** - 迭代器一次拉取一个块；如果您等待异步操作，则从您的处理速度自动向源流动背压

1.  **不要累积承诺** - 避免使用 `promises.push(processAsync(chunk))` 模式，该模式在处理之前将整个流读入内存，从而破坏背压

1.  **使用受控并发** - 对于具有有限并发的并行处理，请使用 `p-limit` 等库来限制正在进行的操作

从可读流章节：**您必须在循环内等待异步操作**。没有 `await`，您会失去背压：

```js
// WRONG - loses backpressure
for await (const chunk of source) {
 slowOperation(chunk); // No await! Loop continues immediately
}
 // CORRECT - maintains backpressure
for await (const chunk of source) {
 await slowOperation(chunk); // Stream waits until this completes
}
```

与 `pipeline()` 相比，这种方法的优势在于对数据流的显式控制。缺点是您必须自己管理错误处理——没有 `pipeline()` 提供的自动清理。

关于异步迭代协议如何实现背压的完整机制，请参阅可读流章节中的“异步迭代中的背压：它是如何工作的”部分。

## 可组合的转换

在转换流章节中，我们介绍了如何实现自定义转换。现在让我们看看如何通过工厂函数构建**可重用管道组件**。

模式很简单——创建返回配置流实例的函数：

```js
function createCSVParser() {
 return new Transform({
 objectMode: true,
 transform(chunk, encoding, callback) {
 const lines = chunk.toString().split("\n");
 for (const line of lines) {
 if (line.trim()) {
 this.push(line.split(","));
 }
 }
 callback();
 },
 });
}
 // Use in multiple pipelines
await pipeline(source1, createCSVParser(), dest1);
await pipeline(source2, createCSVParser(), dest2);
```

每次调用 `createCSVParser()` 都会返回一个新的转换实例。您不能在多个管道中重用流实例（一旦流结束或出错，它就完成了），但您可以重用工厂函数。

您可以使工厂可配置：

```js
function createFieldExtractor(fields) {
 return new Transform({
 objectMode: true,
 transform(obj, encoding, callback) {
 const extracted = {};
 for (const field of fields) {
 extracted[field] = obj[field];
 }
 this.push(extracted);
 callback();
 },
 });
}
 await pipeline(
 source,
 createFieldExtractor(["name", "email"]),
 dest
);
```

现在转换是参数化的。不同的管道可以提取不同的字段。

对于更复杂的管道，您可以组合管道段：

```js
function createProcessingPipeline(source, dest) {
 return pipeline(
 source,
 createCSVParser(),
 createFieldExtractor(["name", "email"]),
 createValidator(),
 dest
 );
}
 await createProcessingPipeline(source1, dest1);
await createProcessingPipeline(source2, dest2);
```

`createProcessingPipeline()` 函数封装了整个转换序列。您传入源和目的地，然后它会连接所有中间转换。

这是一个高阶函数模式：创建和组合流的函数。这对于构建模块化、可测试的流代码很有用。

您也可以组合生成器函数：

```js
const parseCSV = async function* (source) {
 for await (const chunk of source) {
 const lines = chunk.toString().split("\n");
 for (const line of lines) {
 if (line.trim()) {
 yield line.split(",");
 }
 }
 }
};
 const extractFields = (fields) =>
 async function* (source) {
 for await (const obj of source) {
 const extracted = {};
 for (const field of fields) {
 extracted[field] = obj[field];
 }
 yield extracted;
 }
 };
 await pipeline(
 source,
 parseCSV,
 extractFields(["name", "email"]),
 dest
);
```

每个生成器都是一个从异步可迭代到异步可迭代的功能。您通过将一个函数的输出作为另一个函数的输入来组合它们。`pipeline()` 函数处理管道连接。

这种函数组合风格与 Node.js 的流模型自然契合。

## 管道段

让我们正式化管道段的概念。一个段是管道的可重用部分——它可能是一个转换，或一系列转换，或条件逻辑，将路由到不同的转换。

这里是一个验证对象并决定是将其传递通过还是将其路由到错误目的地的段：

```js
function createValidationSegment(schema, errorDest) {
 const valid = new PassThrough({ objectMode: true });
 const invalid = new PassThrough({ objectMode: true });
 invalid.pipe(errorDest);
 return new Transform({
 objectMode: true,
 transform(obj, encoding, callback) {
 if (schema.validate(obj)) {
 this.push(obj);
 } else {
 invalid.write(obj);
 }
 callback();
 },
 });
}
```

这个转换验证每个对象。有效的对象被推送到下游。无效的对象被写入错误目的地（如日志文件或错误流）。这是一个分支段：一个输入，两个输出。

你可以这样使用它：

```js
const errorLog = createWriteStream("errors.log");
 await pipeline(
 source,
 createValidationSegment(mySchema, errorLog),
 dest
);
```

有效的对象发送到 `dest`。无效的对象发送到 `errorLog`。即使遇到无效对象，管道也会继续运行——它们只是被路由到其他地方。

另一个模式是条件段，根据运行时状态选择不同的转换：

```js
function createConditionalSegment(condition, trueTransform, falseTransform) {
 return new Transform({
 objectMode: true,
 async transform(obj, encoding, callback) {
 try {
 const result = await condition(obj);
 const transform = result ? trueTransform : falseTransform;
 transform.write(obj);
 callback();
 } catch (err) {
 callback(err);
 }
 },
 });
}
```

根据条件函数，每个对象要么通过 `trueTransform`，要么通过 `falseTransform`。这是一个路由段。

这些模式——分支、路由、条件——是复杂数据流的构建块。你将它们组合成更大的管道，创建复杂的处理逻辑，同时保持每个段集中和可重用。

## Tee 和广播模式

有时你需要将相同的数据发送到多个目的地。这被称为 tee（类似于管道中的 T 形连接）或广播模式。

Tee 流的最简单方法是将其管道连接到多个目的地：

```js
source.pipe(dest1);
source.pipe(dest2);
```

`dest1` 和 `dest2` 都从 `source` 接收相同的数据。源每次只发射每个数据块一次，两个管道将其分别转发到各自的目的地。

但有一个问题：背压。如果 `dest1` 慢并且发出背压信号，则源暂停。但 `dest2` 可能很快，准备好接收更多数据。通过暂停源，你无必要地减慢了 `dest2` 的速度。

源不能只为一个目的地暂停，而继续为另一个目的地服务。它是 **全有或全无** 的。所以当你使用 `pipe()` 将流 tee 时，**最慢的目的地控制所有目的地的节奏**。

如果这是可以接受的（所有目的地都需要相同的数据，并且你接受最慢的一个设置节奏），那么简单的管道就足够了。但如果你想要每个目的地都有独立的背压，你需要更复杂的方法。

一种技术是使用 `PassThrough` 流作为中间代理：

```js
const pass1 = new PassThrough();
const pass2 = new PassThrough();
 source.on("data", (chunk) => {
 pass1.write(chunk);
 pass2.write(chunk);
});
 source.on("end", () => {
 pass1.end();
 pass2.end();
});
 pass1.pipe(dest1);
pass2.pipe(dest2);
```

现在 `dest1` 和 `dest2` 有独立的背压。如果 `dest1` 慢，`pass1` 缓冲。如果 `dest2` 快，`pass2` 不缓冲。源不会被任何一个目的地暂停。

但这打破了源级别的背压。如果两个目的地都慢，`pass1` 和 `pass2` 都会缓冲，而源并不知情。你在内存中无限制地缓冲。

对于具有有限内存的真正独立目的地，你需要使用一个扇出流，该流监控所有目的地的背压，并且仅在所有目的地发出背压信号时才暂停源：

```js
class FanOut extends Writable {
 constructor(destinations, options) {
 super(options);
 this.destinations = destinations;
 }
 _write(chunk, encoding, callback) {
 const allReady = [];
 // Write to all destinations and check for backpressure
 for (const dest of this.destinations) {
 const canContinue = dest.write(chunk, encoding);
 if (!canContinue) {
 // Destination buffer is full, wait for drain
 allReady.push(
 new Promise((resolve) => {
 dest.once('drain', resolve);
 })
 );
 }
 }
 // If any destination signaled backpressure, wait for all to drain
 if (allReady.length > 0) {
 Promise.all(allReady)
 .then(() => callback())
 .catch((err) => callback(err));
 } else {
 // All writes succeeded without backpressure
 callback();
 }
 }
 _final(callback) {
 // End all destinations when this stream ends
 for (const dest of this.destinations) {
 dest.end();
 }
 callback();
 }
}
```

这个可写流将每个块转发到多个目的地。与简单管道的关键区别在于它如何处理背压：`write()` 返回一个布尔值，指示你是否可以继续写入。如果它返回 `false`，目的地的缓冲区已满，你应该在写入更多之前等待 `drain` 事件。此实现收集任何表示背压的 destinations 的承诺，并在调用回调之前等待所有这些承诺都排空，这会在源上创建背压。

你可以这样使用它：

```js
pipeline(source, new FanOut([dest1, dest2]), (err) => {
 // Pipeline done
});
```

这是一个更复杂的模式，在应用程序代码中并不常见。大多数时候，你或者接受最慢的目的地控制节奏，或者接受 PassThrough 中间件的未限定缓冲。真正的扇出，每个目的地都有独立的背压，是复杂的，通常只在像日志或监控系统这样的特定场景中需要。

## 终止信号集成

Node.js 流支持 `AbortSignal` 用于取消。你可以将信号传递给基于承诺的 `pipeline()`，如果信号被终止，管道将被销毁：

```js
import { pipeline } from "stream/promises";
 const controller = new AbortController();
 try {
 await pipeline(source, transform, dest, { signal: controller.signal });
} catch (err) {
 if (err.name === "AbortError") {
 console.log("Pipeline cancelled");
 } else {
 throw err;
 }
}
 // Note: You can also check err.code === 'ABORT_ERR' which is more robust
// since the code property is harder to accidentally modify
 // To cancel: controller.abort();
```

当你调用 `controller.abort()` 时，管道立即被销毁。所有流都被拆除，承诺因 **`AbortError`** 而拒绝，并且任何挂起的操作都被取消。

这对于用户发起的取消、超时或在长时间运行的操作中的资源清理很有用。

你也可以创建一个超时信号：

```js
const signal = AbortSignal.timeout(5000); // 5 second timeout
 try {
 await pipeline(source, transform, dest, { signal });
 console.log("Pipeline completed");
} catch (err) {
 if (err.name === "AbortError") {
 console.log("Pipeline timed out");
 } else {
 throw err;
 }
}
```

如果管道在 5 秒内没有完成，它将自动终止。

对于具有多个取消源的复杂场景，你可以使用 `AbortSignal.any()`：

```js
const userCancel = new AbortController();
const timeout = AbortSignal.timeout(10000);
 const signal = AbortSignal.any([userCancel.signal, timeout]);
 try {
 await pipeline(source, transform, dest, { signal });
} catch (err) {
 if (err.name === "AbortError") {
 console.log("Cancelled by either user or timeout");
 } else {
 throw err;
 }
}
```

这会创建一个复合信号，如果用户取消或超时到期，则会终止。

终止信号集成使取消变得明确和标准化。而不是手动在流上调用 `destroy()`，你终止一个信号，管道处理清理。

## 实际的管道示例

让我们实现几个完整的管道，以展示我们已覆盖的所有概念。

**1) 日志文件处理**

读取大日志文件，将每一行解析为 JSON，按日志级别过滤，并写入单独的输出文件：

```js
import { pipeline } from "stream/promises";
import { createReadStream, createWriteStream } from "fs";
import { Transform } from "stream";
 async function* parseLines(source) {
 let buffer = "";
 for await (const chunk of source) {
 buffer += chunk.toString();
 const lines = buffer.split("\n");
 buffer = lines.pop();
 for (const line of lines) {
 if (line.trim()) {
 try {
 yield JSON.parse(line);
 } catch (err) {
 console.error("Parse error:", err);
 }
 }
 }
 }
 if (buffer.trim()) {
 try {
 yield JSON.parse(buffer);
 } catch (err) {
 console.error("Parse error:", err);
 }
 }
}
 class LevelSplitter extends Transform {
 constructor(level, dest, options) {
 super({ ...options, objectMode: true });
 this.level = level;
 this.dest = dest;
 }
 _transform(log, encoding, callback) {
 if (log.level === this.level) {
 this.dest.write(JSON.stringify(log) + "\n");
 }
 this.push(log);
 callback();
 }
}
 const errorDest = createWriteStream("errors.log");
const warnDest = createWriteStream("warnings.log");
 await pipeline(
 createReadStream("app.log"),
 parseLines,
 new LevelSplitter("ERROR", errorDest),
 new LevelSplitter("WARN", warnDest),
 createWriteStream("all.log")
);
```

`parseLines` 生成器处理基于行流的常见挑战：块与行边界不对齐。一个块可能在行中间结束，将 `{"level":"ERROR"...` 分割到两个块中。解决方案使用缓冲区累积：

```js
let buffer = "";
for await (const chunk of source) {
 buffer += chunk.toString();
 const lines = buffer.split("\n");
 buffer = lines.pop(); // Save incomplete line for next chunk
 // Process complete lines...
}
```

当你在 `\n` 上分割时，最后一个数组元素要么是空的（如果块以换行符结束），要么是不完整的行。弹出该元素并保存它意味着下一个块将附加到它，完成该行。循环结束后，任何剩余的缓冲数据都会被处理——处理不以换行符结尾的文件。

`JSON.parse()` 周围的 `try/catch` 是关键的：

```js
try {
 yield JSON.parse(line);
} catch (err) {
 console.error("Parse error:", err);
}
```

没有错误处理，单个格式错误的 JSON 行会导致整个管道崩溃，丢失所有进度。有了它，管道记录错误并继续。现实世界的日志文件包含损坏的条目，因此管道需要在不停机的情况下处理无效数据。

`LevelSplitter` 既可以过滤数据到侧通道，也可以传递所有数据：

```js
_transform(log, encoding, callback) {
 if (log.level === this.level) {
 this.dest.write(JSON.stringify(log) + "\n"); // Side channel
 }
 this.push(log); // Pass through to next stage
 callback();
}
```

每个日志条目都会沿着主管道继续向下传递，但 ERROR 级别的日志也会写入 `errors.log`。这创建了一个分支管道：

```js
Logs → parseLines → [All logs continue]
 ↓
 ERROR logs → errors.log
 ↓ [All logs continue]
 WARN logs → warnings.log
 ↓ [All logs continue]
 All logs → all.log
```

这种方法内存效率高。一次读取文件并在流中分割使用恒定内存。两个独立的管道会加倍 I/O 和内存使用。

`{ objectMode: true }` 选项至关重要，因为此转换从 `parseLines` 接收 JavaScript 对象，而不是缓冲区。当写入侧目的地时，我们使用 `JSON.stringify(log) + "\n"` 将其转换回 JSON 字符串。一次解析，在管道中处理对象，仅在写入磁盘时进行序列化。

分割器按顺序链接：

```js
parseLines → LevelSplitter("ERROR") → LevelSplitter("WARN") → all.log
```

每个分割器调用 `this.push(log)`，传递对象。最终目的地 `all.log` 也会接收对象，但由于可写流会自动在对象上调用 `toString()`，因此你可能想要添加一个最终转换，将其序列化为 JSON 以进行适当的格式化（为了清晰起见，我们简化了这一点，但在生产中你会在最终目的地之前添加 `async function* serializeJSON(source) { for await (const obj of source) yield JSON.stringify(obj) + "\n"; }`）。

**2) 带验证的 CSV 导入**

读取 CSV 文件，解析行，验证，并以批量方式插入到数据库中：

```js
async function* parseCSV(source) {
 let buffer = "";
 let headers = null;
 for await (const chunk of source) {
 buffer += chunk.toString();
 const lines = buffer.split("\n");
 buffer = lines.pop();
 for (const line of lines) {
 if (!headers) {
 headers = line.split(",");
 } else {
 const values = line.split(",");
 const row = {};
 for (let i = 0; i < headers.length; i++) {
 row[headers[i]] = values[i];
 }
 yield row;
 }
 }
 }
}
 async function* validate(source, schema) {
 for await (const row of source) {
 if (schema.validate(row)) {
 yield row;
 } else {
 console.error("Invalid row:", row);
 }
 }
}
 async function* batch(source, size) {
 let batch = [];
 for await (const item of source) {
 batch.push(item);
 if (batch.length >= size) {
 yield batch;
 batch = [];
 }
 }
 if (batch.length > 0) {
 yield batch;
 }
}
 class DatabaseWriter extends Writable {
 constructor(db, options) {
 super({ ...options, objectMode: true });
 this.db = db;
 }
 async _write(batch, encoding, callback) {
 try {
 await this.db.insertMany(batch);
 callback();
 } catch (err) {
 callback(err);
 }
 }
}
 await pipeline(
 createReadStream("data.csv"),
 parseCSV,
 (source) => validate(source, mySchema),
 (source) => batch(source, 100),
 new DatabaseWriter(db)
);
```

此管道使用可组合的生成器函数进行数据转换和批量处理数据库操作。

与 `parseLines` 不同，`parseCSV` 在块之间保持状态——它需要记住标题行：

```js
let buffer = "";
let headers = null; // Persists across all chunks
 for await (const chunk of source) {
 // ... process chunks
 if (!headers) {
 headers = line.split(","); // First line becomes headers
 } else {
 // Subsequent lines become data objects
 const values = line.split(",");
 const row = {};
 for (let i = 0; i < headers.length; i++) {
 row[headers[i]] = values[i];
 }
 yield row;
 }
}
```

`headers` 变量在生成器的整个生命周期中持续存在，捕获第一行并使用它来结构化所有后续行。原始 CSV：

```js
name,email,age
Alice,alice@example.com,30
Bob,bob@example.com,25
```

变成结构化对象：

```js
{ name: "Alice", email: "alice@example.com", age: "30" }
{ name: "Bob", email: "bob@example.com", age: "25" }
```

我们不会将整个 CSV 文件加载到内存中——每一行都在数据流通过时进行处理。

`validate` 生成器在不对数据进行修改的情况下进行过滤：

```js
async function* validate(source, schema) {
 for await (const row of source) {
 if (schema.validate(row)) {
 yield row; // Valid rows continue
 } else {
 console.error("Invalid row:", row); // Invalid rows logged, not yielded
 }
 }
}
```

有效的行继续向下流。无效的行被记录但不会产生，防止不良数据到达数据库。

在无效数据上抛出错误会导致管道崩溃并丢失所有进度。现实世界的数据很混乱——在导入完成后，记录日志并继续可以让你检查错误。

`batch` 生成器对于数据库操作至关重要：

```js
async function* batch(source, size) {
 let batch = [];
 for await (const item of source) {
 batch.push(item);
 if (batch.length >= size) {
 yield batch; // Emit full batch
 batch = []; // Reset for next batch
 }
 }
 if (batch.length > 0) {
 yield batch; // Don't forget partial final batch
 }
}
```

这将单个项的流转换为批次的流：

```js
Input:  item1, item2, item3, ..., item100, item101, ...
Output: [item1...item100], [item101...item200], ...
```

数据库往返很昂贵。将数据批量分组到 100 可以提供比逐行插入快 100 倍的速度。批量大小是一个权衡：太小意味着许多往返和慢速性能；太大意味着内存使用高，超时风险高，错误恢复更困难。大多数数据库在 100-1000 之间的批量大小下表现良好。

循环之后的`if (batch.length > 0)`处理了最后的部分批次。如果没有这个检查，尾部行会被静默丢弃。

`DatabaseWriter`的可写处理异步 I/O：

```js
async _write(batch, encoding, callback) {
 try {
 await this.db.insertMany(batch);
 callback(); // Signal success
 } catch (err) {
 callback(err); // Signal error
 }
}
```

`_write`方法可以是异步的，但你仍然必须调用回调。在成功时，使用无参数调用`callback()`，或者在出现错误时调用`callback(err)`以传播错误。

当`await this.db.insertMany(batch)`运行时，流会被暂停。下一个批次将在当前插入完成之前不会发送，从而防止数据库过载。

箭头函数`(source) => validate(source, mySchema)`让你可以向生成器传递额外的参数：

```js
await pipeline(
 createReadStream("data.csv"),
 parseCSV,
 (source) => validate(source, mySchema),
 (source) => batch(source, 100),
 new DatabaseWriter(db)
);
```

你正在为这个特定管道创建通用生成器的专用版本。

整个管道使用大约恒定的内存，无论文件大小如何。CSV 解析只保留当前行在内存中。验证过程一次处理一行。批处理最多保留 100 行。写入过程只处理当前批次。一个 10GB 的 CSV 文件使用的内存与一个 10MB 的文件相同。
