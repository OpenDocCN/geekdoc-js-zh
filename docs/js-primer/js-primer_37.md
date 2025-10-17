# 应用程序开发准备

> 原文：[`jsprimer.net/use-case/setup-local-env/`](https://jsprimer.net/use-case/setup-local-env/)

到目前为止，您已经学到的 JavaScript 基本语法可以在任何执行环境中使用。但在接下来的“用例”章节中，我们将处理具体的执行环境，即网络浏览器和 [Node.js](https://nodejs.org/)。此外，即使是浏览器中执行的应用程序，其开发也离不开 Node.js 这个工具。在本节中，我们将为学习用例所需的应用程序开发环境做准备。

## *Node.js 的安装*

*[Node.js](https://nodejs.org/) 是一种服务器端 JavaScript 执行环境，具有以下特点。

+   使用与 Chrome 相同的 [V8](https://v8.dev/) JavaScript 引擎运行

+   开发是开源的

+   OS 交叉平台运行

Node.js 是为服务器端使用而开发的。但如今，它也被用于命令行工具和 [Electron](https://www.electronjs.org/) 等桌面应用程序。因此，Node.js 不仅限于服务器端，还被广泛用作客户端 JavaScript 执行环境。

Node.js 与许多其他编程语言一样，可以通过在机器上安装执行环境来使用。从官方的 [下载页面](https://nodejs.org/en/download/) 下载适合开发机器的安装程序，并进行安装。

+   下载页面 URL: [`nodejs.org/en/download/`](https://nodejs.org/en/download/)

Node.js 有 LTS（长期支持）版和最新版两个发布版本。LTS（长期支持）版是声明了 2 年维护和支持的版本。具体来说，它包括在不破坏向后兼容性的范围内进行更新，以及提供持续的安全补丁。另一方面，最新版可以使用 Node.js 的最新功能，但只维护最新版本。大多数用户推荐使用 LTS 版。如果您是第一次使用 Node.js 进行开发，请毫不犹豫地下载 LTS 版的安装程序。在本章中，我们将使用当时最新的 LTS 版本 20.11.1 进行开发。

安装完成后，您应该在命令行中可以使用 `node` 命令。执行以下命令以确认已安装的 Node.js 版本（`$` 表示命令行输入栏的符号，因此实际上不需要输入）。

```
$ node -v
v20.11.1 
```

此外，Node.js 还包含 [npm](https://www.npmjs.com/) 这个包管理器。安装 Node.js 后，不仅可以使用 `node` 命令，还可以使用 `npm` 命令来处理 npm。执行以下命令以确认已安装的 npm 版本。

```
$ npm -v
10.2.4 
```

Node.js 和 npm 的版本号采用 `{major}.{minor}.{patch}` 的结构，如果前缀的版本号相同，则保证兼容性。

Node.js 的库几乎都可以通过 npm 进行安装。有关 npm 或 `npm` 命令的详细信息，请参阅 [npm 的官方文档](https://docs.npmjs.com/) 或 [npm 的 GitHub 仓库](https://github.com/npm/cli)。实际上，在“用例”章节中，我们将使用 npm 安装库并使用它。

## *通过 npx 命令执行 npm 包*

*Node.js 的命令行工具有很多公开，通过 npm 安装后可以作为命令执行。顺便说一句，通过 Node.js 的安装，您还可以使用 [npx](https://docs.npmjs.com/cli/v8/commands/npx/) 这个命令。使用 `npx` 命令可以将 npm 公开的可执行包的安装和执行合并。在接下来的用例中，我们将使用 `npx` 命令来使用工具，所以在这里尝试执行工具。

在这里，我们将以[@js-primer/hello-world](https://github.com/js-primer/hello-world)这个示例包为例。要使用 `npx` 命令执行命令行工具，请将包名传递给 `npx` 命令并执行。从 npx 7 开始，第一次执行的命令会提示是否安装包。按下 Enter 键后，安装开始，并执行命令。

```
$ npx @js-primer/hello-world
Need to install the following packages:
  @js-primer/[[email protected]](/cdn-cgi/l/email-protection)
Ok to proceed? (y)
# 初回は@js-primer/hello-worldをインストールしていいかを確認するプロンプトが表示される
# Enterを押すとインストールが開始され、コマンドが実行される

Hello World! 
```

默认情况下，会包含交互式提示符，但通过添加 `--yes` 选项，可以自动执行安装和命令。

```
# --yesオプションで、インストールの確認プロンプトをスキップする
$ npx --yes @js-primer/hello-world
Hello World! 
```

这样，通过使用 `npx` 命令，可以轻松地执行 npm 公开的命令行工具。

### *[专栏] 命令行工具的安装与执行*

*执行 npm 公开的可执行命令的方法不仅仅是 `npx` 命令。您可以使用 `npm install` 命令安装包，并执行已安装包的命令。通常的 `npm install` 命令会将包安装到当前目录，但添加 `--global` 标志可以将包全局安装。全局安装的包的命令可以像 `node` 命令或 `npm` 命令一样，从任意位置执行。

在下面的示例中，全局安装了`@js-primer/hello-world`包。 然后，调用包含的`js-primer-hello-world`命令而无需指定绝对路径。

```
$ npm install --global @js-primer/hello-world
$ js-primer-hello-world
Hello World! 
```

## *本地服务器的设置*

*在“值的评估和显示”章节中，创建了名为`index.html`和`index.js`的文件并在浏览器中显���。 当直接在浏览器中加载本地创建的 HTML 文件时，浏览器的地址栏将显示以`file:///`开头的 URL。 使用`file`模式时，由于[同源策略](https://developer.mozilla.org/ja/docs/Web/Security/Same-origin_policy)的安全限制，应用程序在许多情况下将无法正常运行。

在接下来的用例章节中，我们将编写的应用程序为了避免[同源策略](https://developer.mozilla.org/ja/docs/Web/Security/Same-origin_policy)的限制，假定使用`http`模式的 URL 进行访问。 通过使用开发用的本地服务器，可以使用`http`模式的 URL 来显示在本地创建的 HTML 文件。

在这里，我们将看到如何设置用于开发的本地服务器。

### *准备 HTML 文件*

*首先，让我们创建一个只放置最基本元素的 HTML 文件。 在这里，我们创建了名为`index.html`的文件，并在 HTML 文件中编写如下内容。 在这个 HTML 文件中，使用`script`元素加载名为`index.js`的 JavaScript 文件。

index.html

```
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <title>index.html</title>
  </head>
  <body>
    <h1>ローカルサーバで配信中</h1>
    <script src="index.js"></script>
  </body>
</html> 
```

类似地，创建名为`index.js`的 JavaScript 文件。 在这个`index.js`文件中，只需编写输出日志以确认脚本已正确加载的处理。

index.js

```
console.log("index.js: loaded"); 
```

### *启动本地服务器*

*在与刚刚创建的`index.html`相同的目录中，启动本地服务器。 使用以下命令，可以下载并执行为本书创建的本地服务器模块[@js-primer/local-server](https://github.com/js-primer/local-server)。 该本地服务器模块具有提供在执行目录中的文件的功能，以便通过`http`模式的 URL 访问本地文件。

```
# からはじまる行はコメントなので実行はしなくてよい
# cdコマンドでファイルがあるディレクトリまで移動
$ cd "index.htmlがあるディレクトリのパス"

# npx コマンドでローカルサーバーを起動
$ npx --yes @js-primer/local-server

js-primerのローカルサーバーを起動しました。
次のURLをブラウザで開いてください。

  URL: http://localhost:3000 
```

当访问启动的本地服务器的 URL（`http://localhost:3000`）时，将显示先前的`index.html`内容。 在许多服务器中，如果不指定文件路径，例如`http://localhost:3000`，则会提供`index.html`。 `@js-primer/local-server`也具有此功能，因此`http://localhost:3000`和`http://localhost:3000/index.html`两个 URL 都提供相同的`index.html`。

![显示日志的 Web 控制台](img/060fab37fdefacc7581414049c4bbfae.png)

如果可以访问`index.html`，请检查`index.js`是否正确加载。 要查看通过 Console API 输出的日志，请打开浏览器的开发者工具。 大多数浏览器都包含开发者工具，但本书中将使用 Firefox 进行演示。

可以通过以下任一方法打开 Firefox 的开发者工具。

+   从 Firefox 菜单（如果有菜单栏或在 macOS 上，从工具菜单）中选择“Web 开发工具”子菜单中的“浏览器工具”

+   按下键盘快捷键 Ctrl+Shift+K（在 macOS 上为 Command+Option+K）

有关详细信息，请参阅“[浏览器的开发者工具是什么？](https://developer.mozilla.org/ja/docs/Learn/Common_questions/What_are_browser_developer_tools)”。

### *关闭本地服务器*

*最后，关闭启动的本地服务器。 可以通过在启动本地服务器的命令行中按下`Ctrl+C`来关闭。

可以同时启动多个本地服务器，但不能在多个服务器上使用相同的端口号。 端口是指刚刚启动的本地服务器 URL 中的`:3000`部分，表示在 3000 端口上启动了本地服务器。

如果默认端口（3000 端口）已被占用，则`@js-primer/local-server`将查找未使用的端口并启动本地服务器。此外，可以使用`--port`选项在任意端口号上启动本地服务器。

```
$ npx --yes @js-primer/local-server --port 8000 
```

本书将在默认端口号 3000 上使用`@js-primer/local-server`进行操作。 因此，通过按下`Ctrl+C`关闭不再使用的本地服务器，可以确保可以继续使用与书中相同的 URL（端口号）。

## *结论*

*在本章中，我们准备好了接下来用例章节所需的环境。

+   安装了 Node.js 的 LTS 版本

+   使用 npm 和 npx 安装和运行模块

+   使用`@js-primer/local-server`模块启动并关闭了本地服务器

npm 中已经有各种各样的本地服务器模块可供使用。本书将使用名为`@js-primer/local-server`的专用本地服务器模块，以确保所使用的本地服务器功能没有差异。
