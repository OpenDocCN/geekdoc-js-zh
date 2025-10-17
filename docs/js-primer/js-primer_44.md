# Node.js 中的 Hello World

> 原文：[`jsprimer.net/use-case/nodecli/helloworld/`](https://jsprimer.net/use-case/nodecli/helloworld/)

在实际创建应用程序之前，让我们通过 Hello World 应用程序来了解 Node.js CLI 应用程序的基础知识。

## [](#create-project)*创建项目目录*

*在本次创建的 Node.js CLI 应用程序中，我们将处理 JavaScript 和 Markdown 等文件。因此，首先创建一个用于存放这些文件的目录。

在此创建一个名为`nodecli`的新目录。从现在开始，我们将在创建的`nodecli`目录下进行工作。

此外，此项目中创建的文件必须以**UTF-8**编码和**LF**换行符保存。

## [](#hello-world)*Hello World*

*首先，让我们尝试创建一个 Node.js 的 Hello World 应用程序。具体来说，我们将编写一个 CLI 应用程序，当运行时，它将在标准输出中显示字符串`"Hello World!"`。首先要准备的是作为应用程序入口的 JavaScript 文件。在`nodecli`目录中创建一个名为`main.js`的文件，并编写如下内容。

main.js

```
console.log("Hello World!"); 
```

在 Web 浏览器环境中，`console.log`方法的输出目标是浏览器开发者工具的控制台。而在 Node.js 环境中，`console.log`方法的输出目标是标准输出。以下代码将在标准输出中输出字符串`"Hello World!"`。

要在 Node.js 中执行 JavaScript 代码，可以使用`node`命令。在命令行中移动到`nodecli`目录，并使用以下命令使用 Node.js 运行`main.js`。如果尚未准备好`node`命令，请先参阅“准备应用程序开发”章节。

```
# `cd`コマンドでnodecli/ディレクトリに移動する
$ cd nodecli/
# `node`コマンドで`main.js`を実行する
$ node main.js
Hello World! 
```

在 Node.js 中，创建一个作为入口点的 JavaScript 文件，并将该文件作为`node`命令的参数传递以执行是基本操作。此外，就像在 Web 浏览器中的 JavaScript 一样，代码将按顺序从第一行开始执行。

## [](#global-objects)*Node.js 和浏览器的全局对象*

*Node.js 使用了与 Web 浏览器 Chrome 相同的 V8 JavaScript 引擎。因此，基本的 ECMAScript 语法与在浏览器中的使用方式相同。然而，由于浏览器环境和 Node.js 环境具有不同的全局对象，因此在开发应用程序时必须理解这些差异。

ECMAScript 中定义的全局对象存在于浏览器和 Node.js 两个环境中。例如，诸如`Boolean`和`String`之类的包装对象以及诸如`Map`和`Promise`之类的内置对象都存在于这两个环境中。

不过，也存在着因执行环境不同而不同的对象。例如，Web 浏览器环境的全局对象是`window`对象，但在 Node.js 中称为[global](https://nodejs.org/docs/latest-v20.x/api/globals.html)的对象是全局对象。在浏览器的`window`对象中，存在着以下类似的属性和函数。

+   [文档](https://developer.mozilla.org/ja/docs/Web/API/Document)

+   [XMLHttpRequest](https://developer.mozilla.org/ja/docs/Web/API/XMLHttpRequest)

另一方面，Node.js 的`global`对象具有诸如下列之类的属性和函数。

+   [process](https://nodejs.org/docs/latest-v20.x/api/process.html#process_process)

+   [Buffer](https://nodejs.org/docs/latest-v20.x/api/buffer.html)

各种全局对象的属性等可以通过相同的名称作为全局变量或函数进行访问。例如，`window.document`属性也可以作为全局变量的`document`进行访问。

此外，虽然并非由 ECMAScript 定义，但几乎在两个环境中都存在具有相同名称和功能的属性或函数。例如，下面的 API 提供了相同的功能，但方法类型和返回值可能不同。

+   控制台 API

+   `setTimeout`函数

在此基础上，让我们从下一节开始开发 CLI 应用程序。

## [](#section-checklist)*本节的清单*

**   创建了`main.js`文件

+   使用`node`命令执行`main.js`，并确认在标准输出中看到了日志。

+   了解了关于全局对象的情况，包括在 Web 浏览器和 Node.js 中执行环境之间的差异****
