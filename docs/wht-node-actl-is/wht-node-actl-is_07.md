# 缓冲区分配模式

> 原文：[`www.thenodebook.com/buffers/allocation-patterns`](https://www.thenodebook.com/buffers/allocation-patterns)

## TL;DR - 对于没有耐心的人来说

现在你已经对`Buffer`有了概念，让我们将这种理解向前推进——以一种更实际的方式。假设你的服务由于你分配缓冲区的方式而正在泄露秘密或运行速度比在糖浆中的树懒还要慢。你如何识别原因？

有三种主要方式会导致这种情况，每种方式都有一种特定的方式来破坏你的日子。

1.  **`Buffer.alloc(size)`** 是你的慢速、安全、可靠的伙伴。它会请求内存，然后立即在你给出的每个字节上写入零。这保证了你永远不会看到来自系统其他部分的旧敏感数据。问题是？“零填充”需要时间。在一个紧密的循环中，在一个热点路径上，这可能会成为你最大的 CPU 瓶颈，将你的服务锁定在 100%，并降低你的吞吐量。你默认使用这个选项，只有当分析器大声疾呼这条特定的行是你的问题时，你才会更改它。

1.  **`Buffer.allocUnsafe(size)`** 是“快速移动并破坏事物”选项，这里的“事物”指的是你的数据隐私和安全状态。它非常快，因为它只是抓取一块内存并给你，不管是什么垃圾。这“垃圾”可以是任何东西——前一个用户会话令牌的片段、数据库凭证、个人信息（PII）或内存中几分钟前的 API 密钥。如果你使用这个选项并且没有**立即覆盖每个字节**，你正在积极泄露数据。它可能是一个日志文件、通过网络套接字，或者进入缓存。这是一个定时炸弹，唯一可以考虑使用它的时候是当你已经证明`Buffer.alloc()`太慢，并且你有一个会完全填充缓冲区的函数，比如`fs.readSync`。

1.  **`Buffer.from(source)`** 是变色龙。它看起来很方便，但它的行为和性能特征会根据你传递给它的内容而大幅变化。给它一个字符串，它会消耗 CPU 周期进行编码。给它一个数字数组，它会迭代并逐个复制它们。给它另一个 Buffer，它会创建一个完整的副本。但是，如果它接收到的底层内存是像`ArrayBuffer`的`.buffer`这样的特定类型，它可能会创建对该内存的*视图*而不是副本。这意味着如果原始内存发生变化，你的“不可变”Buffer 会默默地破坏自己。如果你不小心，`Buffer.from()`可能会导致微妙的内存损坏错误，这在生产环境中追踪起来是一场噩梦。

简而言之 - 从`alloc()`开始。只有在分析器强制你这样做，并且你可以保证完全、立即覆盖时才使用`allocUnsafe()`。并且三倍检查你传递给`from()`的内容，因为它的便利性隐藏了危险复杂性。

## 你在未初始化的内存中泄露了密码

让我们创建一个假设场景来使事情变得有趣。

想象一下你正在深夜为一个项目工作，试图把事情搞定，而一个生活在不同时区的测试员正在对你大喊大叫。不是崩溃，而是一张高优先级的客户工单。一个用户报告说，当他们尝试下载他们的发票 PDF 时，文件被损坏了。奇怪的是，这个损坏的文件似乎包含了一段看起来像是另一个用户的 API 密钥的片段。你把它当作一个偶然事件，一些奇怪的客户端渲染错误。你道歉，手动重新生成发票，然后试图回去睡觉。

但你无法做到。那个“API 密钥”部分让你感到困扰。

你调出用户的请求日志。有一个错误，但不是你预期的。是下游服务抱怨你发送的请求格式不正确。你查看记录的出站请求的有效负载。然后...你意识到出了大问题。应该包含发票数据的 JSON 有效负载被破坏了。它开始是正确的，但随后被乱码所尾随。在那乱码中，你清楚地看到——`...","line_items": [...], "total": 19.99}} ... bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`。这是一个该死的 JSON Web Token。来自另一个用户请求的会话令牌，就那样静静地坐在一个损坏的发票有效负载的中间。

这怎么可能呢？你追踪代码从发票生成到下游 API 调用的过程。这是一个简单的流程。生成发票的 HTML，将其转换为 PDF 流，缓冲流，然后发送。你找到了创建出站请求缓冲区的代码行。

```js
const payloadBuffer = Buffer.allocUnsafe(estimatedSize);
```

`Buffer.allocUnsafe`。但为什么？你知道在这个上下文中“unsafe”意味着什么。它并不意味着“可能会抛出错误”。它的意思是“包含未初始化的内存”。它的意思是，你向操作系统请求了一块内存，它给了你一个指向刚刚被其他东西使用的位置的指针，而它没有先清理它。在这种情况下，“其他东西”是一个刚刚处理完另一个用户认证请求的不同请求处理器。他们的 JWT 仍然坐在那个内存段中。

你的代码计算了`estimatedSize`错误。它太大。你的应用程序将有效的发票数据写入缓冲区的开始处，但从未覆盖末尾的垃圾数据。然后它发送了整个缓冲区——你的数据和另一个用户的秘密——到下游服务，并记录了它。

你开始搜索日志中的“bearer”和“password”。结果滚动起来感觉像永恒。自从那次“性能优化”被提交以来，你已经泄露了几个月的用户秘密片段。每次使用`allocUnsafe`分配缓冲区而没有完全写入时，你都在玩俄罗斯轮盘赌，赌的是用户的数据。今晚，子弹落下了。这不仅仅是一个错误，而是一场全面的安全漏洞，源于一行被误解的单行代码。

## 理解缓冲内存架构

在我们真正剖析我们刚刚找到的混乱之前，我们需要谈谈 Node.js 是如何处理内存的。当你考虑 Node 应用程序中的内存时，你可能会想到 V8 堆。这是你的 JavaScript 对象、字符串、数字和函数所在的地方。它由 V8 垃圾收集器（GC）管理，它出色地清理了你不再使用的对象。

然而，缓冲区是特殊的。它们被设计用来高效地处理二进制数据，通常是大量的数据。将兆字节的原始二进制数据塞入 V8 堆会非常低效，并且会给垃圾收集器带来巨大的压力。V8 优化用于大量小型、相互关联的 JavaScript 对象，而不是单一的二进制大块。

因此，Node.js 做了一些巧妙的事情。你在 JavaScript 代码中创建的 `Buffer` 实例实际上是在 V8 堆上的一个小对象，它充当指针或句柄。缓冲区的 *实际* 二进制数据生活在 V8 堆之外的所谓 "堆外" 内存中。这是 Node.js 直接从底层操作系统请求的原始内存，由 Node 的 C++ 核心管理。

当你运行 `const buf = Buffer.alloc(1000)` 时，以下是发生的情况 -

1.  节点的 C++ 层请求操作系统分配 1000 字节的内存。

1.  操作系统找到一个空闲块，并将内存地址分配给节点。

1.  节点的 C++ 层封装了这个内存地址。

1.  回到 JavaScript，一个小的 `Buffer` 对象在 V8 堆上创建。这个对象包含 `length` 等属性，但最重要的是，它持有对该堆外内存地址的内部引用。

这种分离对于性能至关重要。你可以在你的 JS 代码中传递这些缓冲区，而你只是在移动小的 V8 堆对象，而不是可能巨大的二进制数据块本身。Node 的核心中的 C++ 绑定（如 `fs` 和 `net`）可以直接操作这种堆外内存，而无需在 JavaScript 世界和 C++ 世界之间不断复制数据。

你可以亲自查看。运行一个简单的 Node 脚本并检查内存使用情况。

```js
// We allocate a 50MB buffer off-heap.
const bigBuffer = Buffer.alloc(50 * 1024 * 1024);
 // This will show where that memory is accounted for.
console.log(process.memoryUsage());
```

输出将类似于以下内容 -

```js
{
 "rss": 39845888,
 "heapTotal": 5341184,
 "heapUsed": 3638280,
 "external": 53790468,
 "arrayBuffers": 52439315
}
```

查看与 `heapUsed` 和 `external` 的对比。V8 堆只为脚本对象使用了大约 3.64 MB 的内存。但 `external` 属性显示了为我们的缓冲区分配的 ~53 MB。这就是堆外内存的作用。

这也是`allocUnsafe`危险来源的地方。出于安全原因，V8 堆在创建新对象时总是将其内存清零。V8 不会显示来自其他对象遗留的数据。但是 Node 管理的堆外内存是另一回事。它更接近底层。当你使用`allocUnsafe`时，Node 会向操作系统请求内存，然后直接将指针传递给你。它跳过了清除内存的步骤。为了性能原因，运行时分配器在**释放**内存时不会清除内存。它只是将其标记为“可用”。所以你得到的是之前留下的任何东西。这是即将探讨的安全风险的根本架构细节。

ℹ️注意

***当你的程序释放内存时，分配器经常在稍后没有擦除的情况下将相同的内存返回。Node 的缓冲池使用这些重用的部分，所以 Buffer.allocUnsafe()可以返回之前工作留下的字节。操作系统在将其映射到另一个进程时会将内存清零，但不会总是清除你自己的进程内部正在回收的内存**。

## `Buffer.alloc()`

`Buffer.alloc(size)`是你在 99%的情况下应该使用的函数。它是你的安全网。它的行为简单、可预测且安全。当你调用它时，它执行两个不同的操作 -

1.  它请求指定`size`的堆外内存块，这被称为**分配**。

1.  然后它遍历新分配内存的每一个字节，并将它们写入`0x00`，这被称为**零填充**。

第二步是关键。它保证了你收到的缓冲区是干净的。它不包含来自先前操作遗留的数据，没有秘密，没有垃圾。它是一张白纸。

让我们看看实际操作。

```js
// Allocate a buffer the safe way.
const buf = Buffer.alloc(10);
 // You can be 100% certain of its contents.
console.log(buf);
// <Buffer 00 00 00 00 00 00 00 00 00 00>
```

无论你运行多少次，无论你的应用程序在毫秒前做了什么，结果总是相同的——一个充满零的缓冲区。这种可预测性是它成为默认选择的原因。你不必去想它。它只是工作。你可以向其中写入数据，知道你从一个已知良好的状态开始。

但这种安全是有代价的。那个零填充步骤不是免费的。在 C++中，这是一个`memset(0)`调用，它是一个写入内存的循环。对于小缓冲区，这个成本是可以忽略不计的，被应用程序的噪音所掩盖。但是当你处理大缓冲区，或者你在紧密循环中分配许多小缓冲区时会发生什么呢？

这引出了我们的第二个假设场景，**神秘的内存峰值**。

你是负责一个高吞吐量图像处理服务的团队的一员。该服务接收图像上传，调整大小，并存储结果。几周来，一切都很顺利。但随着流量的增加，你在高峰时段开始收到高 CPU 警报。服务节点以 95-100%的 CPU 运行，延迟飙升，请求开始超时。

你从一个苦苦挣扎的实例中拉取了 CPU 配置文件。你预计会在图像缩放库（`sharp` 或类似库）中看到瓶颈，因为那是最昂贵的计算工作。但分析器告诉了一个不同的故事。最热的函数，消耗最多 CPU 时间的函数，是 Node.js 内部的，并且它被调用在您的代码中的一个地方。

```js
// This runs for every incoming image chunk.
function processChunk(chunk) {
 // We need a new buffer to apply a watermark.
 const workBuffer = Buffer.alloc(chunk.length);
 chunk.copy(workBuffer);
 // ... rest of the watermarking logic
}
```

该服务每秒处理数千个块，每个块都会分配一个新的、零填充的缓冲区。`Buffer.alloc()` 调用的巨大数量意味着 CPU 的大量时间都花在了仅仅...向内存写入零上。图像处理逻辑很快，但它正被内存分配策略所消耗的 CPU 周期所拖累。

这就是 `Buffer.alloc()` 的权衡。它的安全性和可预测性是以 CPU 周期为代价的。在大多数 Web 应用中，处理 JSON API 或数据库结果，这种成本完全无关紧要。网络 I/O 或数据库查询时间将远远超过分配时间。但在高性能、数据密集型应用，如视频流、实时数据处理或我们的图像服务中，这种成本可能成为主要的性能瓶颈。

💡提示

如果您正在 Node.js 中编写 CPU/数据密集型应用程序，请立即停止。对于不同的任务，总有更好的工具。不要为了坚持单一语言或框架而限制自己。Node.js 在 I/O 密集型、事件驱动的工作负载中表现出色，但当涉及到重计算时，请考虑替代方案，如 Rust、Go、C++，甚至将工作卸载到专用服务。您不需要在所有地方都使用 Node.js。

了解这一点后，您的团队中的一位开发者可能会被诱惑通过切换到 `Buffer.allocUnsafe()` 来“优化”代码。而且在不理解后果的情况下，他们即将用性能问题换取一场安全灾难。

## `Buffer.allocUnsafe()`

这是让人们陷入麻烦的函数。`Buffer.allocUnsafe(size)` 是 `Buffer.alloc()` 的邪恶双胞胎。它们都向操作系统请求内存，但 `allocUnsafe` 完全跳过了第二步——零填充。它返回原始的、未修改的内存段。这使得它显著更快，因为它做的工作更少。

多快？我们稍后会看到硬数据，但对于大量分配，它可能是一个数量级或更多。这是那种让工程师感到聪明的性能提升。它也是那种导致安全漏洞的“聪明”，就像我们开头故事中的那样。

这里有一个更现实的场景。想象一个处理两个并发请求的 Web 服务器 -

**请求 A（用户 1）-**

```js
// Handler for /update-profile
function handleUpdate(req, res) {
 const userSession = { userId: 123, role: "admin", token: "..." };
 const sessionBuffer = Buffer.from(JSON.stringify(userSession));
 // ... do something with the sessionBuffer ...
}
```

**请求 B（用户 2）-**

```js
// Handler for /generate-report
function handleReport(req, res) {
 // Oops, developer tried to optimize.
 const reportBuffer = Buffer.allocUnsafe(1024);
 // They only write 500 bytes of report data.
 const reportData = generateReportData(); // returns 500 bytes
 reportData.copy(reportBuffer, 0);
 // The remaining 524 bytes are uninitialized!
 res.send(reportBuffer);
}
```

如果请求 B 紧接在请求 A 之后运行，用于`sessionBuffer`的内存槽位可能会（在罕见事件中）立即被重新用于`reportBuffer`。当用户 2 收到他们的报告时，它将包含 500 字节的有效数据，随后是 524 字节的内存中剩余的任何内容——这可能是用户 1 的管理会话令牌。你现在已经将管理凭证泄露给了普通用户。这是误用此 API 的直接、可预测的结果。

那么，何时使用`Buffer.allocUnsafe()`是可接受的？只有一个规则——**你必须确保在分配后立即写入缓冲区内存范围的每个字节。**

一个完美的例子是从文件中读取。

```js
const fs = require("node:fs");
 const fd = fs.openSync("script.js", "r");
const size = fs.fstatSync(fd).size;
 // SAFE. We know fs.readSync will fill the buffer completely.
const buf = Buffer.allocUnsafe(size);
const bytesRead = fs.readSync(fd, buf, 0, size, 0);
 // We've now overwritten the entire uninitialized buffer with file data.
```

ℹ️注意

如果你担心池的重复使用，你可以选择`allocUnsafeSlow()`作为替代方案，它永远不会使用内部池（不太可能基于池的重复使用）。

在这种情况下，我们分配一个缓冲区，在接下来的指令中，我们将其传递给一个系统调用（`fs.readSync`），该调用承诺从磁盘从头到尾填充数据。未初始化数据可能暴露的窗口极小，并且完全包含在这个单一操作中。这是有效、安全且高效的`allocUnsafe`使用。

如果你的代码在`allocUnsafe`调用和缓冲区被完全覆盖的点之间有任何逻辑——任何`if`语句、任何可能提前终止的循环、任何错误的可能性——你正在创建一个安全漏洞。这不是“是否”的问题，而是“何时”会烧毁你的问题。

## `Buffer.from()`

乍一看，`Buffer.from()`似乎是其中最有帮助的。它是一个通用的*工厂*函数，可以从你扔给它的几乎所有东西中创建一个缓冲区——一个字符串、一个数组、另一个缓冲区、一个`ArrayBuffer`。这种便利性是其最大的优势，也是其最危险的陷阱。与`alloc`和`allocUnsafe`不同，它们关于内存初始化，`Buffer.from()`关于数据解释和复制，其行为可能会对性能和数据完整性产生微妙和灾难性的后果。

让我们分解它的不同形式。

**`Buffer.from(string, [encoding])`**

这是最常见的用例。你有一个字符串，你想要它的二进制表示。

```js
const buf = Buffer.from("hello world", "utf8");
// <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
```

这看起来很简单，但并不是零成本操作。Node 必须遍历字符串并将字符转换为指定的编码。对于 UTF-8，这通常很快。但如果你在热路径上处理其他编码或非常大的字符串，这种转换可能会出现在 CPU 分析中。更重要的是，它分配了一个*新*缓冲区并将生成的字节复制到其中。这通常是你要的，但你需要意识到这是一个复制，而不是视图。

**`Buffer.from(buffer)`**

这也会创建一个副本。如果你将现有的缓冲区传递给`Buffer.from()`，它将分配一个大小相同的新缓冲区，并将源缓冲区的全部内容复制到新缓冲区中。

```js
const buf1 = Buffer.from("learn_node");
const buf2 = Buffer.from(buf1);
 buf2[0] = 0x6e; // 'n'
 console.log(buf1.toString()); // 'learn_node'
console.log(buf2.toString()); // 'nearn_node'
```

修改`buf2`不会影响`buf1`。这是安全且可预测的，但同样，要注意复制大型缓冲区的性能影响。

**`Buffer.from(array)`**

你可以从字节数组创建一个缓冲区。

```js
const buf = Buffer.from([0x48, 0x69, 0x21]); // 'Hi!'
```

这对于从常量构造缓冲区很有用，但对于大型数组来说速度较慢。Node 必须迭代 JavaScript 数组，检查每个元素，并将值复制到堆外缓冲区。这比直接处理缓冲区要低效得多。

**`Buffer.from(arrayBuffer)`**

这是其中最复杂的一个。

在 JavaScript 中，`ArrayBuffer`是一个原始的二进制数据对象。它们通常由浏览器 API（如`fetch`或`FileReader`）和一些 Node 库使用。关键区别在于`Buffer.from(arrayBuffer)`可以根据上下文创建一个与`ArrayBuffer`共享相同底层内存的*视图*，而不是一个副本。

想象一个文件上传服务。一个库给你上传的文件作为`ArrayBuffer`。你的代码需要处理前几个字节以检测文件类型，而应用程序的另一个部分需要扫描整个文件以查找病毒。

```js
// some-upload-library gives us an ArrayBuffer
const arrayBuffer = getUploadAsArrayBuffer();
 // You create a buffer to inspect the file header.
// This might NOT be a copy! It could share memory with arrayBuffer.
const headerBuffer = Buffer.from(arrayBuffer, 0, 16);
 // Meanwhile, another asynchronous function gets the same ArrayBuffer.
// This function sanitizes the data by overwriting certain byte patterns.
sanitizeFileInMemory(arrayBuffer);
```

你的`headerBuffer`一开始看起来是正确的。你读取了魔数并确定它是一个 JPEG 文件。但在你处理它的时候，`sanitizeFileInMemory`函数运行了。它直接修改了原始的`arrayBuffer`。因为你的`headerBuffer`只是对那个*相同内存*的一个视图，所以它的内容现在在你不知情的情况下被默默地改变了。

突然，你的文件类型检测逻辑间歇性失败。你本以为恒定不变的数据已经被应用程序的另一个完全不同的部分损坏了。这让人难以调试。没有错误，没有崩溃，只有不一致的结果。你可能要花几天时间追踪你的逻辑中的竞争条件，而根本原因是对`Buffer.from()`是在执行复制还是创建共享内存视图的理解错误。

让我们通过事件的顺序来了解这可能会引起的问题。

首先，代码对其文件头进行初始检查。它从`headerBuffer`中读取前几个字节，确认它是一个有效的 PNG 文件，并对自己感到满意。基于此，它决定启动一个异步操作，比如在继续处理图像之前，先在数据库中查找用户权限。

当你的代码等待数据库响应时，它释放了控制权（我们已经在之前的章节中学习了这一点）。JavaScript 事件循环会寻找其他可以执行的工作。它不会只是闲置。它看到队列中还有另一个任务等待，这是我们编写的名为`sanitizeFileInMemory`的安全函数。

```js
// This function runs while your other code is paused
sanitizeFileInMemory(arrayBuffer);
```

这是关键时刻。`sanitizeFileInMemory`函数被设计为扫描*整个文件*并清除任何潜在的恶意字节模式。它接收原始的`arrayBuffer`。它在第 10 个字节处发现了一些它不喜欢的东西，并用零覆盖了它。

因为`headerBuffer`只是一个指向相同内存的视图，所以它所查看的数据已经被从其下更改。没有警告，没有错误。内存被更改了，而`headerBuffer`只是该内存的一个窗口，其内容现在不同了。

几毫秒后，我们的数据库查询完成。你的原始函数醒来，准备完成其工作。现在它试图再次使用`headerBuffer`，可能用于读取图像尺寸。但字节 10 的数据不再是它预期的。从它的角度来看，标题现在已损坏。你的逻辑失败了，可能抛出一个奇怪的错误，或者可能只是产生垃圾数据。

这就是 bug。它只在你检查标题后但在完成使用它之前运行的微小时间窗口中发生。这是一个经典的竞态条件，其中你的程序的两个不同部分正在争夺使用和修改共享内存，结果取决于它们运行的精确顺序。这就是为什么它如此难以调试——当你尝试追踪它时，时间会改变，bug 就会消失。

常规规则——当你从外部源（尤其是作为`ArrayBuffer`）接收数据，并且需要确保其完整性进行操作时，使用`Buffer.alloc()`和`.copy()`创建一个显式的副本，而不是依赖于`Buffer.from()`的模糊行为。

```js
// The safe way to handle an ArrayBuffer you don't own.
const arrayBuffer = getUploadAsArrayBuffer();
 // 1\. Allocate a new, clean buffer that you control.
const headerBuffer = Buffer.alloc(16);
// 2\. Create a temporary view to copy from the source.
const sourceView = Buffer.from(arrayBuffer, 0, 16);
// 3\. Explicitly copy the data into your own buffer.
sourceView.copy(headerBuffer);
 // Now, headerBuffer is completely decoupled from the original arrayBuffer.
sanitizeFileInMemory(arrayBuffer); // This can't hurt you anymore.
```

## 缓冲区池和 8KB 的秘密

我们已经讨论了`allocUnsafe`如何给你带有“旧数据”的内存，但那些旧数据从何而来？是来自服务器上其他进程的随机垃圾数据？有时是。但更常见且更危险的是，它们来自**你自己的进程**。这是由于 Node.js 内部的一个性能优化，称为 Buffer 池。

持续向操作系统请求小块内存是不高效的。每次`malloc`调用都会产生一定的开销。为了加快速度，对于小于某个阈值的缓冲区，Node.js 不会逐个分配它们。相反，它会预先分配一个更大的、8KB 的内存块——即池。

当你调用`Buffer.allocUnsafe(100)`时，Node 不会向操作系统请求 100 字节。它会检查其内部的 8KB 池。如果有空间，它会从池中切出 100 字节并给你一个指向该切片的 Buffer。当你的 Buffer 被垃圾回收时，那 100 字节的切片不会返回给操作系统——它只是在池中标记为可用。

这是一个巨大的性能提升。它使得分配小块 Buffer 变得非常快。`Buffer.allocUnsafe()`和`Buffer.from()`都使用这个池进行小分配。`Buffer.alloc()`也可以使用它，但由于它必须用零填充内存，性能优势更多地与重用无关，而是与避免`malloc`开销有关。

现在，将这连接到安全影响。

在`allocUnsafe`缓冲区中找到的数据很可能来自另一个 Buffer，来自您自己的应用，这是最近使用并丢弃的数据。8KB 池是最近使用敏感信息的温床。

让我们用这个知识回顾我们的 JWT 泄露场景。

1.  **请求 1**为用户 A 到来。您的代码创建一个 200 字节的 Buffer 来保存他们的会话数据。这个缓冲区是从 8KB 内部池中切分的。

1.  请求完成。会话缓冲区不再被引用，并符合垃圾回收的条件。它在池中的 200 字节切片现在被认为是“空闲”的。数据（JWT）仍然在那里。

1.  **请求 2**在几毫秒后为用户 B 到来。您的代码调用`Buffer.allocUnsafe(500)`。

1.  Node 看到这小于 8KB，就转向池中。它找到一个空闲槽位——可能是来自请求 1 的 200 字节切片，以及旁边的 300 字节——然后将其分配给您。

1.  您的`allocUnsafe`缓冲区现在包含的前 200 字节是用户 A 的完整会话数据。

这不是理论风险。这是您的应用将自身秘密泄露给自身的机制。池将您的应用内存空间变成一个微小、高速的数据回收生态系统。使用`allocUnsafe`就像在没有过滤的情况下从回收系统中喝水。

默认池大小是 8KB（`Buffer.poolSize`）。您可以更改它，但您不应该。更改它是一个信号，表明您正在尝试微优化您可能不完全理解的东西。合理的默认值存在是有原因的。

要点很简单。缓冲区池使得小型、不安全的分配更加危险，因为它增加了您得到“未初始化”的数据不是随机噪声，而是来自您自己应用另一部分高度敏感、结构化数据的概率。

## 让我们来衡量一下性能

让我们用一些硬性数字来支持这些说法。`alloc`和`allocUnsafe`之间的性能差异并不细微，而是一个悬崖。

我们将运行一个简单的基准测试。分配一个特定大小的缓冲区 10,000 次，并测量所需时间。这些基准测试的代码可以在`examples/buffer-allocation-patterns/benchmark.js`中找到。

```js
const { performance } = require("node:perf_hooks");
 const ITERATIONS = 10000;
 /**
 * A helper function to run and time a specific buffer allocation method.
 * @param {string} name - The name of the benchmark to display.
 * @param {number} size - The size of the buffer to allocate.
 * @param {(size: number) => Buffer} allocFn - The allocation function to benchmark.
 */
function benchmark(name, size, allocFn) {
 const start = performance.now();
 for (let i = 0; i < ITERATIONS; i++) {
 allocFn(size);
 }
 const end = performance.now();
 console.log(`- ${name}(${size}) x ${ITERATIONS}: ${(end - start).toFixed(2)}ms`);
}
 console.log("--- Benchmarking Buffer Allocation ---");
console.log(`(Iterations: ${ITERATIONS}, Node.js: ${process.version})`);
 // --- Scenario 1: Small allocations that use the internal buffer pool ---
console.log("\nScenario 1: Small Allocations (100 bytes, pooled)");
benchmark("Buffer.alloc", 100, (s) => Buffer.alloc(s));
benchmark("Buffer.allocUnsafe", 100, (s) => Buffer.allocUnsafe(s));
 // --- Scenario 2: Medium allocations just above the pool size ---
console.log("\nScenario 2: Medium Allocations (10KB, non-pooled)");
const mediumSize = 10 * 1024;
benchmark("Buffer.alloc", mediumSize, (s) => Buffer.alloc(s));
benchmark("Buffer.allocUnsafe", mediumSize, (s) => Buffer.allocUnsafe(s));
 // --- Scenario 3: Large allocations where zero-filling is very expensive ---
console.log("\nScenario 3: Large Allocations (1MB, non-pooled)");
const largeSize = 1024 * 1024;
benchmark("Buffer.alloc", largeSize, (s) => Buffer.alloc(s));
benchmark("Buffer.allocUnsafe", largeSize, (s) => Buffer.allocUnsafe(s));
 console.log("\n--- Benchmarking Buffer.from ---");
 const largeString = "a".repeat(largeSize);
const existingLargeBuffer = Buffer.alloc(largeSize);
 let start = performance.now();
Buffer.from(largeString, "utf8");
let end = performance.now();
console.log(`- Buffer.from(1MB string): ${(end - start).toFixed(2)}ms`);
 start = performance.now();
Buffer.from(existingLargeBuffer);
end = performance.now();
console.log(`- Buffer.from(1MB buffer, copy): ${(end - start).toFixed(2)}ms`);
```

### 场景 1 - 小型分配（100 字节）

这是缓冲区池活跃的情况。

```js
- Buffer.alloc(100) x 10000: 3.11ms
- Buffer.allocUnsafe(100) x 10000: 1.23ms
```

在这里，`allocUnsafe`大约**快了 2.5 倍**。填充 100 字节的成本很小，但重复 10,000 次，就会累积起来。`allocUnsafe`只是从预分配的池中抓取一个切片，这非常快。

### 场景 2 - 中型分配（10KB）

这刚刚超过默认的 8KB 池大小，因此每次分配都必须转到操作系统。

```js
- Buffer.alloc(10240) x 10000: 9.65ms
- Buffer.allocUnsafe(10240) x 10000: 12.84ms
```

有趣的是，在这种情况下，`allocUnsafe`在这个系统上实际上**慢了 1.3 倍**。这里的开销是`malloc`调用本身和填充 10KB 内存所花费的时间的混合。

### 场景 3 - 大型分配（1MB）

这是您处理文件上传、视频流或其他大型二进制数据的地方。

```js
- Buffer.alloc(1048576) x 10000: 1151.15ms
- Buffer.allocUnsafe(1048576) x 10000: 988.47ms
```

现在看看这个。`Buffer.allocUnsafe`是**1.2 倍更快**。这是一个明显的性能差异，尽管在某些系统上不如之前那么戏剧化。向操作系统请求 1MB 内存的成本仍然被写入所有 1,048,576 个字节的零的成本所淹没。1151ms 是在仅分配内存上花费的大量时间。如果这是用户请求的路径，你只是因为内存初始化而添加了显著的延迟。

当性能分析器告诉你你在`Buffer.alloc`上花费了 80%的 CPU 时间，即使 1.2 倍的速度提升也可能很有吸引力。这感觉像是免费的性能。但正如我们已经建立的，成本不是以 CPU 周期来支付的；它是支付在安全风险上。

### `Buffer.from()` 性能

那么`Buffer.from()`呢？它的性能完全取决于源。

```js
- Buffer.from(1MB string): 1.42ms
- Buffer.from(1MB buffer, copy): 0.24ms
```

从 1MB 字符串创建一个 1MB 缓冲区大约需要**1.42ms**。这是 UTF-8 编码和复制的成本。

复制现有的 1MB 缓冲区只需**0.24ms**。这是一个高度优化的`memcpy`操作。它非常快，但如果你在循环中这样做，仍然是一个需要意识到的成本。

这些数字为你提供了一个心理模型来做出决策。你的分配大小小吗？性能差异可能微不足道。大吗？差异巨大，你需要仔细思考。分配在每秒运行数千次的“热点路径”上吗？即使是小的差异也可能累积起来。唯一确定的方法是在**实际负载下对应用程序进行性能分析**。不要猜测。不要过早优化。测量，识别瓶颈，然后使用这些数字来理解你解决方案的权衡。

## 安全影响和攻击向量

我们已经关注了最明显的安全漏洞——通过`allocUnsafe`未初始化的内存泄露敏感数据。但影响比这种灾难性的失败模式更广泛、更微妙。让我们像攻击者一样思考。

### 直接信息泄露（显而易见的一种）

这是我们已经广泛讨论过的`allocUnsafe`场景。攻击者收到一个响应、一个文件或触发一个包含来自其他用户或系统本身的数据的错误日志。这些数据可以包括 -

+   会话令牌、API 密钥、JWT

+   在传输中的密码、密码散列或盐

+   数据库凭据或连接字符串

+   PII（个人可识别信息）如姓名、电子邮件、地址

+   加密密钥

+   TLS 证书或私钥片段

关键漏洞是任何调用`Buffer.allocUnsafe(size)`的地方，后续逻辑未能覆盖**整个**缓冲区。这可能是由于错误的尺寸计算、早期的`return`错误路径，或者乐观的`try...catch`块没有正确处理部分填充的缓冲区。

### 泄露加密材料

这是信息泄露的一个特别恶劣的子集。如果您的应用程序处理加密或解密，密钥、nonce 或明文/密文数据将在内存中的缓冲区中存在一段时间。缓冲池使得如果为了普通目的（如构建 JSON 响应）使用`allocUnsafe`分配缓冲区，它可能包含用于在另一个请求中较早时刻签名令牌的私钥的残留部分。如果攻击者可以重复触发这种不安全分配，他们可能能够拼凑足够的泄露片段，从而危害您的整个加密基础设施。

### 通过`Buffer.from()`进行拒绝服务（DoS）

这是一个更为微妙的攻击。想象一个接受 JSON 有效负载的 API 端点，其中一个字段预期是一个 base64 编码的字符串，然后您将其转换为缓冲区。

```js
// Attacker sends: { "data": "very...long...string" }
const body = JSON.parse(req.body);
// The server decodes and allocates based on attacker input.
const dataBuffer = Buffer.from(body.data, "base64");
```

使用字符串输入的`Buffer.from()`调用根据字符串的*解码*大小分配一个新的缓冲区。攻击者可以发送一个相对较小的有效负载，解码后扩展成巨大的缓冲区。几个这样的请求可能会耗尽服务器的内存，导致其崩溃或对合法流量无响应。虽然这是一个通用应用层 DoS 向量，但如果您在尝试从它分配缓冲区之前不对输入字符串长度施加严格限制，`Buffer.from`的便利性可能会使其成为一个容易引入的漏洞。

### 时间攻击

这更偏向于理论，且不太可能实现——但在特定的加密环境中，可能性并非为零。`Buffer.alloc()`完成所需的时间与缓冲区的大小成正比，因为它必须将其清零。`Buffer.allocUnsafe()`对于所有池化大小都花费大约恒定（且非常短）的时间。如果攻击者可以影响分配的大小并精确测量服务器的响应时间，他们可能能够推断信息。例如，如果缓冲区的大小取决于秘密的长度，`alloc`和`allocUnsafe`之间的分配时间差异可能会泄露有关该**长度**的信息。这是一个高级攻击向量，但它突出了即使是性能特征也可能有安全后果。

ℹ️注意

**实际上**，攻击者会面临噪声（操作系统调度、网络延迟、CPU 竞争）。这是一个值得在加密代码中提及的先进/边缘情况向量，但并不是针对典型 Web 应用的常见实际攻击手段。加入此内容是为了让您意识到这种攻击向量存在。

你的主要防御措施很简单。**将来自 `allocUnsafe` 缓冲区的任何数据视为不可信且可能具有放射性的，直到你亲自覆盖它为止。**代码审查必须对此毫不留情。任何使用 `allocUnsafe` 的代码都需要被质疑——"你能否证明，在所有可能的代码路径和错误条件下，这个整个缓冲区在读取或发送到任何地方之前都被覆盖了？" 如果答案不是自信且明显的 "是"，则必须重构为 `Buffer.alloc()`。

ℹ️注意

对于来自 Node 内部池（slab）的小分配，allocUnsafe()非常快（本质上是一个 O(1)的切片）。然而，（a）如果池耗尽，则需要一个新的 slab 或操作系统分配，成本增加，并且（b）allocUnsafeSlow()不使用池。所需的时间并不是“对所有大小都是恒定的”，而是对于池中小分配非常快（O(1)），但在需要新的 slab 或操作系统分配时行为会改变。

## 内存碎片化和 GC 压力

📌重要

让我帮你避免我看过太多开发者犯的错误。如果你发现自己一直在与 `Buffer.allocUnsafe()` 作斗争以获得性能，进行大量的二进制操作，或者运行 CPU 密集型数据处理，你很可能使用了错误的工具。Node.js 在处理数千个并发 I/O 操作、管理网络连接和编排服务方面非常出色。但是，它在单线程的事件循环中运行 JavaScript，这意味着 CPU 密集型工作会阻塞其他所有操作。

当你处理视频流、进行实时图像处理、运行压缩算法或在大型数据集上执行加密操作时，像 Rust、Go 或 C++这样的语言将为你提供更好的服务。这些语言提供了直接内存控制，没有垃圾回收暂停，跨多个 CPU 核心的真正并行性，以及编译为高度优化的机器代码的无成本抽象。一个让 Node.js 哭泣的视频转码操作在 Rust 中可以平稳运行。在 Node.js 中需要仔细缓冲区管理的数据处理管道在 Go 中变得简单，因为 Go 具有出色的并发原语。

这就是使优秀工程师变得伟大的原因——他们为每一项工作选择合适的工具。不要将自己限制在单一的语言上，学习多种语言。你可以绝对地从 Node.js 中调用 Rust 代码，使用原生插件或 WebAssembly，让 Node.js 专注于它最擅长的事情（处理 HTTP 请求、管理业务逻辑），同时将重计算委托给为它而构建的语言。我看到过团队通过将热点路径移动到 Rust（同时保持 API 层在 Node.js 上）将处理时间从分钟缩短到秒。不要让语言忠诚度使你的应用程序变得更差。你的用户不在乎你的整个堆栈是否是 JavaScript；他们关心的是你的服务是否快速且可靠。

我们到目前为止的讨论主要集中在 CPU 性能和安全上。但分配模式也对内存使用和垃圾收集器（GC）的行为有深远的影响。

### 垃圾收集器压力

你创建的每个 Buffer 对象，无论其 off-heap 存储有多大，都有一个对应的小对象，它存在于 V8 堆上。当你每秒创建和丢弃数千个 Buffer 时，你正在为 V8 垃圾收集器制造麻烦。垃圾收集器必须跟踪所有这些小堆对象，确定它们何时不再可达，并清理它们。

这对于`Buffer`本身来说是一个相对较小的问题，但它与一个更大的问题有关——临时复制。考虑在流解析器中这个常见的模式。

```js
let internalBuffer = Buffer.alloc(0);
 function handleData(chunk) {
 internalBuffer = Buffer.concat([internalBuffer, chunk]);
 // ... try to parse messages from internalBuffer ...
}
```

`Buffer.concat`很方便，但看看它做了什么。它分配了一个足够大的新缓冲区来容纳`internalBuffer`和`chunk`，将数据从两者复制到新缓冲区中，然后丢弃旧的缓冲区。如果你接收 100 个小块来形成一个消息，你刚刚进行了 100 次分配和 99 次复制操作，创建了 99 个中间缓冲区并立即丢弃它们。这给 GC 带来了巨大的压力，并浪费了 CPU 周期在复制数据上。更好的方法是将单个较大的缓冲区和指针进行管理，但这将是另一章的主题。重点是，你的分配*策略*（而不仅仅是分配函数）有巨大的影响。

### 内存碎片化

当处理不适用于池化的大型缓冲区（即，> 8KB）时，这是一个更大的问题。当你的应用程序长时间运行并频繁地分配和释放不同大小的缓冲区时，可能会导致内存碎片化。

让我们想象进程的可用内存就像一个长长的空箱子队列。

1.  你分配了一个 1MB 的缓冲区（块 A）。

1.  你分配了一个 2MB 的缓冲区（块 B）。

1.  你又分配了一个 1MB 的缓冲区（块 C）。现在，你的队列有 `[A:1MB][B:2MB][C:1MB]`。

1.  现在，你释放了中间的 2MB 缓冲区（块 B）。你的架子看起来像 `[A:1MB][---EMPTY:2MB---][C:1MB]`。

你有 2MB 的空闲内存。但如果你的下一个请求是分配一个*3MB*的缓冲区，分配将失败（或者 Node 将不得不从操作系统请求更多内存）。尽管你总共有足够的内存，但它不是*连续的*。你有一个 2MB 的空洞。这就是碎片化。

随着时间的推移，长时间运行的 Node 进程可以积累许多这样的空洞，即使活动内存（`heapUsed` + `external`）保持稳定，也会导致整体内存使用（`rss`或 Resident Set Size）增加。这是因为 C++内存分配器（`malloc`）很难有效地重用这些碎片化的间隙。

分配模式是如何影响这个的？

**频繁的`Buffer.alloc(largeSize)`**是碎片化的主要驱动因素。不断创建和销毁大型、可变大小的缓冲区是最坏的情况。

**缓冲池**是针对小型分配的直接防御碎片的方法。通过为所有小型缓冲区重用单个大型内存块，Node 避免了在内存空间中散布成千上万的微小分配和释放操作。这是其最重要的但最不被欣赏的好处之一。

如果你的服务处理大型二进制块，并且你看到其内存占用（`rss`）随着时间的推移而增长，而没有相应的 `heapUsed` 或 `external` 增加时，你可能成为了内存碎片化的受害者。解决方案通常是转向更复杂的内存管理策略，如启动时分配几个非常大的“区域”缓冲区，并在其中自行管理内存，而不是不断向 Node 请求新的大型缓冲区。这是一种高级技术，但这是在默认分配模式在极端规模下崩溃时的逻辑结论。

## 平台差异和分配器行为

虽然 Node.js 在底层操作系统之上提供了一个出色的抽象，但重要的是要记住它并非孤立存在。你所观察到的行为，尤其是在性能和未初始化内存方面，可能会受到你所运行的平台潜移默化的影响。

Node.js 最终调用来从操作系统获取内存的函数通常是 `malloc` 或其变体。`malloc` 的实现可能因操作系统（如 Linux、macOS、Windows）而异，甚至在同一操作系统上的不同 C 标准库实现之间也有所不同（如 `glibc`、`musl`、`jemalloc`）。

**但是...这对您意味着什么？**

你在 `Buffer.allocUnsafe` 缓冲区中看到的内容高度依赖于操作系统和分配器的策略。一些分配器在请求大块内存时可能会更有可能从操作系统提供新零内存，而其他分配器可能会更积极地回收你进程中的内存。安全风险始终存在，但你可能泄露的*具体数据*可能会从开发者的 macOS 机器到生产 Linux（Alpine）容器而发生变化。永远不要假设因为你没有在测试环境中看到敏感数据，漏洞就不存在。生产环境的行为将不同。

虽然 `alloc`（慢速）和 `allocUnsafe`（快速）之间的相对差异始终存在，但绝对数值可能会有所不同。像 `jemalloc`（在高性能应用中很受欢迎）这样的分配器，对多线程分配和减少碎片进行了大量优化。与使用系统默认的 `glibc` `malloc` 相比，针对 `jemalloc` 编译的 Node.js 二进制文件在处理大量分配工作负载时可能会显示出不同的性能特征。这通常属于微优化范畴，但对于超大规模服务来说，这可能会很重要。

Node 的内部缓冲池位于系统分配器之上。它通过 `malloc` 请求其 8KB 块。池本身的效率在各个平台上是一致的，但整个系统如何处理 Node 对这些块的需求可能会有所不同。

这里的关键要点不是您需要成为系统内存分配器的专家。这是关于保持一种健康的偏执感。您开发机器上的方便、可预测的环境并不是您的生产环境的完美复制品。与内存布局相关的安全漏洞可能更难在本地重现，这使得它更加危险，因为它可以在生产流量模式中潜伏。

这加强了核心原则——**编写防御性代码**。不要依赖于您机器上 `allocUnsafe` 的观察行为。只依赖于其文档化的合同——它返回未初始化的内存，您负责清除它。这个合同在所有平台上都成立，即使内存变化的幽灵。您的代码应该足够健壮，能够在底层分配器的实现细节未知的情况下保持正确。

## 生产决策框架

您已经看到了危险、性能悬崖和微妙的复杂性。现在，您如何在日常工作中做出正确的选择？当您即将输入 `Buffer.` 时，您应该选择哪个函数？

这是一个简单、安全的决策框架，您可以遵循。把它想象成一个心理流程图。

**首先，从 `Default` 开始**

您的默认、本能的选择应该是 `Buffer.alloc()`。

**问题：您是否正在分配缓冲区？**

**回答：使用 `Buffer.alloc(size)`。**

不要考虑性能。不要考虑微优化。您的主要目标是正确性和安全性。`Buffer.alloc()` 提供了这两者。对于绝大多数应用程序代码（解析请求、构建响应、与数据库交互），零填充的性能成本非常小，以至于它永远不会成为您的瓶颈。网络延迟、磁盘 I/O、数据库查询时间，甚至您自己的业务逻辑都会慢得多。在这些上下文中使用 `allocUnsafe` 是一个典型的过早优化——所有邪恶的根源。

**接下来，等待 `Evidence`**

除非您有具体、不可否认的证据表明缓冲区分配是性能瓶颈，否则不要偏离第一步。

**问题：这种证据看起来像什么？**

**回答：一个 CPU 配置文件（来自像 `0x`、Node 内置的剖析器或生产 APM 工具这样的工具），它清楚地显示在 `Buffer.alloc()` 函数调用中花费了大量的时间，尤其是在您考虑更改的代码行上。**

如果您没有这个配置文件，您将不被允许继续。猜测、感觉或“我认为这可能更快”都不是证据。

**最后，如果 `alloc()` 是已证实的瓶颈，考虑使用 `allocUnsafe()`**

你有一个配置文件。`Buffer.alloc()` 像圣诞树一样闪烁，导致你的服务错过服务级别协议（SLA）。现在，只有现在，你才能 *考虑* 使用 `Buffer.allocUnsafe()`。为此，你必须能够对以下问题回答“是” -

**问题：** 在我调用 `Buffer.allocUnsafe(size)` 之后，我的下一个操作将 **无条件且完全** 覆写该缓冲区的每个字节，从索引 0 到 `size - 1` 吗？

“无条件”意味着没有 `if` 分支，没有 `try...catch` 块，没有可能提前退出的循环，这会允许缓冲区的任何部分在完全覆盖之前被使用。

+   **良好候选者 -** `fs.readSync(fd, buf, ...)` 其中你读取缓冲区的完整大小。操作系统保证覆盖。

+   **良好候选者 -** 在分配后立即使用 `buf.fill(someValue)`。你正在明确地覆盖内存。

+   **不良候选者 -** 你分配了一个 1024 字节的缓冲区，然后有一个循环，可能只写入 500 字节，这取决于输入。这是一个等待发生的安全漏洞。

+   **不良候选者 -** 你分配了一个缓冲区，然后进入一个 `try...catch` 块来填充它。如果在中间抛出错误，捕获块可能会记录或暴露部分填充的缓冲区。

如果你无法满足这个严格的要求，但仍然存在性能问题，`allocUnsafe` 不是解决方案。你的问题出在其他地方。你可能需要重新思考你的算法，以避免完全分配，例如通过更有效地使用流或预分配单个较大的工作缓冲区。

**关于 `Buffer.from()` -**

决定基于你的源数据。

**问题：** 你试图做什么？

**如果需要从字符串、数组或其他缓冲区创建缓冲区？** 使用 `Buffer.from()`。注意转码/复制的性能成本。

**如果从 `ArrayBuffer` 或其他你无法控制的内存创建缓冲区？** 非常小心。如果你需要保证缓冲区的内容不会改变，请使用 `Buffer.alloc()` 和 `source.copy(destination)` 明确复制。不要相信 `Buffer.from()` 会为你复制。假设它可能创建一个共享内存视图，并防御性地编写代码。

这个框架优先考虑安全性，只有在数据证明合理且可以证明是安全的情况下才允许性能优化。遵循它将防止你可能遇到的 99%的与缓冲区相关的灾难。

## 迁移模式和安全默认值

假设你继承了遗留代码库，或者你刚刚阅读了这一章，正大汗淋漓。你该如何找到并修复这些问题？

你的第一步是审计代码库中的危险模式。简单的 `grep` 或你的文本编辑器的全局搜索是你的最佳朋友。

首先，**搜索 `new Buffer()`**。这是旧的、已弃用的构造函数。它臭名昭著地不安全，其行为取决于参数。其行为类似于 `allocUnsafe` 和 `from` 的混合。每个 `new Buffer()` 的实例都必须被移除。这不是一个“是否”的问题，这是一个关键的安全漏洞。大多数 Node.js 环境甚至会对这个发出运行时弃用警告。

然后，**搜索 `Buffer.allocUnsafe`**。这是你的主要目标。对于每个结果，你必须应用上一节中的生产决策框架。

+   是否有证明其使用的分析器输出？可能没有。

+   如果是，它是否随后是一个无条件、完整的覆盖？

+   如果这两个问题的答案都是否定的，它需要被替换。

迁移路径通常是直接的。

**`new Buffer(number)`** -> **`Buffer.alloc(number)`** 是最常见且关键的修复。旧的构造函数在传递一个数字时，并没有对内存进行零填充。现代、安全的等效方法是 `Buffer.alloc()`。

```js
// BEFORE - Leaks uninitialized memory.
const unsafeBuf = new Buffer(1024);
 // AFTER - Safe, zero-filled buffer.
const safeBuf = Buffer.alloc(1024);
```

如果无法证明 `allocUnsafe` 调用是安全的，从 **`Buffer.allocUnsafe(size)`** 到 **`Buffer.alloc(size)`** 的修复就是简单地切换到其安全的对应版本。是的，这可能会影响性能。这是安全性的代价。如果性能回归是不可接受的，这意味着你需要重构那部分代码以减少分配的重量，而不是你应该坚持使用不安全的版本。

从 **`new Buffer(string)`** 到 **`Buffer.from(string)`**，因为旧的构造函数也可以接受一个字符串。

```js
// BEFORE - Deprecated and less explicit.
const oldWay = new Buffer("hello", "utf8");
 // AFTER - Modern, clear, and correct.
const newWay = Buffer.from("hello", "utf8");
```

审计一次是好的，但预防新问题是更好的。你应该使用代码检查器自动执行这些规则。`eslint-plugin-node` 有几个对于这一点至关重要的规则。

规则 **`node/no-deprecated-api`** 会自动标记 `new Buffer()` 的使用，防止任何人重新引入它。

为了更高级的保护，你可以编写一个自定义的 ESLint 规则，标记所有 `Buffer.allocUnsafe` 的使用。然后，你可以在少数经过仔细审查、其使用是合理的行上使用 `// eslint-disable-next-line` 注释。这迫使每个使用它的开发者明确承认风险，并提供一个注释解释为什么他们的用例是安全的。这使得不安全代码在代码审查中突出，这正是你想要的。

存在工具和模式可以使安全的方式变得简单。使用它们。

## 回顾最佳实践

让我们将我们讨论的所有内容提炼成一套清晰、可操作的指南。这是你在处理 Node.js 中的二进制数据时应该牢记在心的速查表。

+   **始终默认使用 `Buffer.alloc()`**。让它成为肌肉记忆。这是最重要的实践。它是可预测的、安全的，并且其性能对于绝大多数用例来说已经足够。

+   **除非你有 CPU 性能分析证明`Buffer.alloc()`是你的瓶颈，否则永远不要使用`Buffer.allocUnsafe()`。** 不要猜测。不要假设。测量。如果你没有性能分析，你就不存在需要不安全解决方案的问题。

+   **如果你*必须*使用`Buffer.allocUnsafe()`，你必须保证立即、同步和完整地覆盖整个缓冲区。** 任何允许在缓冲区完全填满之前读取或使用缓冲区的代码路径都是安全漏洞。在代码审查中仔细检查这些位置。

+   **立即删除并替换所有已弃用的`new Buffer()`构造函数实例。** 它是不安全的，已被`Buffer.alloc()`和`Buffer.from()`取代。这是不可协商的。

+   **对`Buffer.from()`与`ArrayBuffer`持怀疑态度。** 当从外部来源接收`ArrayBuffer`时，假设它创建了一个共享内存视图。如果你需要一个稳定、不可变的副本，请使用`Buffer.alloc(size)`和`.copy()`显式创建。

+   **检查不安全模式。** 使用 ESLint 插件如`eslint-plugin-node`自动禁止`new Buffer()`。考虑创建一个自定义规则来标记`Buffer.allocUnsafe`，迫使开发者证明其使用理由。

+   **避免在热点路径中采用多话分配模式。** 在紧密循环中创建许多小型、短暂存在的缓冲区（例如，重复使用`Buffer.concat`）可能会使垃圾收集器过载并损害性能。寻找使用流或预分配单个较大缓冲区的方法以减少分配波动。

+   **注释危险代码。** 如果你有一个合法的、经过基准测试证明的理由使用`Buffer.allocUnsafe`，留下一个详细的注释解释*为什么*。链接到基准数据或分析器输出。下一个开发者（可能六个月后就是你）需要理解风险和理由。

```js
// A GOOD comment that explains the reason
//
// Using allocUnsafe here because profiling showed Buffer.alloc
// was consuming 30% of CPU under load. See [link-to-profiler-results].
// The buffer is immediately and completely overwritten by the contents
// of the file read by fs.readSync, mitigating the security risk.
const buf = Buffer.allocUnsafe(fileSize);
fs.readSync(fd, buf, 0, fileSize, 0);
```

## 结束

在选择`alloc`、`allocUnsafe`和`from`之间，是关于理解每个函数提供的具体合约并将其与你的代码的具体需求相匹配，其中对安全性的重视程度很高。`allocUnsafe`的速度是一个强大的诱惑，但失败的成本是灾难性的数据泄露。

你现在拥有了知识和框架来负责任地做出这些决定。你理解了内存架构，你看到了现实世界的测量结果，你也感受到了做错事的直观风险。勇敢地去构建令人惊叹的事物，但请确保安全。分析你的代码，在代码审查中无情的批判，并且始终，始终默认选择安全路径。
