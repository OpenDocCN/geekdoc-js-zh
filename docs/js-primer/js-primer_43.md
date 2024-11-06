# 用例：Node.js CLI 应用程序

> 原文：[`jsprimer.net/use-case/nodecli/`](https://jsprimer.net/use-case/nodecli/)

在这里，我们将开发一个在 Node.js 中使用 CLI（命令行界面）的应用程序。 作为 CLI 的用例，我们将创建一个将 Markdown 格式的文本文件转换为 HTML 文本的工具。

所创建的应用程序应满足以下要求：

+   接收转换目标文件路径作为命令行参数

+   读取 Markdown 格式的文件，并将转换后的 HTML 显示在标准输出中

+   可以通过命令行参数提供转换设置选项

## [](#summary)*目录*

*### [](#helloworld)*Node.jsでHello World*

*通过 Hello World 应用程序学习 Node.js CLI 应用程序的基础知识。

### [](#argument-parse)*处理命令行参数*

*学习如何接收命令行参数并将其解析为应用程序易于使用的形式。

### [](#read-file)*读取文件*

*学习使用 Node.js 的`fs`模块来读取文件。

### [](#md-to-html)*将 Markdown 转换为 HTML*

*使用 marked 包将 Markdown 文件转换为 HTML。

### [](#refactor-and-unittest)*编写单元测试*

*引入单元测试并对源代码进行重构和模块化。******
