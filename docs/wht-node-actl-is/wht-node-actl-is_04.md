# 事件循环解析

> [`www.thenodebook.com/node-arch/event-loop-intro`](https://www.thenodebook.com/node-arch/event-loop-intro)

## 单线程模型

如果你在这里，你可能之前写过一些 Node.js 代码。你生活在异步的世界里，呼吸着异步的气息。你可能在梦中都在写 `Promise.then()` 链，并且使用 `async/await` 的流畅度让其他人羡慕。你知道——深入骨髓——你绝对不能阻塞主线程。你明白编写非阻塞代码的**意义**。

这章是关于原因和方法的。我们将打开你习惯的“黑盒”，并仔细观察使这一切发生的酷炫、复杂的机械装置。

这不是一本入门指南。远非如此。这是一本针对准备从“它只是工作”的直觉过渡到精确、机械的 Node.js 运行时模型的实践工程师的深入指南。我们将分解 Node 如何同时处理许多事情，超越简单的解释，看看它真正是如何工作的。

Node.js 的核心悖论，让人们夜不能寐的问题是这样的：它如何在单线程上处理**数以万计**的并发连接？一个没有并行执行 JavaScript 的单个工作器，达到这样的规模？这听起来不可能。答案是**事件循环**。在我们完成这里之后，你不仅会知道它的名字；你还会理解它的阶段、优先级以及它是如何与其他 Node 运行时核心部分交流的。你将能够自信地预测任何异步代码的执行顺序，并追踪那些曾经像机器中的幽灵一样的性能问题。

让我们动手实践一下。

### 调用栈

在我们甚至能说出“异步”这个词之前，我们必须先熟悉它的对立面。所有 JavaScript 执行的基础是同步调用栈。把它想象成一堆盘子。每次调用一个函数，你就在栈顶放置一个新的盘子——一个“帧”。这个帧包含了所有函数的参数和局部变量。

调用栈是一个经典的先进后出（LIFO）结构。你最后放置的盘子是第一个被取下的。当一个函数完成其工作并返回时，它的调用帧从栈顶弹出。

让我们追踪一下这极其简单的同步代码：

```js
function third() {
 console.log("Three");
}
 function second() {
 console.log("Two");
 third();
}
 function first() {
 console.log("One");
 second();
 console.log("Done with first");
}
 first();
```

这会输出：

```js
One
Two
Three
Done with first
```

这里是如何工作的——

1.  `first()` 被调用。它的调用帧被推入栈中。`[first]`

1.  `first()` 输出 "One"。很简单。

1.  `first()` 调用 `second()`。`second()` 的调用帧直接放在上面。`[first, second]`

1.  `second()` 输出 "Two"。

1.  `second()` 调用 `third()`。`third()` 的调用帧被添加到不断增长的塔中。`[first, second, third]`

1.  `third()` 输出 "Three"。它已经完成了所有工作，所以它返回。它的调用帧被弹出。`[first, second]`

1.  控制权返回到 `second()`。它没有其他事情要做，所以它返回。它的调用帧被弹出。`[first]`

1.  我们回到了`first()`。它记录了“完成 first”，运行完所有行后返回。它的帧被弹出。`[]`

栈现在为空。脚本已完成。这是你所有 JavaScript 的唯一工作空间。只有一个调用栈，一次只能做一件事：栈顶上的任何内容。

### “阻塞”真正意味着什么

“阻塞”不是一个模糊的、抽象的概念。它是一个单一调用栈的直接、残酷的后果。说“阻塞事件循环”只是个花哨的说法，意思是你在调用栈上放置了一个拒绝离开的函数。它就那样坐着，永远完成不了它的工作。

而当那个函数的帧占据了栈顶，没有任何其他东西——没有任何东西——可以运行。整个过程被绑架了。

让我们看看一个更现实的例子：一个 CPU 密集型的加密操作。

```js
const crypto = require("node:crypto");
 function hashContinuously() {
 console.log("Starting a blocking operation...");
 const startTime = Date.now();
 let salt = "some-salt";
 // This loop will run for 5 seconds, monopolizing the call stack.
 while (Date.now() - startTime < 5000) {
 salt = crypto.pbkdf2Sync("password", salt, 1000, 64, "sha512").toString("hex");
 }
 console.log("...Blocking operation finished.");
}
 setTimeout(() => {
 console.log("This timer will be delayed by 5 seconds!");
}, 1000);
 hashContinuously();
```

运行这个脚本，你会看到一些有趣的事情。应该在秒后触发的`setTimeout`回调，却只在“阻塞操作完成”之后出现。这是怎么回事？

1.  我们调用`setTimeout`。Node 的 API 愉快地安排了一个 1000 毫秒后触发的定时器。这是一个非阻塞、一触即发的操作。

1.  `hashContinuously()`被推入调用栈。

1.  “地狱般的”`while`循环开始了。在五秒钟的痛苦中，V8 引擎完全被消耗在反复哈希一个值。`hashContinuously`帧就静静地坐在栈顶。

1.  在 1 秒的标记处，定时器耐心地“触发”。这意味着它的回调函数被放入队列中，准备并等待执行。

1.  **但是这里有个问题**：事件循环无法触及这个队列。为什么？因为调用栈还没有空！它仍然卡在`hashContinuously()`上。事件循环实际上被冻结了，就像在玻璃上敲击，等待栈清空。

1.  最后，五秒后，`while`循环结束。`hashContinuously()`记录了它的最后一条消息并返回。它的帧幸运地从栈中弹出。

1.  最后，调用栈为空。事件循环苏醒过来，抓取等待的定时器回调，将其推入栈中，并执行它。

这是栈的独裁统治。一个慢速函数可以让你整个应用瞬间停止。这正是 Node 的整个异步架构旨在解决的问题。所以，如果调用栈是瓶颈，解决方案从何而来？为了回答这个问题，我们必须揭开盖子，超越 JavaScript 本身。

## V8、Libuv 和 Node.js 绑定

就像你在上一章中读到的[Inside the v8 Engine](https://thenodebook.com/node-arch/v8-engine-intro)，我们所说的“Node.js 运行时”并不是一个单独的程序。它更像是一个超级组合——一些强大的技术完美地一起工作。理解它们的独特角色非常重要。

### **V8：JavaScript 执行引擎**

ℹ️注意

我们在前一章中深入探讨了[v8](https://thenodebook.com/node-arch/v8-engine-intro)，并且还将有一个专门的课程，包含六个章节（21.1–21.6）。尽管如此，我假设一些读者会直接跳到这一章，所以为了清晰起见，我将提供简要的解释。

V8 是 Google 的传奇开源 JavaScript 引擎，用 C++ 编写。如果 Node 是一辆车，V8 就是引擎（因此得名）。它是实际执行你的 JavaScript 代码的部分。其主要工作如下 -

+   V8 捕获你的原始 JavaScript，并通过一个极其聪明的即时编译（JIT）过程，将其转换为高度优化的本地机器代码。这就是为什么现代 JavaScript 运行如此之快。

+   就像我们所看到的，V8 是单一线程栈的严格管理者，负责推入和弹出帧。

+   V8 在一个称为堆的地方处理你的对象和变量的内存分配。它也是垃圾回收器，清理你完成的工作。

现在，这里有一个绝对关键的问题要弄清楚：V8 *不*做什么。V8 本身对现实世界一无所知。它没有文件系统的概念，不知道如何打开网络套接字，也不知道如何设置定时器。像 `setTimeout`、`fs.readFile` 和 `http.createServer` 这样的函数？它们不是 JavaScript 或 V8 的一部分。它们是由浏览器或在我们的情况下由 Node.js 提供的 API。

将 V8 想象成一个出色的语言学教授，他只会说 JavaScript。要在现实世界中做任何事情，它都需要一个解释器和助手。

### Libuv

Libuv 是一个从头开始构建的 C 库，其目的只有一个：异步 I/O。它最初是为 Node.js 制作的，也是赋予 Node 事件驱动、非阻塞超能力的秘密配方。它的责任重大，但我们可以将其归纳为三个大类别 -

**事件循环本身**。没错。我们一直谈论的事件循环？它是通过 Libuv 实现和运行的。我们很快将要讨论的六个阶段都是由 Libuv 的 C 代码编排的。当你启动 Node 进程时，是 Libuv 启动循环。

**抽象操作系统**。这里的魔法真正发生了。不同的操作系统都有自己处理异步操作的高效方式。Linux 有 `epoll`，macOS 有 `kqueue`，Windows 有它的 I/O 完成端口（IOCP）。这些都是内核级别的工具，允许程序说：“嘿，帮我监视这些文件和网络套接字，当发生有趣的事情时，就唤醒我。”Libuv 提供了一个单一、美丽的 C API，它可以在所有这些不同的系统上工作。这就是为什么你的 `http.createServer` 代码可以在任何地方高效运行，而无需你更改任何一行。当你告诉 Node 在端口上监听时，是 Libuv 正确地调用操作系统进行非阻塞调用。

**线程池**。好吧，这是一个常见的混淆点，所以请靠近一点。我们说 Node 是单线程的，但这只是故事的一半。你的 JavaScript 在一个线程上运行，是的。但是 Libuv 维护它自己的一个小型内部线程池。为什么？因为尽管现代操作系统很强大，但有些操作是不可避免地、固执地阻塞的。这包括大多数文件系统操作，一些 DNS 查询，以及一些 CPU 密集型的加密函数。如果 Node 在主线程上运行这些操作，它们会阻塞循环——游戏结束。相反，Libuv 聪明地将这些特定的工作委托给它的线程池。线程池中的一个工作线程执行缓慢的阻塞系统调用。当它完成时，它会向主事件循环发出信号，然后主事件循环最终可以执行你的 JavaScript 回调。这个池的默认大小是四个，但你可以使用 `UV_THREADPOOL_SIZE` 环境变量来更改它（这是一个非常实用的技巧！）。

### Node.js C++ 绑定

因此，我们现在有两个独立的世界。我们有 V8 世界，它使用 JavaScript 进行交流，还有 Libuv 世界，这是一个使用文件描述符和系统调用的 C 库。他们究竟是如何互相交流的呢？

它们通过一组称为 **Node.js 绑定** 的 C++ 程序进行通信。这些绑定是至关重要的翻译层，是连接 V8 世界和 Libuv 世界的桥梁。

当你进行一个看似简单的调用，如 `fs.readFile('/path/to/file', callback)` 时，幕后会发生一系列动作——

1.  **V8** 看到了 `fs.readFile` 函数调用，并开始执行它。

1.  但等等！这个函数不是纯 JavaScript；它是一个绑定。调用立即被路由到 Node 源代码中的一个特定 C++ 函数。

1.  这个 C++ 绑定函数充当翻译者的角色。它接收你的 JavaScript 参数（文件路径和你的回调函数），并将它们打包成一个 Libuv 可以理解的“请求”对象。

1.  然后将这个请求交给 **Libuv**，并告诉它：“帮我读取这个文件，请使用线程池，因为这个操作可能需要一段时间。”

1.  Libuv 做它的事情。一旦文件被读取，它将在队列上放置一个“完成事件”。

1.  之后，**事件循环**（由 Libuv 运行）看到这个事件正在等待。

1.  事件信号返回到 C++ 绑定。

1.  绑定将结果（无论是文件数据还是错误）转换回 JavaScript 友好的格式，然后——最终！——它使用这些结果调用你的原始 JavaScript **回调函数**，并将这个回调推送到 V8 **调用栈**上执行。

呼吸一下。这个往返——从 JavaScript 到 C++ 到 Libuv 到操作系统，然后再返回——是 Node.js 中每个异步操作的生命故事。

## 六个阶段的详细说明

💡提示

有一个由 [@vagostep](https://github.com/vagostep) 创建的出色工具，可以让你可视化事件循环的工作方式。你可能想尝试一下。以下是链接 - [NodeLoops](https://nodeloops.com/)

事件循环不仅仅是一个大队列。这是一个常见的误解。它是一个高度结构化的、多阶段的循环。在这个循环中完整一圈被称为一个“tick”。理解这些阶段是绝对的关键，了解异步操作为什么以它们执行的顺序执行。Libuv 的循环只是对这些六个核心阶段的重复旅行。

### “Tick”：单次迭代的概述

首先：一个“tick”不是一个时间单位。它只是事件循环所有阶段的单次完整进展。一个 tick 不一定等于特定的毫秒数；它持续多长时间取决于那次迭代中完成的工作。

ℹ️注意

如果你玩过（或开发过）游戏，可以把一个 tick 想象成一个帧：事件循环，就像游戏循环一样，在每个 tick 重复执行一次工作。

在一个 tick 期间，循环检查每个阶段的队列。如果一个阶段的队列有等待的回调，它将按顺序逐个执行它们，直到队列为空或达到某些系统依赖的限制。然后，它继续前进到下一个阶段。就这么简单。

### 第一阶段：计时器

这是我们的导游的第一站。循环在这里的唯一任务是运行由 `setTimeout()` 和 `setInterval()` 安排的回调。

现在，从技术上讲，计时器回调并不保证在指定的**确切**毫秒运行。你提供的延迟是回调可以运行的**最小**时间。当循环进入这个阶段时，它会检查时钟。我们安排的任何计时器的时间是否已经过去？如果是，它们的回调将被运行。

你可能想知道 Node 如何处理数千个计时器而不需要不断检查一个巨大的列表。它比这更聪明。Libuv 使用一种特殊的数据结构，称为 **最小堆**。最小堆是一种树状结构，其中最小的元素始终在根节点。在这种情况下，“最小”意味着下一个即将到期的计时器。这使得 Libuv 能够知道它可以在 O(1) 时间内“睡眠”多久，直到下一个计时器到期 - 非常快。这是 Node 计时器如此便宜的一个巨大原因。

ℹ️注意

Libuv 使用最小堆，因此可以在 O(1) 时间内发现下一个即将到期的计时器，但插入或删除计时器是 O(log n)。这使得计时器在大集合中效率很高，但创建或取消许多计时器仍然有非零成本。

### 第二阶段 & 第三阶段：挂起回调和内部操作

在计时器之后，循环快速跳过几个你很少直接与之交互的内部阶段。

+   **第二阶段：挂起回调** - 这个阶段运行被推迟到下一次循环迭代的 I/O 回调。这是一个真正奇怪的边缘情况。例如，如果一个 TCP 套接字在写入数据时抛出 `EAGAIN` 错误，Node 将将回调排队以在此处重试。对于 99% 的开发者来说，这个阶段只是背景噪音。

+   **第三阶段：空闲、准备** - 这些是由 Libuv 在处理真正重要的事情之前内部使用的。在 JavaScript 领域对我们完全不公开。

### 第 4 阶段：轮询阶段

ℹ️注意

**轮询**通常意味着反复询问“是否有任何准备就绪？”例如，询问内核哪些 I/O 处理程序（文件描述符、套接字等）准备好执行 I/O。

好吧，请注意，这是最重要的一个。轮询阶段可以说是整个循环中最重要和最复杂的部分。它做两件主要的事情 -

1.  **计算等待时间和轮询 I/O。** 循环确定它可以为新的 I/O 事件等待多长时间。它查看下一个计时器何时到期以及其他因素，然后调用操作系统的通知系统（例如 Linux 上的 `epoll_wait`）。这是事件循环中唯一的“阻塞”部分，但这是良好类型的阻塞。进程使用零 CPU，只是耐心地等待内核轻拍它的肩膀说，“嘿，你正在读取的那个文件已经完成了，”或者“你有一个新的网络连接。”

1.  **处理轮询队列。** 当等待结束（无论是时间到了还是发生了 I/O 事件），循环处理轮询队列。这个队列持有几乎所有 I/O 操作的回调：网络连接建立、从套接字读取数据，或文件读取（从线程池）完成。

这里的行为很聪明。如果轮询队列**不为空**，循环将处理其回调，直到队列清空。但如果轮询队列**为空**，循环的行为会改变 -

+   如果有任何脚本使用 `setImmediate()` 被安排，循环将立即结束轮询阶段，进入 检查阶段 来运行它们。

+   如果没有等待的 `setImmediate()`，循环将在这里挂起，等待新的 I/O 事件到来。当它们到来时，它们的回调将立即执行。

这个阶段也是 Node 进程决定何时死亡的时候。如果事件循环进入轮询阶段，看到没有挂起的 I/O、没有活跃的计时器、没有立即执行的任务，也没有其他保持其存活的处理程序，它就会得出结论，没有更多工作要做，进程优雅地退出。

### 第 5 阶段：检查阶段

这个阶段非常简单。它只有一个任务：执行由 `setImmediate()` 安排的回调。如果你需要在轮询阶段完成其事件后立即运行一些代码，这是你的工具。

#### 用例：食品配送应用的订单确认

⚠️警告

`setImmediate()` 由于延迟原因解耦了工作，但不是持久的 - 它仅在进程存活时执行。对于关键或保证的后台任务，请使用持久作业队列（RabbitMQ、Redis 队列、Kafka 或数据库作业表）或外部工作员以确保重试和持久性。我在这里使用它作为示例来阐述事件循环。

想象你正在构建像 Uber Eats 或 Zomato 这样的食品配送服务的后端。当用户下单时，需要发生两件主要的事情 -

1.  **确认订单** - 将订单详情写入你的主数据库。这是一个关键的 I/O 操作，必须成功完成。

1.  **通知餐厅** - 向餐厅的平板电脑或系统发送通知。这是一个单独的操作，不应延迟用户的确认。

你想在数据库写入完成后立即告诉用户他们的订单已确认。餐厅通知可以在几秒钟后发生；它不需要成为同一数据库事务的一部分。

#### `setImmediate()`如何解决这个问题

你会使用`setImmediate()`来解耦餐厅通知和数据库确认。

系统接收一个订单并调用一个函数将其保存到数据库中（一个 I/O 操作）。这个回调将在事件循环的**轮询阶段**运行。

在数据库操作的回调内部，一旦你知道订单已成功保存，你立即使用`setImmediate()`安排餐厅通知。

事件循环完成轮询阶段，并立即进入**检查阶段**，在那里执行`setImmediate()`回调，向餐厅发送通知。

这确保了核心的用户任务（在数据库中确认订单）尽可能快地完成。次要的内部任务（通知餐厅）被可靠地安排在之后发生，而不会减慢主要任务。

简化的代码逻辑看起来是这样的 -

```js
// Function to handle a new food order
function placeOrder(orderDetails) {
 // 1\. Save order to the database (this is an async I/O operation)
 database.saveOrder(orderDetails, (error, savedOrder) => {
 // This callback runs in the Poll phase after the database write is done.
 if (error) {
 console.error("Failed to save order!", error);
 return;
 }
 console.log(`Order #${savedOrder.id} confirmed in the database.`);
 // 2\. Schedule the restaurant notification to run immediately after this.
 // This decouples the notification from the database logic.
 setImmediate(() => {
 // This code will run in the Check phase, right after the current Poll phase.
 notificationService.sendToRestaurant(savedOrder);
 console.log(`Notification for order #${savedOrder.id} sent to the restaurant.`);
 });
 // We can now immediately respond to the user without waiting for the notification to be sent.
 console.log(`Sending confirmation back to the user for order #${savedOrder.id}.`);
 });
}
```

### **阶段 6：关闭回调**

蜱虫的最终阶段是清理。它处理“关闭”事件。例如，如果你使用`socket.destroy()`突然销毁一个套接字，那么在这个阶段将会触发`'close'`事件的回调。

然后，循环检查是否还有任何东西让它保持活跃。如果有，整个周期将重新开始，回到下一个 tick 的计时器阶段。就这样一直进行下去。

## 快车道：微任务与宏任务

因此，我们刚刚概述了事件循环的六车道高速公路。但事实上，还有一个更高优先级的快速车道，它在这个系统之外运行。理解这一点对于预测执行顺序至关重要。欢迎来到微任务的世界。

### 这些究竟是什么？

为了正确理解，我们需要对我们的术语更加正式一些。

+   **宏任务（或任务）** - 这是指被放入六个事件循环阶段之一的队列中的任何回调。定时器回调？那是宏任务。I/O 回调？宏任务。立即回调？没错，宏任务。事件循环在每个 tick 中从每个阶段的队列中处理宏任务。

+   **微任务** - 这是指被放入一个特殊的高优先级队列中的回调，该队列存在于主循环阶段之外。在 Node 中，有两个这样的队列：`nextTick`队列和 Promise Jobs 队列。

这里是执行顺序的金科玉律，你应该将其纹在脑中：在 **任何单一阶段的宏任务** 执行完毕后，运行时会立即执行微任务队列中当前的所有任务，然后再考虑移动到下一个宏任务。 

这非常重要。这意味着微任务可以在宏任务之间插队并执行，甚至来自同一阶段的宏任务。

### `process.nextTick()` 队列：最高优先级

你使用 `process.nextTick()` 安排的回调生活在微任务队列的 VIP 休息室。这个名字有点误导；它不是在“下一个 tick”上运行。它在调用栈上的当前操作完成后立即运行，甚至在事件循环被允许进入下一个阶段或下一个宏任务之前。这是最激进的“插队”方式。

这赋予它巨大的力量，但也使其变得极其危险。因为 `nextTick` 队列在循环可以继续之前会全部处理，递归的 `process.nextTick()` 调用可以饿死事件循环，防止任何 I/O 或定时器运行。

我曾经花了一整天的时间调试一个完全对网络请求无响应但并未崩溃的服务器。罪魁祸首？一个库在特定的错误条件下意外地递归调用 `process.nextTick`。循环永远在处理微任务。

```js
let count = 0;
 function starveTheLoop() {
 console.log(`Starvation call: ${++count}`);
 process.nextTick(starveTheLoop);
}
 // A timer that will never get to run
setTimeout(() => {
 console.log("This will never be logged!");
}, 1000);
 console.log("Starting the starvation...");
starveTheLoop();
```

运行那段代码。它将永远打印“Starvation call...”。`setTimeout` 回调永远不会得到机会，因为事件循环永远卡住，无法到达定时器阶段。

#### 那么，这是为什么会发生呢？

让我们通过执行那段代码来了解它是如何将事件循环困在永无止境的循环中的。

1.  `let count = 0;` 我们首先设置一个简单的计数器。这只是为了证明我们的函数确实在反复运行。

1.  `function starveTheLoop() { ... }` 这是我们要讨论的代码。我们定义了函数，但还没有发生任何事情。它只是静静地坐在内存中，等待被调用。

1.  `setTimeout(() => { ... }, 1000);` 我们安排了一个定时器。Node.js 看到这个并说：“好吧，酷。大约一秒后，我会把这个回调放入 **定时器队列**（一个宏任务队列）以执行。”然后它立即继续。它不会等待第二秒过去。

1.  `console.log("Starting the starvation...");` 这是实际运行的第一个代码片段。这是一个同步操作。它被推入调用栈，将其消息打印到控制台，然后弹出。很简单。

1.  `starveTheLoop();` 这就是陷阱被触发的位置。我们第一次调用我们的函数。接下来发生的情况是 -

    +   `starveTheLoop` 被推入调用栈。

    +   它打印 `Starvation call: ${++count}`。现在控制台显示“Starvation call: 1”。

    +   现在是关键部分：它调用 **`process.nextTick(starveTheLoop)`**。这并不会立即调用函数。相反，它将 `starveTheLoop` 函数放入高优先级的 `nextTick` 队列中。

    +   第一次 `starveTheLoop` 调用完成并被弹出调用栈。

1.  主脚本现在已执行完毕，调用栈为空。事件循环准备接管。它的任务是检查待处理任务的队列。它应该按阶段工作：检查计时器，检查 I/O 等。**但是**，在它能够移动到 *任何* 阶段之前，它有一个严格的规则：**"我必须处理整个 `process.nextTick()` 队列，直到它为空。"**

1.  无解循环开始

    +   事件循环查看 `nextTick` 队列，看到我们的 `starveTheLoop` 函数在那里等待。

    +   它将其拉出并执行它（这是第 #2 次调用）。

    +   函数打印 "饥饿调用：2"。

    +   然后...它在 `nextTick` 队列中安排了 *另一个* `starveTheLoop` 回调。

    +   第二次调用完成。事件循环再次检查 `nextTick` 队列。它是空的吗？**不是！** 有一个新的任务在等待。

    +   因此，它第三次运行 `starveTheLoop`，打印 "饥饿调用：3" 并将一个 *第四个* 作业放回队列中。

事件循环完全卡住了。它永远无法完成处理 `nextTick` 队列，因为每次它处理一个项目，该项目就会将另一个项目放回队列中。可怜的 `setTimeout` 回调坐在计时器队列中，耐心地等待它的轮次，但事件循环永远不会通过 `nextTick` 阶段去查看计时器。它已经被有效地 **饿死**。

### Promise 作业队列

第二个微任务队列用于 Promise。每当 Promise 解决或拒绝时，通过 `.then()`、`.catch()` 或 `.finally()` 添加的任何回调都会作为微任务在此队列中安排。是的，我们喜爱的 `async/await` 只不过是一种使用这个机制的同义词的语法糖。

此队列的优先级略低于 `process.nextTick()`。操作顺序始终是：

1.  执行当前宏任务。

1.  清空整个 `nextTick` 队列。

1.  清空整个 Promise 作业队列。

1.  好吧，**现在**我们可以继续到下一个宏任务。

当你 `await` 什么时，你实际上是在将你的 `async` 函数分成两部分。`await` 之前的一切都是同步运行的。函数的其余部分被包装在 `.then()` 中，并作为微任务在此队列中安排，以便在等待的承诺解决后执行。

### 复杂执行顺序分析

让我们用一个看起来令人恐惧的代码片段将这些内容综合起来，以测试我们新的思维模型。

```js
const fs = require("node:fs");
 console.log("1\. Start");
 // Macrotask: Timer
setTimeout(() => console.log("2\. Timeout"), 0);
 // Microtask: Promise
Promise.resolve().then(() => console.log("3\. Promise"));
 // Microtask: nextTick
process.nextTick(() => console.log("4\. nextTick"));
 // Macrotask: I/O
fs.readFile(__filename, () => {
 console.log("5\. I/O Callback");
 // Macrotask from I/O: Immediate
 setImmediate(() => console.log("6\. Immediate from I/O"));
 // Microtask from I/O: nextTick
 process.nextTick(() => console.log("7\. nextTick from I/O"));
 // Microtask from I/O: Promise
 Promise.resolve().then(() => console.log("8\. Promise from I/O"));
});
 console.log("9\. End");
```

此处的输出总是，确定性地：`1, 9, 4, 3, 2, 5, 7, 8, 6`。让我们一步一步地分析原因：

1.  `'1. 开始'` 和 `'9. 结束'` 是同步记录的。所有异步内容都被安排。

1.  主脚本结束。调用栈为空。**黄金法则时间：清空微任务队列！**

1.  `nextTick` 队列总是首先执行。我们记录 `'4. nextTick'`。

1.  接下来是 Promise 作业队列。我们记录 `'3. Promise'`。

1.  微任务队列现在为空。事件循环终于可以开始它的第一次滴答。

1.  **阶段 1：计时器。** 我们的 `setTimeout(..., 0)` 已准备好。宏任务运行，我们记录 `'2. Timeout'`。

1.  循环快速通过接下来的几个阶段。

1.  **阶段 4：轮询**。循环等待 I/O。最终，`fs.readFile`完成。它的回调现在在轮询队列中作为一个宏任务，准备就绪。

1.  宏任务被执行。我们记录`'5\. I/O Callback'`。在这个函数内部，安排了一个新的立即执行、nextTick 和 promise。

1.  I/O 宏任务完成。接下来会发生什么？**黄金法则再次适用！我们必须在继续之前清空微任务**。

1.  检查`nextTick`队列。我们找到一个并记录`'7\. nextTick from I/O'`。

1.  检查 Promise 作业队列。我们找到一个并记录`'8\. Promise from I/O'`。

1.  微任务队列再次为空。循环现在可以从轮询阶段离开的地方继续进行。

1.  **阶段 5：检查**。循环看到了我们安排的`setImmediate`。宏任务运行，我们记录`'6\. Immediate from I/O'`。

1.  循环完成其 tick，没有其他事情可做，进程退出。

看见了吗？这不是魔法。只是规则。

## 3Ps：性能、模式和陷阱

这不仅仅是一个学术练习。真正理解事件循环的内部结构直接影响到你编写好代码的方式，更重要的是，如何调试坏代码。

### 明显和微妙的阻塞器

我们已经把明显的阻塞器打得体无完肤：同步 API 如`fs.readFileSync`和长、CPU 密集型循环。但我甚至看到有经验丰富的开发者被更微妙的阻塞器绊倒，这些阻塞器可能会毒害应用程序的性能。

1.  **大型 JSON 操作**。这是一个狡猾的例子。`JSON.parse()`和`JSON.stringify()`是 100%同步、阻塞的操作。如果你正在处理一个带有大量 JSON 有效负载的 API 请求（想想十或数百兆字节），解析所需的时间可能非常长——你的循环完全冻结时，可能需要数十或数百毫秒。如果你发现自己处于这种情况，可以考虑使用类似`stream-json`的流式 JSON 解析器。

1.  **复杂正则表达式**。编写不良的正则表达式就像一颗定时炸弹。存在一种被称为“灾难性回溯”的讨厌现象，它可能导致正则表达式引擎处理某些字符串时花费指数级的时间。单个恶意用户输入就能触发这种情况，导致正则表达式匹配阻塞 CPU 数秒甚至数分钟。这是一个经典的拒绝服务（DoS）向量。始终，**始终**测试你的正则表达式对“邪恶”字符串，并考虑使用提供保护的库。

### 再次探讨 Libuv 线程池

记住那个 Libuv 线程池吗？如果你不记得，我们的下一章将深入探讨 Libuv 本身！重要的是要记住它是一个全局、共享的资源，默认情况下，它只有四个线程。虽然从 JavaScript 的角度来看，像`fs.readFile`和`crypto.pbkdf2`这样的函数*感觉*是异步的，但它们都在等待非常少的实际线程。

这可能会产生一些令人惊讶的瓶颈。想象一下，一个服务器接收到一个请求，需要从慢速网络驱动器中读取一个文件（`fs.readFile`）并验证密码（`crypto.pbkdf2`）。现在，想象一下五个这样的请求同时到达。

1.  前四个请求将各自向线程池派遣一个任务（假设文件读取先到达）。现在所有四个线程都在忙碌。

1.  第五个请求的 `fs.readFile` 调用被发起。Libuv 尝试将其传递出去，但池已满。现在这个第五个任务必须等待在队列中。

1.  那么前四个请求的密码散列怎么办？它们也必须等待在那个相同的队列中，直到其中一个文件读取完成并释放一个线程。

突然之间，你的慢速文件系统使你的认证延迟急剧上升。所有使用线程池的应用程序都是相连的。如果你有一个在文件 I/O、DNS 和加密方面负担较重的应用程序，你可能需要认真考虑通过 `UV_THREADPOOL_SIZE` 环境变量增加线程池大小，以避免这种类型的拥堵。

### 分析和调试事件循环

那么你如何知道你的循环是否在挣扎？你必须测量它。不要猜测，要测量。

+   **方法 1：穷人的延迟检查器。** 这是一种低技术手段，感觉有点“黑客”风格，但出人意料地有效，可以快速进行直观检查。

    ```js
    let lastCheck = Date.now();
    setInterval(() => {
     const now = Date.now();
     const delay = now - lastCheck - 1000;
     if (delay > 50) {
     // a 50ms delay is pretty noticeable
     console.warn(`Event Loop Latency: ${delay}ms`);
     }
     lastCheck = now;
    }, 1000);
    ```

    如果你开始看到警告，这意味着一个简单的 `setInterval` 宏任务被延迟了，这是一个表明其他东西正在占用 CPU 的明显信号。

+   **方法 2：`perf_hooks.monitorEventLoopDelay`。** 对于更专业的方法，Node 提供了一个内置的高分辨率工具来完成这个特定的目的。

    ```js
    const { monitorEventLoopDelay } = require("node:perf_hooks");
    const h = monitorEventLoopDelay();
    h.enable();
     // ... your application logic ...
     // Periodically check the stats
    setInterval(() => {
     // The mean is in nanoseconds, so we convert to ms
     console.log("Event Loop Delay (ms):", h.mean / 1_000_000);
     h.reset();
    }, 5000);
    ```

    这比 `setInterval` 诡计更准确，并且开销更低。在生产环境中使用这个。

+   **方法 3：`async_hooks`。** 这是重型武器。对于超级高级调试，`async_hooks` 模块允许你追踪你应用程序中每个异步资源的整个生命周期。它非常强大，但也非常复杂。你通常只有在构建开发者工具或 APM（应用程序性能管理）解决方案时才会使用它。

## CPU 密集型和并行工作的策略

有时候你有一个真正密集的 CPU 任务。无论多么巧妙的异步技巧都无法解决这个问题。解决方案不是阻塞循环；而是将工作完全从循环中移除。

### 将任务卸载到循环中

对于可以分割成更小部分的长任务，你可以使用一个巧妙的技巧来避免阻塞。想法是完成一块工作，然后使用 `setImmediate()` 安排下一块工作。这实际上在块之间将控制权交还给事件循环，允许它处理 I/O 并保持响应。

```js
// A very long array to process
const bigArray = Array(1_000_000)
 .fill(0)
 .map((_, i) => i);
let sum = 0;
const CHUNK_SIZE = 1000;
 function processChunk() {
 const chunk = bigArray.splice(0, CHUNK_SIZE);
 for (const item of chunk) {
 sum += item; // Do a little bit of work
 }
 if (bigArray.length > 0) {
 // There's more to do, so yield to the event loop
 // and schedule the next chunk to run ASAP.
 setImmediate(processChunk);
 } else {
 console.log("Processing complete. Sum:", sum);
 }
}
 processChunk();
console.log("Started processing... but the loop is not blocked!");
```

这种模式对于防止你的应用程序冻结非常有效，但请注意，它实际上并没有加快总计算时间。为了实现这一点，我们需要真正的并行处理。

### 真正的并行处理：`worker_threads`

ℹ️注意

我们有一个专门的课程专门讲解`worker_threads`——包括 7 个章节。本节只是一个快速概述。所以，如果你在这里没有完全理解所有细节，请不要担心。只需理解大局即可。

`worker_threads`模块（自 node v12 起稳定）是针对 CPU 密集型工作的现代、权威的解决方案。工作线程不是来自 Libuv 池的线程。它是一个完全独立的 V8 实例，在自己的线程上运行，拥有自己的事件循环和隔离的内存。

这种隔离是杀手级特性。因为内存不共享，你可以避开所有多线程编程的经典头痛问题，如竞态条件和死锁。你可以通过消息传递通道安全地在主线程和工作线程之间进行通信。

```js
// main.js
const { Worker } = require("node:worker_threads");
 console.log("Main Thread: Kicking off a worker for a heavy task.");
const worker = new Worker("./heavy-task.js");
 worker.on("message", (result) => {
 console.log(`Main Thread: Got the result back! -> ${result}`);
});
 worker.on("error", (err) => console.error(err));
 // heavy-task.js
const { parentPort } = require("node:worker_threads");
 let result = 0;
// A truly, horribly, no-good heavy task
for (let i = 0; i < 1e10; i++) {
 result += 1;
}
 // Send the result back when we're done
parentPort.postMessage(result);
```

在这里，那个糟糕的`for`循环在一个完全独立的 CPU 核心上运行，让我们的主线程的事件循环保持空闲，可以继续处理 Web 请求或它需要做的其他事情。

### `cluster`模块

重要的是不要混淆`worker_threads`和较旧的`cluster`模块。它们解决不同的问题。`cluster`不是用于卸载一个重任务。它是用于在整个机器的 CPU 核心上扩展整个 I/O 密集型应用程序（如 HTTP 服务器）的工具。

它通过将主 Node 进程分叉成多个子进程来工作。主进程获取一个端口（比如，8000），然后充当负载均衡器，将传入的 TCP 连接分配给工作进程。每个工作进程都是你 Node 应用的完整副本，拥有自己的独立事件循环。这使得 8 核机器可以运行 8 个你的服务器实例，有效地增加了处理并发连接的能力。

ℹ️注意

想象一下它们的初级扩展策略是这样的：`worker_threads`用于将特定的、长时间运行的 CPU 密集型任务从单个事件循环卸载，以防止该循环阻塞。另一方面，`cluster`模块是用于通过运行多个独立进程实例来扩展整个应用程序，使其跨越多个 CPU 核心。这对于你的服务器来说非常有效，因为它允许你通过将它们分配到多个事件循环来处理更多的并发连接。这些工具不是互斥的，可以强大地结合使用。

## 人们常犯的错误

让我们以几个经典的脑筋急转弯来结束，这些脑筋急转弯真正考验你是否已经内化了循环的工作方式。

### 7.1 `setTimeout(..., 0)` 与 `setImmediate()`

这是一个著名的面试问题：这些哪个先运行？答案是令人沮丧的：**这取决于**。

**案例 1：从主模块调用**

```js
setTimeout(() => console.log("Timeout"), 0);
setImmediate(() => console.log("Immediate"));
```

当你直接在脚本中运行时，执行顺序是**非确定性的**。你可能会先得到 Timeout 然后是 Immediate，或者反过来。原因是微妙的。当脚本被处理时，计时器和 Immediate 被安排。然后事件循环启动。`setTimeout(..., 0)`实际上并没有 0ms 的延迟；它受系统最小值的限制，通常在 1ms 左右。当循环进入计时器阶段时，它会检查 1ms 是否已经过去。如果循环的初始启动时间超过 1ms（在繁忙的系统上这是完全可能的），计时器会首先触发。如果启动非常快，循环会飞过（仍然为空的）计时器阶段，进入轮询阶段，然后是检查阶段，首先运行`setImmediate`。这是一个竞争条件。

**案例 2：在 I/O 回调中调用**

```js
const fs = require("node:fs");
 fs.readFile(__filename, () => {
 setTimeout(() => console.log("Timeout"), 0);
 setImmediate(() => console.log("Immediate"));
});
```

在这里，顺序是**始终，100%确定性的**。`setImmediate`将首先执行。**为什么这么确定？**I/O 回调本身在轮询阶段运行。当它安排计时器和 Immediate 时，循环正处于轮询阶段。下一个阶段是什么？检查阶段。所以`setImmediate`回调有保证会运行。计时器回调必须等待循环完成整个周期，并在*下一个 tick*回到计时器阶段。

### 垃圾回收及其对循环的影响

我们需要讨论的一个最后、看不见的阻塞来源是 V8 的垃圾回收器（GC）。为了清理不再使用的对象占用的内存，GC 必须定期暂停你的 JavaScript 执行。这通常被称为“停止世界”事件，它听起来就像它听起来那样戏剧性。

虽然 V8 的 GC 是工程奇迹，但在内存压力大的应用程序中，一个主要的 GC 周期仍然可以使你的事件循环冻结数十毫秒甚至数百毫秒。在这段暂停期间，什么都没有发生。没有 JavaScript 运行。你的服务器就像被同步代码阻塞一样无响应。这就是为什么在 Node 中良好的内存管理——比如使用流而不是缓冲大文件——是如此关键。它使那些 GC 暂停变得短暂而甜蜜。

## 最后的话

呼呼。我们已经从简单的调用栈过渡到了 V8 和 Libuv 之间的舞蹈，经历了循环的六个阶段，进入了微任务的 VIP 地位。

掌握事件循环不仅仅是记住六个阶段的名称来参加知识竞赛。这是关于构建一个稳固、可靠的思维模型，让你能够推理你的代码将如何实际运行。这个模型是一种超级能力。它让你能够写出极快、非阻塞的应用程序。它帮助你诊断最棘手的性能问题。并且它让你真正欣赏到是什么让 Node.js 成为一个如此强大和独特的环境。

在脑海中有了这个模型，你不再仅仅是*使用* Node.js。你开始*思考*它。现在去写一些代码，`console.log`一切，看看你是否能预测结果。这就是真正理解它的方法。快乐编码。
