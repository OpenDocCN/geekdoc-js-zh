# Node.js 进程生命周期

> 原文：[`www.thenodebook.com/node-arch/node-process-lifecycle`](https://www.thenodebook.com/node-arch/node-process-lifecycle)

# Node.js 进程生命周期

⚠️警告

你已经提前获得了这一章节的访问权限。你的反馈是无价的，所以请在下面的评论部分分享你的想法。

好吧，让我们来谈谈 Node.js 进程的生命周期。

我知道你在想什么。“生命周期？那不是一些无聊的学术东西吗？”大多数工程师认为 Node 服务只是一个命令：`node server.js`。他们假设`require()`是免费的，`process.exit()`是一种干净的停止方式，一个单一的`SIGTERM`处理器就足够用于“优雅关闭”。

我曾经也这么想。而这些假设是导致我见过的最严重的生产问题的原因之一。我谈论的是简单部署期间的数据损坏，在负载下陷入崩溃循环的服务，以及触发整个系统 OOM 杀死的内存泄漏。

事实上，Node.js 进程有一个复杂的生命周期，从你输入 `node` 的那一刻直到它的最后一息。每个阶段都是可能出现严重错误的地方。

这不仅仅是一个章节。这是一本关于生命周期的*深入*指南。我们将剖析整个旅程：在你代码甚至得到一看之前的 C++引导，`require()`令人惊讶的高成本，以及一个真正的优雅关闭的谨慎团队合作。忘记简单的`try...catch`。我们谈论的是资源管理的现实——可能会泄漏并使整个服务崩溃的文件描述符、套接字和定时器。

到了最后，你将最终理解*为什么*你的服务启动需要花费很长时间，*为什么*它在重启时有时会损坏数据，*为什么*它会泄漏句柄。更重要的是，你将拥有一个构建 Node.js 应用程序的坚实基础，这些应用程序启动快、运行可靠、关闭干净。

这不是关于边缘情况。这是任何严肃的后端工程师的核心能力。忽视进程生命周期是选择构建脆弱系统。尊重它是迈向真正能在生产中生存的第一步。

## Node.js 进程的诞生

当你输入 `node my_app.js` 时，你正在启动一系列事件，这些事件在你 JavaScript 代码的任何一行运行之前就已经发生了。我们大多数人只是把它当作理所当然。Node 只是...启动，对吧？

不。实际上，这是 C++、V8 引擎和内部引导脚本之间精心编排的舞蹈。而你看到的很多奇怪现象——缓慢的启动、奇怪的环境问题——都始于这里。

旅程不是从 JavaScript 开始的，而是在 Node.js 源代码内部，在一个 C++文件中。这是真正的入口点。

这里是 C++领域中发生的事情的简化序列 -

1.  过程开始于 `main` 函数。它解析你的命令行参数（`--inspect`、`--max-old-space-size` 等等）并设置基本的过程属性。

1.  我们已经讨论过这一点，但 node 是基于 Google 的 V8 引擎构建的，它首先要做的是 **唤醒它**。这设置了用于后台任务（比如垃圾回收）的共享资源，如线程池。这只会发生一次。

1.  然后，它创建一个 **V8 Isolate**。一个 **isolate** 是 V8 引擎的单个、沙盒化的实例。它有自己的内存堆和自己的垃圾回收器。想象一下，这是一个你的 JavaScript 可以居住的小星球。创建这个是一个重量级的操作；它是在堆上直接分配大量内存的地方。

1.  在创建 V8 Isolate 之后，它必须在那个 isolate 内部 **创建一个 V8 Context**。这是所有内置内容的执行环境，你的代码期望的，比如 `Object`、`Array` 和 `JSON`。`global` 对象就生活在这里。

1.  然后，它继续 **初始化 libuv 事件循环**。这是重中之重。Node 的非阻塞 I/O 功能全靠 **libuv**。C++ 代码启动一个新的 libuv 事件循环。这个循环是 Node 的心脏。它处理所有的网络请求、文件操作和定时器，而不会阻塞。目前，它只是被创建，还没有运行。

1.  现在，是配置 libuv Threadpool 的正确时机。与事件循环一样，libuv Threadpool 被配置。想象一下，这是一支随时待命的辅助线程队伍。任何可能对操作系统造成缓慢和阻塞操作的事情（比如使用 `fs` 读取大文件、DNS 查询或一些密集的 `crypto` 操作），Node 都会将这项工作卸载到这些线程之一。这是让主事件循环保持空闲以处理其他传入请求的东西，确保没有任何东西被阻塞。

1.  之后，它必须 **创建 Node.js 环境**。创建了一个名为 `node::Environment` 的 C++ 对象。这是将一切粘合在一起的东西 - V8 isolate、context、libuv loop - 所有这些。

1.  现在，它将 **加载原生模块**或者从某种意义上说 - Node.js 标准库。所有酷炫的内置功能（`fs`、`http`、`crypto`）实际上不是 JavaScript。它们是与操作系统通信的 C++ 组件。在这个阶段，它们被注册，以便稍后可以通过 `require()` 将它们暴露给 JavaScript。

1.  但这还没有结束。现在，它必须 **执行引导脚本**。Node 首次实际运行一些 JavaScript。但这不是你的。这是一个内部脚本（`lib/internal/bootstrap/node.js`），它使用所有 C++ 绑定来构建我们熟悉和喜爱的 JavaScript 世界。它设置 `process` 对象，创建 `require` 函数本身，并为你的代码准备一切。这个脚本是从原始 C++/V8 世界到友好 Node.js API 的桥梁。

1.  最后但同样重要的是，它**加载你的代码**。只有在所有这些完成之后，Node 才会最终查看`my_app.js`。由引导脚本刚刚设置的模块加载器被调用以查找、读取和执行你的应用程序。

这里是整个漏斗 -

你为什么应该关心？因为这并不是免费的。这个预执行过程可能需要数百毫秒，有时甚至几秒钟。如果你在一个微小的容器中运行，这可能会成为一个巨大的瓶颈。我曾经在一个无服务器函数上工作，我们为了每一个冷启动时间毫秒而奋斗。我们发现，在我们`index.js`的任何一行代码被触及之前，就已经消耗了大约 300 毫秒。理解这个过程让我们可以使用工具来快照 V8 堆，有效地预编译代码并跳过一些这些步骤。这是成功产品和失败产品之间的区别。

## V8 和本地模块初始化

好吧，一旦 C++框架搭建完成，真正的任务就开始了：设置 V8 和本地模块。这是定义你整个应用程序性能和内存配置文件的地方。如果你不理解这部分，你会 wonder 为什么你的进程在服务器甚至开始之前就已经消耗了 100MB 的 RAM。

### 堆分配和即时编译（JIT）

当 Node 创建一个 V8 隔离时，它不仅仅是切换一个开关。它要求 V8 为 JavaScript 堆分配一个巨大的、连续的内存块。这就是你的每一个对象、字符串和函数将居住的地方。大小是可配置的（`--max-old-space-size`），但默认值相当大。

这个初始分配是启动成本的一大部分。Node 需要向操作系统请求内存，在一个压力之下，这可能非常慢。

一个常见的误解是 V8 的即时编译器（JIT）在这里“预热”。它不是。JIT 是懒惰的。它只有在函数运行了几次并变得“热”之后，才会将你的函数编译成优化的机器代码。在启动期间，V8 只是在解释内部的引导脚本。真正的 JIT 烟花在应用程序实际处理流量时才会发生。

ℹ️注意

V8 通常为堆保留一个大的虚拟地址范围，并强制执行堆限制，但操作系统在分配时可能不会实际提交所有这些内存 - 分配/提交行为取决于平台和 V8 标志（`--max-old-space-size`、`--initial-old-space-size`），并且可以进行微调。

### 连接本地模块

这是启动序列中最被低估的部分。像`fs`、`http`、`crypto`这样的模块——它们是工作马。它们是你美好的、安全的 JavaScript 世界与操作系统的原始力量之间的桥梁，通常是用 C++实现的。

在引导过程中，Node 实际上并没有加载所有这些模块。那样会慢且浪费。相反，它只是注册它们。它构建了一个内部映射，将字符串名称（如`'fs'`）映射到 C++函数指针。

所以当你的代码第一次调用`require('fs')`时，这是发生的 -

1.  `require`函数看到'fs'是一个内置模块。

1.  它在内部映射中查找'fs'。

1.  它调用它找到的 C++初始化函数。

1.  *这个* C++函数承担了繁重的工作。它创建了一个将成为`fs`模块的 JavaScript 对象，并将所有函数附加到它上面，比如`readFileSync`和`createReadStream`。这些 JS 函数只是底层 C++代码的薄包装。

1.  这个全新的模块对象被塞入缓存（`require.cache`）中，然后返回到你的代码。

这种懒加载是一个关键的优化。如果你的应用永远不会需要`crypto`，你永远不会为完全初始化它付出内存或时间的成本。

但是——这是一个很大的但是——在第一次`require`初始化这些本地模块的成本并不为零。我们曾经有一个服务，部署后的第一个 API 请求总是非常慢，有时超过 100ms。我们最终追踪到这是一个安全库，它在请求处理器内部第一次调用`require('crypto')`。设置所有 OpenSSL 上下文和 C++对象的单次成本正好发生在用户请求的关键路径上。

修复方法是可笑的简单：只需在主`server.js`文件的顶部添加`require('crypto')`。这把初始化成本从第一个请求移到了启动序列。是的，它使我们的启动时间慢了 100ms，但它使我们的运行时性能变得可预测。而在现实世界中，可预测几乎总是更好的。

## 模块加载和解析

模块系统是众多主题之一，通常开发者并不会给予它太多关注，因为它感觉非常简单，只需`require`一下就能开始构建。这一切都由`require()`函数管理，这个函数如此常见，以至于我们把它当作是瞬间的。

这是一个危险的假设。

模块系统，包括其解析算法和缓存，对启动性能和内存有巨大的影响。我仍然记得当我为使用虚幻引擎构建的游戏构建后端服务时，它对我来说是多么痛苦。

我们有一个服务，在生产环境中，有时启动需要近一分钟的时间。它只是在那里，消耗 CPU，在它开始监听端口之前很久。在我们的开发笔记本电脑上？3 秒。预发布环境？5 秒。生产环境？简直是灾难。部署编排器会直接放弃并杀死 pod，触发一个持续很长时间的崩溃循环。

突破来自于一个鲜为人知的 Node 标志：`--trace-sync-io`。这个标志会在主线程上发生同步 I/O 时大声呼喊。我们用这个标志运行我们的应用，控制台就爆炸了。成千上万的消息，都指向`fs.readFileSync`。但我们并没有直接调用那个函数！堆栈跟踪都结束在`require()`内部。

看看，`require()`并不是魔法。它是一个同步操作，猛烈地打击文件系统。这里就是它真正在做的事情——

给定一个像`'./utils'`或`'express'`这样的字符串，Node 需要找到文件的绝对路径。这是一个令人惊讶复杂的查找。

+   如果它是一个核心模块（`'fs'`），那就太好了，它已经完成了。

+   如果它以`./`或`../`开头，它是一个文件路径。它会尝试在末尾添加`.js`、`.mjs`、`.json`和`.node`。如果它是一个目录，它会查找`package.json`的`"main"`字段，或者回退到`index.js`。

+   如果它是一个裸名，如`'express'`，它将开始臭名昭著的`node_modules`遍历。它会检查`./node_modules`，然后是`../node_modules`，然后是`../../node_modules`，一直到文件系统的根目录。每一个这样的检查都是一个同步的文件系统调用。

一旦找到文件，它会检查一个缓存（`require.cache`）。

+   如果它已经在缓存中（一个**缓存命中**），它就只返回`exports`对象。这就是为什么你第二次`require('express')`时它超级快。但这并不是**免费的** - 它仍然是一个哈希表查找。

+   如果它不在缓存中（一个**缓存未命中**），这将是一条慢路径。节点创建一个新的`Module`对象，从磁盘读取文件（`fs.readFileSync` - 就是我们的罪魁祸首！），并准备编译它。

你文件的代码被这个函数包装起来 -

```js
(function (exports, require, module, __filename, __dirname) {
 // Your module's code goes here
});
```

那个包装器就是给你那些神奇的、模块局部变量的。然后整个字符串将由 V8 编译并运行。你放在`module.exports`上的任何内容都是结果。

我们的 45 秒启动时间是由一个巨大的`node_modules`目录造成的。每次`require()`都会触发数百次同步的文件系统检查。在我们的快速本地 SSD 上，你永远不会注意到。但在生产网络附加存储（NFS）上，由于其更高的延迟，所有这些微小延迟的影响加起来就是一场灾难。

解决方案有两个方面。首先，我们开始使用 Webpack 这样的打包器进行生产构建。这会将一切压缩成一个文件，并在运行时完全消除`node_modules`遍历。其次，我们对我们的依赖进行了无情的审计，并将树尽可能扁平化。

⚠️警告

不要将整个 Node.js 服务器打包在一起，这可能会引起很多问题。打包一切可能会破坏动态导入和本地模块。对于有针对性的修复，请使用像`esbuild`这样的工具来仅打包必要的部分。

这种经历也让我们遇到了另一个问题 - **模块缓存内存炸弹**。我们有一个长时间运行的过程，它在内存中不断增长，直到被 OOM-killed（内存不足）。我们在自己的代码中找不到任何泄漏。我们进行了堆快照，并发现了问题：`require.cache`。该服务正在动态生成报告，并且有位聪明的开发者写了这样一段代码：

```js
function renderReport(templateName) {
 // templateName was a unique path like '/tmp/report-1662781800.js'
 const template = require(templateName); // PLEASE, NEVER DO THIS
 return template.render();
}
```

因为每个 `templateName` 都是一个唯一的路径，Node 会将其视为一个全新的模块。它会加载文件，编译它，并将其存储在 `require.cache` 中……永远。一天后，缓存就有成千上万的条目，消耗了超过 2GB 的 RAM。解决办法是停止滥用 `require` 并使用 `fs.readFileSync` 与 `vm` 模块结合，在临时、沙箱化的环境中运行模板，这样实际上可以进行垃圾回收。

ℹ️注意

建议使用支持预编译和带驱逐缓存的成熟模板引擎（例如，Handlebars、Nunjucks），或者编译模板一次并重用函数。如果使用 `vm`，创建短期上下文，避免全局保留，并实现显式的缓存驱逐/限制以及监控。

模块系统很强大，但每个 `require()` 调用都可能成为性能瓶颈，并永久增加你进程的内存占用。请尊重它。

ℹ️注意

`require.cache` 中的条目可以手动删除（`delete require.cache[path]`），但使用 `require` 来处理动态、用户驱动的代码是不安全的。对于模板或临时模块，你应该使用 `fs.readFile` 与 `vm` 结合，这允许进行适当的垃圾回收。

## ES 模块（`import`）

好吧，所以关于 `require()` 我们所讨论的一切都是 Node 在过去十年中经过实战检验的经典方式。但多年来，在 JavaScript 社区中发生了一场缓慢的、持续的战争：CommonJS (`require`) 与 ES 模块 (`import`/`export`)。

我不会撒谎，Node 中的过渡非常混乱。长期以来，试图在 Node 中使用 `import` 感觉就像是在破坏规则。我们有过 `.mjs` 文件，然后在 `package.json` 中有 `"type": "module"`，关于互操作性的无尽争论——这真是个头疼的问题。但我们现在终于走上了正轨，ESM 现在已经成为标准。

那么这究竟是怎么回事？我们为什么要经历所有这些痛苦？

因为 `import` 不只是 `require` 的新语法。它从根本上改变了模块加载的生命周期。`require()` 是一个同步、动态、坦白说有点笨拙的函数，而 `import` 是异步的、静态的，并且要聪明得多。

### ESM 的新三阶段生命周期

记得 `require()` 只是读取并运行一个文件，在执行过程中会阻塞一切吗？ESM 使用完全不同的方法来处理这个问题。它分为三个阶段，你的代码甚至不会在最后一个阶段运行之前执行。

第一阶段是解析阶段（也称为**构建**阶段）。当 Node 遇到 `import` 时，它不会执行文件。相反，它会解析它，只查找其他 `import` 和 `export` 语句。它会递归地跟随这些导入，构建整个应用程序的完整依赖图，而不会运行你实际逻辑的任何一行。这是一件大事。它可以在你的应用程序甚至开始之前就找到缺失的文件或语法错误。`require()` 只会在启动过程中崩溃。

ℹ️注意

重要的是要注意，动态`import()`和其他运行时解析机制（条件导入、加载器）引入了运行时图的变化，并且可能在构建时不知道。

> 第二阶段是实例化。这是魔法部分。一旦它有了完整的图，Node 就会遍历它，为所有导出变量分配内存。然后它“连接”导入，使其指向导出的内存位置。想象一下创建了一堆指针。`moduleA.js`中的`exportedThing`和`moduleB.js`中的`importedThing`现在指向内存中的**确切相同位置**。它们是活绑定，不是副本。但——这是关键——它们还没有任何值。

最后的阶段是**评估**阶段。*现在*，终于，节点开始执行代码。它运行每个模块中的代码以“填补”已为其分配内存的导出值。因为它拥有完整的依赖图，所以它可以智能地从下往上开始评估；没有依赖的模块先执行。

这是一个完全的范式转变。`require()`将查找、加载和运行混合成一个阻塞步骤。ESM 将它们分开，这允许做一些令人难以置信的事情。

### 但...

尽管这个新系统很强大，但如果你是从 CJS 世界过来的，它可能会让你感到困惑。

首先，坏消息。所有那些你理所当然认为有用的变量？消失了。

```js
// throws a ReferenceError in an ES module
console.log(__filename);
console.log(__dirname);
 // The new way. It's... a bit clunky, I'll admit.
import { fileURLToPath } from "url";
import { dirname, join } from "path";
 const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
 console.log("Look mom, I found my way:", join(__dirname, "logs.txt"));
```

是的，那个`import.meta.url`东西是新的标准。它更明确，但肯定不那么方便。你会习惯的。

但现在到了真正酷的部分。所有这些复杂性的回报：**顶级`await`**。

因为 ESM 加载器是异步的，你现在可以在模块的顶层使用`await`，而不在`async`函数之外。还记得我们之前的异步引导模式吗？它变得简单多了。

```js
// Old CJS way - wrap everything in an async function
async function main() {
 const db = await connectToDatabase();
 // ... rest of the startup
}
main();
 // New ESM way with top-level await:
import { connectToDatabase } from "./database.js";
 console.log("Connecting to database...");
// This is happening at the top level! No async function needed.
const db = await connectToDatabase();
console.log("Database connected!");
 // Now you can start your server, confident the DB is ready.
import { startServer } from "./server.js";
startServer(db);
```

这是一个巨大的改进。它使启动序列线性且易于阅读，消除了整个类别的样板代码。该过程将在评估阶段的这个点简单地等待，直到承诺解决后再继续。这正是事情应该一直以来的样子。

### 那么，真实世界的影响是什么？

好吧，那么你为什么应该关心这个三阶段加载和顶级`await`？

+   **更快、更智能的启动（理论上）**。因为 Node 可以首先构建依赖图，所以它可以潜在地在网络上或从磁盘并行加载模块。而`require()`是一个串行的文件 I/O 队列，`import`则更像是一个协调的团队努力。

+   对**静态分析和树摇**的支持。因为`import { thing } from '...'`是静态和声明的，工具可以在不运行代码的情况下分析你的代码。这就是允许打包器如 Rollup 或 Webpack 进行“树摇”的原因——如果你从未使用过导出函数，它可以从最终包中完全删除，使你的代码更小。你无法用`require()`的动态特性可靠地做到这一点。

+   **模块缓存是不同的**。ESM 加载器仍然有一个缓存（在内部称为模块映射），但与`require.cache`不同，它不是一个公开的 API 你可以随意操作。你不能简单地深入其中并删除一个模块来强制重新加载。这对稳定性是个好事，但它剥夺了我们在 CJS 世界中使用（并滥用）的“高级用户”技巧。

说实话，生态系统仍在过渡中。你会遇到只支持`require`的包，你将不得不使用动态的`import()`语句来加载它们，这感觉像是退步了一步。但方向是明确的。ESM 的静态特性和以异步为首要的方法更适合构建大型、复杂和性能良好的应用程序。它解决了 CommonJS 的基本设计缺陷，虽然迁移过程很痛苦，但它为 Node 搭建了一个更加智能的未来。

## 进程引导模式

好吧，一旦 Node 完成了它的事情，最终将控制权交给你的主应用程序文件，下一个阶段就开始了：你应用程序自己的引导。这是你加载配置、连接到数据库、设置你的 Web 服务器，并准备实际工作的地方。这部分完全取决于你，这意味着它是一个既能取得巨大成功又能犯下巨大错误的好地方。

服务器引导的典型模式看起来像这样。坦白说，我们所有人都写过这样的代码。不要评判我，但当我想要快速编写脚本或创建一个测试 http 服务器时（嗯，为了测试目的），我经常这样做。

```js
// A common, but seriously flawed, bootstrap pattern
console.log("Process starting...");
 // 1\. Load configuration
const config = require("./config"); // Sync I/O
 // 2\. Initialize services
const database = require("./database"); // More sync I/O
const logger = require("./logger");
 // 3\. Connect to the database
database
 .connect(config.db)
 .then(() => {
 console.log("Database connected.");
 // 4\. Set up the web server
 const app = require("./app"); // Loads Express, routes, etc.
 const server = app.listen(config.port, () => {
 console.log(`Server listening on port ${config.port}`);
 });
 // ... some shutdown logic ...
 })
 .catch((err) => {
 console.error("Bootstrap failed. Bailing out.", err);
 process.exit(1);
 });
```

看起来不错，对吧？但它隐藏了一些糟糕的反模式。

+   所有那些顶层`require()`调用都是同步的。如果`./config`或`./database`做任何事情甚至稍微复杂一点，它们都会阻塞整个启动。我们之前已经看到这导致了 45 秒的启动时间。

+   `database.connect()`调用是异步的，这是好的。但如果数据库关闭了怎么办？进程记录一个错误并以代码 1 退出。在 Kubernetes 中，这会立即触发重启。所以你的应用程序又开始了，试图连接到仍然关闭的数据库，失败，退出，然后重启。你刚刚创建了一个`CrashLoopBackOff`，它在猛击你那可怜的数据库。

+   你`require`东西的顺序开始变得重要。如果`./app`需要数据库模块但你还没有连接，你可能会遇到一些非常奇怪的竞争条件。

### 更好的方法 - 异步初始化器

一个更稳健的模式是将所有启动逻辑包裹在一个显式的`async`函数中。

```js
// A much more robust bootstrap pattern
class Application {
 constructor() {
 this.config = null;
 this.db = null;
 this.server = null;
 }
 async start() {
 console.log("Starting application bootstrap...");
 try {
 // Load config. Still sync, but keep it minimal.
 this.config = require("./config");
 // Asynchronously initialize all I/O dependencies.
 // You can even run these in parallel with Promise.all if they're independent.
 console.log("Connecting to database...");
 this.db = require("./database");
 await this.db.connect(this.config.db, { retries: 5, delay: 1000 }); // Resilience! But not the preferred way. Check the note below
 console.log("Database connected.");
 // Start the server, but only after dependencies are ready.
 // Use dependency injection! Pass the DB connection to the app.
 const app = require("./app")(this.db);
 this.server = app.listen(this.config.port);
 // Wait for the 'listening' event. This is the true "ready" signal.
 await new Promise((resolve) => this.server.on("listening", resolve));
 console.log(`Server is ready and listening on port ${this.config.port}.`);
 } catch (error) {
 console.error("FATAL: Application failed to start.", error);
 // Try to clean up whatever might have started.
 await this.stop();
 process.exit(1);
 }
 }
 async stop() {
 // ... shutdown logic goes here ...
 }
}
 // And in your main entry point:
const app = new Application();
app.start();
```

🚨注意

而不是仅仅在循环中重试，添加适当的退避 - 让每次重试等待的时间比上一次长，并添加一些随机性（抖动），这样就不会有一堆进程同时重试。使用已经做得很好的库。同时，确保你正在重试的东西可以安全地运行多次，或者用锁来保护它。如果服务一直处于关闭状态，那么有一个断路器或健康检查，这样你就不会无休止地猛击它。

这样要好得多。

+   **明确性**。启动逻辑全部集中在一个地方，发生什么以及顺序如何一目了然。

+   **弹性**。数据库连接现在有了重试机制。如果出现短暂的网络中断，它不会立即崩溃。

+   **可测试性**。通过将数据库连接等依赖项注入到应用程序中（`依赖注入`），你使得你的代码在隔离状态下更容易测试。

+   **诚实性**。应用程序只有在服务器实际上正在监听连接后才会将自己视为“已启动”。这是一个发送给 Kubernetes 的更可靠的信号。

引导启动不仅仅是让服务器运行起来。它是关于以可预测、弹性、可观察的方式运行。你在这里花费的每一秒都是你的服务关闭的一秒。优化它不是一个“锦上添花”的事情——它是生产级应用程序的一个标志。

## 信号处理和进程通信

好的，你的应用程序已经引导启动并运行。但它们不能永远运行。最终，某个东西——一个开发者、一个部署脚本、一个容器编排器——会告诉它停止。而这个对话是通过**信号**来进行的。

这是一些老式的 Unix 东西，但对于编写不会直接崩溃和燃烧的服务来说，这是绝对不可协商的。

信号基本上是操作系统发送给你的进程的软件中断。当你你在终端中按 `Ctrl+C` 时，你正在发送 `SIGINT`（信号中断）信号。当 Kubernetes 想要关闭你的 pod 时，它会发送 `SIGTERM`（信号终止）。

这些是你实际上需要关注的信号：

+   **`SIGINT`** - 来自 `Ctrl+C` 的“中断”信号。

+   **`SIGTERM`** - “请优雅地终止”信号。这是编排器如 Kubernetes 使用的信号。**这是你的主要关闭信号**。

+   **`SIGHUP`** - “挂起”信号。守护进程通常使用这个信号来触发配置重新加载，而无需完全重启。

+   **`SIGKILL`** - 杀死信号。这个信号不能被捕获或忽略。操作系统会立即终止你的进程。没有清理，没有遗言。这是 Kubernetes 在你的进程长时间忽略 `SIGTERM` 时发送的信号。

+   **`SIGUSR1` / `SIGUSR2`** - 用户定义的信号。你可以用这些信号做任何你想做的事情，比如触发堆转储或按需清除缓存。

⚠️警告

为了创建可靠、跨平台的关闭逻辑，你应该只处理 `SIGINT` 和 `SIGTERM` 信号，因为像 `SIGUSR1` 这样的信号在 Windows 上不受支持。为了最大兼容性，特别是在 Windows 服务中，还应通过创建显式的程序触发器来补充，例如 IPC 消息。

在 Node 中，你可以在 `process` 对象上监听信号，它是一个 `EventEmitter` -

```js
console.log(`My PID is: ${process.pid}`);
 // This is the one you absolutely must handle in production.
process.on("SIGTERM", () => {
 console.log("Received SIGTERM. I should start shutting down now...");
 // ... kick off your graceful shutdown logic ...
 process.exit(0);
});
 // This is for local development (Ctrl+C).
process.on("SIGINT", () => {
 console.log("Received SIGINT. Cleaning up and getting out of here...");
 // ... maybe a faster shutdown for dev ...
});
 // A custom signal for debugging.
process.on("SIGUSR2", () => {
 console.log("Received SIGUSR2\. Dumping some state...");
 // ... print debug info ...
});
 // Let's keep the process alive so we can send it signals.
setInterval(() => {}, 1000);
```

你可以测试这个。运行脚本，获取其 PID，然后在另一个终端中运行 `kill -s SIGTERM <PID>`。

🚨注意

如果当前信号处理程序没有调用 `process.exit()`，则进程在收到 `SIGINT` (`Ctrl + C`) 或 `SIGTERM` 信号时不会终止。使用 `Ctrl + Z` 发送 `SIGTSTP` 信号来挂起其执行。

### 信号处理问题

你可能一直在想信号处理很简单，但现在你进入了混乱的真实世界。你花了一整天的时间调试一个服务，它就是无法干净地关闭。你有一个 `SIGTERM` 处理程序，但它似乎永远不会触发。进程在 30 秒（或 `terminationGracePeriodSeconds`）后消失，你知道这意味着什么：它正在被 `SIGKILL` 杀死。

你甚至从哪里开始查找？

经过数小时的挖掘，你找到了罪魁祸首：一个第三方指标库。你发现它有自己的关闭逻辑，并且在初始化时，它注册了自己的 *自己的* `SIGTERM` 处理程序。更糟糕的是，你发现它在添加自己的处理程序之前做了相当于 `process.removeAllListeners('SIGTERM')` 的事情。它完全摧毁了你的关闭逻辑，甚至没有发出警告。

**这就是你打破一个重大误解的地方 -** 你的 `process.on('SIGTERM', ...)` 处理程序并不是神圣不可侵犯的。你现在意识到，你的任何依赖项都可能正在干扰它。你决定，一个真正健壮的系统必须最后注册其关键信号处理程序，或者有一个中央关闭管理器，所有其他组件都连接到它。

ℹ️注意

然而，如果没有人移除监听器，`process.on` 简单地 **添加** 另一个处理程序到队列中。当信号到达时，**所有注册的处理程序都将按添加的顺序触发**。真正的危险是，一个库 *移除* 其他监听器。你不应该盲目地信任 `node_modules` 中的所有内容。

所以你修复了它。但现在你必须非常小心你在信号处理程序中做的事情 *内部*。不要试图直接在那里执行复杂的异步操作。你应该意识到最好的模式就是仅仅使用信号来切换开关，并让你的主应用程序逻辑处理实际的关闭序列。

那么，这个更安全的模式看起来是什么样子？这可能是你最终得到的结果 -

```js
// A much safer pattern we may come up for signal handling
let isShuttingDown = false;
 function gracefulShutdown() {
 if (isShuttingDown) {
 // Already shutting down, don't start again.
 return;
 }
 isShuttingDown = true;
 console.log("Shutdown initiated. Draining requests...");
 // 1\. You stop taking new requests.
 server.close(async () => {
 console.log("Server closed.");
 // 2\. Now you close the database.
 await database.close();
 console.log("Database closed.");
 // 3\. All clean. You exit peacefully.
 process.exit(0);
 // or even better -> process.exitCode = 0
 });
 // A safety net. If you're still here in 10 seconds, something is wrong.
 setTimeout(() => {
 console.error("Graceful shutdown timed out. Forcing exit.");
 process.exit(1);
 }, 10000);
}
 process.on("SIGTERM", gracefulShutdown);
process.on("SIGINT", gracefulShutdown);
```

看看这有多健壮。你的信号处理程序的唯一任务是调用 `gracefulShutdown`。然后该函数为你管理状态和序列。最重要的是，你添加了一个超时。这是你的安全回退机制。它防止进程永远挂起，并确保 **你** 在 `SIGKILL` 锤子落下之前以自己的方式退出。

⚠️警告

有一个控制关闭的地方比让每个模块添加自己的信号处理程序要好。把它想象成一个关闭管理器或事件总线，所有组件都会注册到那里。这样，你可以确保重要的处理程序总是运行。你甚至可以在自己的辅助函数中包装 `process.on`，这样随机的库就不能干扰它。注意监听器的数量，如果有什么东西移除了它们，就记录下来。不要依赖于你注册处理程序的顺序 - 那会带来麻烦。

## 优雅关闭

优雅的关闭是指有控制的程序终止，确保所有进行中的任务都已完成，数据完整性得到维护，并且活动连接被正确关闭。它应该作为引导过程的逆过程，应该有系统地释放资源以防止数据损坏等错误。

核心思想是状态转换 - `接受流量 -> 排空 -> 关闭`。

1.  你应该做的第一件事是**停止接受新的工作**。你锁上了前门。对于一个 Web 服务器来说，这是`server.close()`。这告诉服务器停止接受新的连接。它**不会**终止现有的连接；那些连接被允许完成它们正在做的事情。

1.  然后你**完成正在进行的任务（排空）**。这是最关键的一步，也是大家最容易出错的地方。你的应用程序必须等待它目前正在做的所有事情完成。这可能是一个 HTTP 请求、一个数据库事务、来自队列的消息，等等。跟踪这个“正在进行的”工作是最困难的部分。对于 Web 服务器，`server.close()`中的回调有所帮助，但它只能告诉你 TCP 连接何时关闭，而不是你的应用程序逻辑对该请求已完成。你通常需要实现这一点，而且可能会相当困难。

1.  最后一步是**清理资源**。一旦你确定没有更多的工作在进行，你就可以开始拆解东西了。关闭你的数据库连接池。从 Redis/Valkey 或 RabbitMQ 断开连接。清除你的日志。这里的顺序至关重要——你不能在请求仍在尝试使用它时关闭数据库连接。这就是为什么清理总是在排空之后进行。

1.  最后，在清理完一切之后，进程可以安全地以代码`0`退出，告诉世界这是一个成功且干净的关闭。`process.exit(0)`。

下面是这个流程 -

下面是一个更好的关闭管理类的例子 -

```js
// A stateful shutdown manager
class ShutdownManager {
 constructor(server, db) {
 this.server = server;
 this.db = db;
 this.isShuttingDown = false;
 this.SHUTDOWN_TIMEOUT_MS = 15_000;
 process.on("SIGTERM", () => this.gracefulShutdown("SIGTERM"));
 process.on("SIGINT", () => this.gracefulShutdown("SIGINT"));
 }
 async gracefulShutdown(signal) {
 if (this.isShuttingDown) return;
 this.isShuttingDown = true;
 console.log(`Received ${signal}. Starting graceful shutdown.`);
 // A timeout to prevent hanging forever.
 const timeout = setTimeout(() => {
 console.error("Shutdown timed out. Forcing exit.");
 process.exit(1);
 }, this.SHUTDOWN_TIMEOUT_MS);
 try {
 // 1\. Stop the server
 await new Promise((resolve, reject) => {
 this.server.close((err) => {
 if (err) return reject(err);
 console.log("HTTP server closed.");
 resolve();
 });
 });
 // 2\. In a real app, you'd wait for in-flight requests here.
 // 3\. Close the database
 if (this.db) {
 await this.db.close();
 console.log("Database connection pool closed.");
 }
 console.log("Graceful shutdown complete.");
 clearTimeout(timeout);
 process.exit(0);
 } catch (error) {
 console.error("Error during graceful shutdown:", error);
 clearTimeout(timeout);
 process.exit(1);
 }
 }
}
 // How you'd use it:
// new ShutdownManager(server, db);
```

`process.exit()`**不是**一个干净的关闭。它是核选项。它是一种立即的、强制性的终止。事件循环停止。任何挂起的异步工作都被放弃。任何缓冲区中的数据都永远消失。它是软件上拔掉电源插头的等效操作。它**应该**只在优雅关闭序列的末尾调用，在你确认一切都很干净之后。使用它来“只是退出”是你丢失数据的方式。就是这样。

## 处理和资源管理

你有没有遇到过 Node 进程就是...不会死的情况？你已经关闭了服务器，你以为一切都已经完成，但进程仍然挂在那里，直到你再次按下`Ctrl+C`或者它被`SIGKILL`终止。

原因几乎总是“把手”泄漏。

我有一个 API 服务器，部署了几次之后，开始出现令人恐惧的`Error: EMFILE: too many open files`错误。在服务器上快速运行`lsof`命令显示，进程有数万个打开的文件描述符，大多数是网络套接字处于`CLOSE_WAIT`状态。我的关闭逻辑是关闭服务器，但某些东西正在泄漏套接字。进程会挂起，被`SIGKILL`终止，操作系统留下清理套接字。快速重启部署产生了这些即将死亡套接字的队列，最终耗尽了系统的文件描述符限制。

那么“句柄”到底是什么？就像我之前说的，这是一个 libuv 的东西。把它想象成一个表示长期 I/O 资源的对象。一个活跃的服务器、一个套接字、一个定时器（`setTimeout`或`setInterval`）、一个子进程——这些都是由句柄支持的。

默认情况下，这些句柄是“引用的”。一个引用的句柄是在告诉事件循环，“嘿，我还在做事情，你敢退出。”进程只有在没有更多引用句柄时才会自行优雅退出。

检查这个简单的服务器 -

```js
// This process will never exit.
// The setInterval creates a referenced handle that keeps it alive forever.
setInterval(() => {
 console.log("Still here...");
}, 1000);
```

但你也可以“取消引用”一个句柄。你可以调用`.unref()`。这告诉事件循环，“即使我还在运行，你也可以退出。我只是一个后台任务，不用等待我。”

```js
// This process WILL exit immediately.
const timer = setInterval(() => {
 // This will never even run.
 console.log("You won't see me.");
}, 1000);
 timer.unref();
```

这个`ref()`/`unref()`机制是关键。在我们的`EMFILE`问题中，我们泄漏了引用的套接字句柄。它们使进程保持活跃，这导致了`SIGKILL`，进而导致了资源泄漏。

这里有一个小片段，如果客户端连接保持活跃，则在关闭时将会挂起。你可以将其复制粘贴到文件中，并用`node server.js`运行它。

### 让我们创建泄漏的句柄

这个服务器将跟踪所有活跃的连接。当它收到`SIGTERM`信号时，它将尝试优雅地关闭，但在 5 秒超时后将强制销毁任何挂起的连接。

```js
const http = require("http");
 const PORT = 8080;
// We'll use a Set to keep track of all active socket connections.
const activeSockets = new Set();
 const server = http.createServer((req, res) => {
 console.log("🔌 Client connected!");
 // Keep the connection alive for 20 seconds before responding.
 // This gives us plenty of time to send the shutdown signal.
 setTimeout(() => {
 res.writeHead(200, { "Content-Type": "text/plain" });
 res.end("Hello from the slow server!\n");
 console.log("✅ Response sent to client.");
 }, 20000);
});
 // When a new connection is established, add its socket to our Set.
server.on("connection", (socket) => {
 activeSockets.add(socket);
 // When the socket closes, remove it from the Set.
 socket.on("close", () => {
 activeSockets.delete(socket);
 });
});
 server.listen(PORT, () => {
 console.log(`🚀 Server started on port ${PORT} with PID: ${process.pid}`);
 console.log('   Run "curl http://localhost:8080" in another terminal to connect.');
 console.log(`   Then, run "kill ${process.pid}" to send the shutdown signal.`);
});
 // Here's the graceful shutdown logic.
function shutdown() {
 console.log("\n🛑 SIGTERM signal received: closing HTTP server...");
 // 1\. Stop accepting new connections.
 server.close((err) => {
 if (err) {
 console.error(err);
 process.exit(1);
 }
 // 4\. If server.close() callback fires, all connections were closed gracefully.
 console.log("✅ All connections closed. Server shut down successfully.");
 process.exit(0);
 });
 // 2\. The server is no longer accepting connections, but existing ones might still be open.
 //    If connections don't close within 5 seconds, destroy them forcefully.
 setTimeout(() => {
 console.error("💥 Could not close connections in time, forcefully shutting down!");
 // 3\. Destroy all remaining active sockets.
 for (const socket of activeSockets) {
 socket.destroy();
 }
 }, 5000); // 5-second timeout
}
 // Listen for the SIGTERM signal. `kill <PID>` sends this by default.
process.on("SIGTERM", shutdown);
// Listen for Ctrl+C in the terminal.
process.on("SIGINT", shutdown);
```

打开第一个终端并运行脚本。注意它打印的**进程 ID (PID)**。

```js
node server.js
 🚀 Server started on port 8080 with PID: 54321 <-- REMEMBER THIS
 Run "curl http://localhost:8080" in another terminal to connect.
 Then, run "kill 54321" to send the shutdown signal.
```

现在，打开一个**第二个终端**并使用`curl`连接到服务器。由于我们添加了 20 秒的延迟，这个命令将会挂起，保持套接字连接打开。

```js
curl http://localhost:8080
```

`curl`命令现在将等待。当`curl`仍在等待时，打开第三个终端并使用前一个输出中的 PID 运行`kill`命令。

```js
# Replace 54321 with the actual PID of your server process
kill 54321
```

现在，看看你的**第一个终端**（服务器正在运行的终端）。你会按以下顺序看到以下情况 -

服务器立即打印 -

```js
🛑 SIGTERM signal received: closing HTTP server...
```

到目前为止，已经调用了`server.close()`，但由于`curl`连接仍然活跃，进程**不会退出**。**等待 5 秒...** 我们关闭逻辑中的`setTimeout`将会触发，因为`curl`连接阻止了优雅的退出。你会看到：

```js
💥 Could not close connections in time, forcefully shutting down!
```

脚本现在销毁了挂起的套接字，最终允许进程终止。你另一个终端中的`curl`命令可能会因为错误而失败，例如`curl: (56) Recv failure: Connection reset by peer`。

ℹ️注意

对于 Node 版本 v18 或更高版本的应用程序，你可以简化服务器关闭过程。而不是手动跟踪套接字，考虑使用内置的 `server.closeAllConnections()` 或 `server.closeIdleConnections()` 方法。这些提供了更安全和更直接的方式来主动关闭保持连接的套接字。仍然重要的是要将此与应用程序级别的排空相结合，以优雅地处理任何正在进行的请求。

### 调试处理泄漏

Node 有一个用于调试的未记录功能：`process._getActiveHandles()`。它返回一个数组，包含目前维持你的进程活跃的所有内容。

🚨 警告

`process._getActiveHandles()` 函数是 Node API 的一个内部部分。它的行为可能会改变，或者在未来版本中可能没有任何通知就被完全移除。它仅应用于调试，并且不适合生产代码。还有其他替代的包，如 `wtfnode`，你可以使用。

```js
const net = require("net");
 function printActiveHandles() {
 console.log("--- Active Handles ---");
 process._getActiveHandles().forEach((handle) => {
 console.log(`Type: ${handle.constructor.name}`);
 });
 console.log("----------------------");
}
 console.log("Initial state:");
printActiveHandles();
 const server = net.createServer(() => {}).listen(8080);
console.log("\nAfter creating server:");
printActiveHandles();
 const timer = setInterval(() => {}, 5000);
console.log("\nAfter creating timer:");
printActiveHandles();
 // Now let's clean up
server.close();
clearInterval(timer);
console.log("\nAfter cleaning up:");
// Give the event loop a tick to process the close events
setTimeout(printActiveHandles, 100);
```

运行这个，你会看到一个 `Server` 处理器出现然后消失。你可能会注意到 `setInterval` 并没有向列表中添加可见的 `TIMER` 处理器 - 这是因为现代 Node 对定时器的管理进行了优化。但不要被骗；那个定时器仍然在后台活跃，防止你的应用程序关闭。关键是要观察你在调用 `.close()` 之后 `Server` 处理器的消失。如果你的进程在退出时挂起，只需在你认为它应该退出之前调用 `printActiveHandles()`。它将告诉你你忘记关闭了什么。

ℹ️ 备注

Node 并不会自动为你清理资源。垃圾回收器确实会清理内存，但它对文件描述符或网络套接字一无所知。如果你打开一个文件，你必须关闭它。如果你创建了一个服务器，你必须调用 `.close()`。忘记做这些是处理泄漏的主要原因。

## 内存生命周期和堆

Node 进程的内存使用不是一个单一的数字。它是一个活生生的、有呼吸的东西。了解它是如何增长和缩小的，这是诊断内存泄漏并阻止你的应用程序被 OOM 杀死的途径。当你的进程启动时，它的内存使用（常驻集大小，或 RSS，这是操作系统实际看到的）会迅速上升。这归因于以下原因 -

第一个原因是 **V8 堆初始化**。V8 立即为堆抓取一大块内存。第二个原因是 **模块加载**。当你使用 `require()` 加载文件时，它们的代码会被读取、编译并填充到内存中。`require.cache` 保留了你所加载的每个模块。对于大型应用程序，这个缓存本身就可以轻易达到 100-500MB。这基本上是业务运营的固定成本。

启动时的内存增长看起来像是一个陡峭的斜坡 -

```js
Memory (RSS)
 ^
 |
 |      +-------------------------> Phase 2: Operational Plateau
 |     /
 |    /  <-- Module Cache Growth
 |   /
 |  /   <-- V8 Heap Init
 +-------------------------------------> Time
 ^
 Process Start
```

你可以自己查看。只需在你大的 `require` 语句前后记录 `process.memoryUsage()`。跳跃将很明显。

一旦您的服务器开始运行并处理请求，其内存使用就会进入一个模式。每个请求都会创建新的对象，导致`heapUsed`增加。定期地，V8 的垃圾回收器（GC）运行并清理旧的、未引用的对象。在 GC 运行后，`heapUsed`会降回。在一个健康的应用程序中，这看起来像锯齿状的模式。当您工作时，它会上升，然后下降。内存泄漏是指锯齿形的谷底随着时间的推移而不断升高。GC 正在运行，但它无法释放您意外保留的一些内存。

这里是每个人都容易出错的地方 - “外部”内存。这是在 V8 堆之外分配的内存，最常见的是通过`Buffer`对象。当您将大文件读入`Buffer`时，该内存不是 V8 堆的一部分。

这很重要，因为您的 V8 堆可能看起来完全正常，远远低于其限制，但您的进程的总 RSS 可能因为缓冲区而非常大。这可能导致 OOM 杀戮，如果您只查看 V8 堆快照，那么调试起来会非常困惑。您必须记住，您的进程的内存不仅仅是 V8 堆。

## 退出代码和进程状态

当您的进程最终终止时，它会向启动它的任何东西（您的 shell、脚本、Kubernetes）返回一个**退出代码**。这个小整数是它的最终状态报告。正确使用它们是构建不会默默失败的系统的重要组成部分。惯例很简单：`0`表示成功。其他任何东西都表示失败。

Node 有几个内置的退出代码，但最重要的是`1`，这是未捕获异常的默认值。您可以通过两种方式控制退出代码：

1.  **`process.exit(code)`** - 这是一种不好的方法。正如我们之前提到的，这是一种突然终止。除非您处于火灾中，否则不要使用它。

1.  **`process.exitCode = code`** - 这是正确的方法。这只是一个您设置的属性。它不会立即执行任何操作。它只是告诉 Node，“嘿，无论何时您完成并优雅地退出，请使用此退出代码。”

ℹ️注意

避免在服务器和长时间运行的服务中调用`process.exit()`，因为它会强制立即终止并可能跳过异步清理；首选`process.exitCode` + 优雅地处理关闭。但是，对于短期 CLI 工具或致命的早期启动失败，其中没有初始化其他任何内容，`process.exit()`是可以接受的，并且是首选的关闭方式。

这让您可以将清理逻辑与状态报告分离。

```js
async function gracefulShutdown(error) {
 // ... do all your cleanup ...
 if (error) {
 console.error("Shutting down because of an error:", error);
 process.exitCode = 1;
 } else {
 console.log("Shutdown completed successfully.");
 process.exitCode = 0;
 }
 // Now, we just let the event loop empty. No need for process.exit()!
 // Node will exit on its own once all handles are closed.
}
```

### 为什么退出代码在生产中如此重要？

容器编排器如 Kubernetes 的生存和死亡都取决于退出代码。当容器退出时，Kubernetes 会检查代码。如果它不是零，它假定失败，并根据您的`restartPolicy`重新启动容器。如果代码是`0`，它假定进程有意完成其工作，可能不会重新启动它。

您可以创建自己的应用程序特定的退出代码，使调试变得容易一千倍。例如 -

+   `70`: 启动时数据库连接失败。

+   `71`: 无效配置文件。

+   `72`: 无法绑定到所需的端口。现在，当你的服务无法启动时，退出代码 `70` 的警报会立即告诉查看它的人“这是一个数据库问题。”他们不必浪费时间挖掘日志来找出这一点。

ℹ️注意

忽略退出代码就像告诉你的基础设施你无法区分成功和五级火灾。一个无法连接到数据库但以代码 `0` 退出的进程会欺骗 Kubernetes 认为一切正常。这会导致无声的失败，直到你的客户开始尖叫你才会发现。使用有意义的退出代码是不可或缺的。

## 子进程和集群生命周期

ℹ️注意

如果你一生中从未创建过子进程，那也无可厚非。我们将在后面的章节中深入探讨子进程、工作线程和集群。请耐心等待。

到目前为止，我们讨论的是单个进程。但为了真正使用多核服务器，你可能正在使用 `cluster` 模块或通过 `child_process` 启动工作进程。现在你不仅是一个进程管理器；你是一个父亲。你对你孩子负责。

### `cluster` 模块

`cluster` 模块是进行多进程 Node 的标准方式。主进程不处理请求；它的任务是管理工作进程。这包括协调优雅的关闭。主进程接收 `SIGTERM` 信号。主进程不会退出。相反，它会通过调用 `worker.disconnect()` 告诉每个工作进程优雅地关闭。每个工作进程都会收到一个 `disconnect` 事件并触发它自己的优雅关闭逻辑（停止服务器，排空请求等）。

主进程监听来自每个工作进程的 `exit` 事件。只有当 *所有* 工作进程都已退出时，主进程才会最终清理并退出自己。这防止了“雷鸣般的群体”问题，即你一次性杀死所有进程并丢弃每个活动连接。

### `child_process` 和孤儿问题

当你使用 `child_process.spawn()` 或 `fork()` 时，你 100%负责该子进程的生命。

ℹ️注意

子进程在其父进程死亡时不会自动死亡。如果你只是 `SIGKILL` 父进程，其子进程将成为“孤儿”。它们会被系统 `init` 进程（PID 1）收养并继续运行，可能会消耗资源。

负责任的父进程 *必须* 在退出前清理其子进程。父进程的 `SIGTERM` 处理程序需要了解所有其活动子进程。它需要遍历它们并向每个发送 `SIGTERM`。它需要 *等待* 所有子进程退出后才能继续自己的关闭。

```js
// Being a responsible parent process
const { spawn } = require("child_process");
const children = [];
 const child = spawn("node", ["cool-lil-script.js"]);
children.push(child);
 process.on("SIGTERM", () => {
 console.log("Parent got SIGTERM. Telling children to shut down...");
 children.forEach((child) => {
 child.kill("SIGTERM"); // Pass the signal on
 });
 // Let's just wait for all children to exit before the parent exits
 Promise.all(children.map((c) => new Promise((resolve) => c.on("close", resolve)))).then(() => {
 console.log("All children are gone. Parent exiting.");
 process.exit(0);
 });
});
```

没有这样做会导致资源泄漏的巨大来源。

⚠️警告

管理你的子进程。这不是边缘情况；这是一个要求和责任。

## 调试进程问题

当事情出错时 - 启动缓慢、内存泄漏、挂起进程 - `console.log` 是不够的。你需要一个更好的工具包。以下是我当进程行为异常时使用的工具。

**问题** - 服务启动需要很长时间。**我的首选工具** - `node --cpu-prof --cpu-prof-name=startup.cpuprofile server.js`。这会生成你的启动 V8 CPU 配置文件。你可以将 `startup.cpuprofile` 文件拖到 Chrome DevTools（性能标签页）中，得到一个漂亮的火焰图，它显示了哪些函数一直在消耗时间。这就是我发现一个验证库在启动时同步编译了数百个模式，增加了我们启动时间 5 秒的原因。**另一个首选工具** - `node --trace-sync-io server.js`。正如我之前提到的，这是找到阻塞 I/O 的最佳方式，这几乎总是 `node_modules` 深处的 `require()` 调用。

**问题** - 内存使用量持续上升。**我的首选工具** - 堆快照。

```js
const v8 = require("node:v8");
// Hook this up to a signal for on-demand snapshots in production.
process.on("SIGUSR2", () => {
 const filename = v8.getHeapSnapshot();
 console.log(`Heap snapshot written to ${filename}`);
});
```

将 `.heapsnapshot` 文件加载到 Chrome DevTools（内存标签页）。"比较" 视图是纯金。当应用启动时拍一张快照，然后在它运行一段时间后再次拍一张，并比较它们。这将显示正在创建且从未释放的确切对象类型。这就是我们找到 `require.cache` 泄漏的方法。

**问题** - 你的进程不会优雅地退出。**我的首选工具** - `process._getActiveHandles()`。在你的关闭逻辑中调用此工具，以查看哪些 libuv 资源仍然打开并保持事件循环活跃。**真相大白** - `lsof -p <PID>`。这个操作系统级别的工具（"列出打开的文件"）显示了你的进程打开的每个文件描述符 - 网络套接字、文件，等等。这是我们诊断我们的 `EMFILE` 问题的方式。

**问题** - 进程突然死亡。**你的最后一道防线** - `process.on('uncaughtException', ...)` 和 `process.on('unhandledRejection', ...)`。你必须为这些设置处理程序。它们唯一的任务是尽可能详细地记录错误，然后优雅地关闭。**绝对不要在未捕获的异常后尝试继续运行**。应用程序处于未知、可能已损坏的状态。只需记录并关闭。

好吧，这有很多。这里有一些你可以随时使用的 DOs 和 DONTs - 如果你想的话，随时可以添加更多。

### **要做的**

+   **分析启动时间**。真的。不要猜测。使用 `--cpu-prof`。

+   **延迟加载大型模块**。如果一个端点很少使用，在处理程序内部而不是在文件顶部 `require()` 它的依赖项。

+   **实现真正的优雅关闭**。处理 `SIGTERM`，停止接受新工作，等待旧工作完成，然后清理。按照这个顺序。

+   **跟踪所有资源**。每个 `createServer` 或 `connect` 都需要在你的关闭逻辑中有一个相应的 `close` 或 `disconnect`。

+   **使用有意义的退出代码**。`0` 表示成功，非零表示失败。使它们具体。你的值班工程师会感谢你的。

+   **做一个好家长**。如果你生成子进程，你负责在关闭时终止它们。

### **不要做的事情**

+   **不要在启动时阻塞事件循环**。不要在顶层进行同步 I/O 或重负载 CPU 工作。

+   **不要使用 `process.exit()` 来关闭。** 这不是优雅的。这是一个车祸。使用 `process.exitCode` 并让进程自然退出。

+   **不要假设 `require()` 是免费的。** 它会消耗 CPU 时间和内存。永远不要在 `require()` 调用中使用动态变量。

+   **不要忽视信号。** 如果你没有处理 `SIGTERM` 信号，Kubernetes 将在 30 秒或 `terminationGracePeriodSeconds` 设置的时间后直接杀死你的进程。

+   **不要信任第三方库。** 它们可能会泄露句柄或干扰你的信号处理器。验证它们的行为。

+   **不要忽视未捕获的异常。** 它们是致命的。记录它们并立即关闭。

### 生产安全检查清单

在我批准任何新服务的 PR 之前，我会问这些问题 -

+   你实际测量过启动时间了吗？

+   我们对于模块（打包、懒加载）有策略吗？

+   是否有健壮的 `SIGTERM` 和 `SIGINT` 处理器？

+   你能证明在关闭期间你打开的每个资源都被关闭了吗？

+   进程是否以正确的代码退出，以区分成功和不同的失败？

+   如果你创建了子进程，你绝对确定你已经清理它们了吗？

## 关闭 - 尊重进程生命周期

我们喜欢关注那些性感的东西——聪明的算法、流畅的 API 设计。我们把运行我们代码的过程当作这个无聊的、黑盒，它只是正常工作。

但正如我们所看到的，这个盒子有自己的生命。它诞生于 C++ 和系统调用的风暴中，它通过吞噬代码和内存而成长，它通过事件循环的节奏而生存，它最终必须死亡。你美丽的代码可能存在一段时间，但最终，地面会移动，一切都会崩溃。数据会被损坏。服务会中断。客户会生气。

尊重进程生命周期意味着将你的应用程序视为一个动态的、有生命的实体，而不是一个静态的脚本。这意味着要考虑它的诞生（快速启动）、它的生命（弹性运行）和它的死亡（干净关闭）。这是从“只是运行我的代码”到“管理这个进程”的转变。

诚实地讲，这种转变是区分初级开发人员和高级工程师的关键。它是所有健壮、可靠、生产就绪的系统的基础。
