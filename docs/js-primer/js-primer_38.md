# 用例：Ajax 通信

> 原文：[`jsprimer.net/use-case/ajaxapp/`](https://jsprimer.net/use-case/ajaxapp/)

在这里，我们将创建一个应用程序，作为在 Web 浏览器上进行 Ajax 通信的用例，从 GitHub 的用户 ID 中获取个人资料信息。

应用程序需满足以下要求：

+   可以在文本框中输入 GitHub 的用户 ID

+   根据输入的 GitHub 用户 ID 获取用户信息

+   在应用程序中显示获取的用户信息

## [](#summary)*目录*

*### [](#entrypoint)*入口点*

*创建应用程序中最先调用的入口点。

### [](#http-communication)*HTTP 通信*

*使用 Web 标准的 Fetch API 进行 HTTP 通信，并调用 GitHub 的 API。

### [](#display)*显示数据*

*使用 Fetch API 获取数据，构建 HTML 并在浏览器上显示。

### [](#promise)*利用 Promise*

*利用 Promise，整理源代码并处理错误。*****
