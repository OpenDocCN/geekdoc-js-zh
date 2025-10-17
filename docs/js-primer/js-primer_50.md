# 入口点

> 原文：[`jsprimer.net/use-case/todoapp/entrypoint/`](https://jsprimer.net/use-case/todoapp/entrypoint/)

エントリーポイント指的是应用程序中首先调用的部分。

在“Ajax 通信:エントリーポイント”用例中，入口点只是 HTML（`index.html`）。 首先加载 HTML，然后加载 HTML 中的`script`元素指定的 JavaScript 文件。

本 Todo 应用程序将 JavaScript 处理模块化，并将每个模块作为单独的 JavaScript 文件创建。 JavaScript 模块可以通过 HTML 中的`<script type="module">`加载，但每个`script`元素都有自己的模块作用域。 模块作用域是指在模块的顶层自动创建的作用域，位于全局作用域下。 通过在不同的`script`元素中加载 JavaScript 模块，模块之间的作用域不同，因此无法协作。

下面的代码示例展示了由于每个`<script type="module">`的作用域不同，无法访问在另一个`script`元素中定义的变量。 这也适用于将 JavaScript 模块作为文件并通过`src`属性加载的情况。

```
 <script type="module"> export const scopeA = "A"; </script>
  <script type="module"> // 異なるmoduleスコープの変数には直接アクセスできない
    console.log(scopeA); // => ReferenceError: scopeA is not defined </script> 
```

当将模块分别放在不同的`script`元素中处理时，模块之间无法协作。 因此，在 HTML 中只加载`index.js`的`script`元素，并从`index.js`中使用`import`语句加载其他模块。 通过使用`import`语句，模块之间将位于一个`<script type="module">`的作用域内，从而实现模块之间的协作。 将从 HTML 加载的 JavaScript 文件（`index.js`）作为 JavaScript 的入口点。

因此，在本次创建的 Todo 应用程序中，我们将准备 HTML 和 JavaScript 这两个入口点。

+   `index.html`：首先加载的文件，加载`index.js`

+   `index.js`：从`index.html`加载的文件，JavaScript 中首先加载的文件

在本节中，我们将创建这两个入口点并加载它们。

## [](#project-directory)*创建项目目录*

*在这个应用中，需要多个文件，包括 HTML 和 JavaScript 等。因此，首先要创建一个目录来存放这些文件。

在这里，我们将创建一个名为`todoapp`的新目录。 然后我们将在创建的`todoapp`目录下继续操作。

在这个项目中，确保将文件保存为**UTF-8**编码，并使用**LF**作为换行符。

## [](#preparing-html)*准备 HTML 文件*

*首先，创建一个包含最基本元素的 HTML 文件作为入口点。 在`todoapp`目录中创建名为`index.html`的 HTML 文件，并编写以下内容。 在`body`元素的底部使用`script`元素加载`index.js`，这是本应用程序的处理 JavaScript 文件。

index.html

```
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <title>Todo App</title>
  </head>
  <body>
    <h1>Todo App</h1>
    <script type="module" src="index.js"></script>
  </body>
</html> 
```

接下来，在`todoapp`目录中创建`index.js`，内容如下。 为了确保`index.js`被正确加载，我们只编写输出日志到控制台的处理。

index.js

```
console.log("index.js: loaded"); 
```

到目前为止，`todoapp`目录的文件布局如下：

```
todoapp
├── index.html
└── index.js 
```

接下来，我们将在浏览器中打开`index.html`，并确认控制台中是否输出了日志。

## [](#local-server)*在本地服务器上查看 HTML*

*在打开`index.html`之前，准备一个用于开发的本地服务器。 虽然可以直接打开 HTML 文件而不启动本地服务器，但这样的 URL 将以`file:///`开头。 使用`file`协议时，由于[同源策略](https://developer.mozilla.org/ja/docs/Web/Security/Same-origin_policy)，JavaScript 模块将无法正常工作。 因此，在本章中，我们假设已经启动了本地服务器，并通过以`http`开头的 URL 进行访问。

在命令行中进入`todoapp`目录，并使用以下命令启动本地服务器。 使用`npx`命令下载并执行为本书创建的`@js-primer/local-server`本地服务器模块。 如果尚未准备好`npx`命令，请先参考“应用程序开发准备”章节。

```
# todoapp/ディレクトリに移動する
$ cd todoapp/
# todoapp/をルートにしたローカルサーバーを起動する
$ npx --yes @js-primer/local-server

todoappのローカルサーバーを起動しました。
次のURLをブラウザで開いてください。

  URL: http://localhost:3000 
```

访问启动的本地服务器的 URL（`http://localhost:3000`）。 页面将显示`index.html`的内容，并且可以在开发者工具的控制台中看到`index.js: loaded`的日志输出。

![Web 控制台中显示的日志](img/c033f8a07de6770d0a4a364a4a65cebe.png)

### [](#view-console-log-in-dev-tools)*在开发者工具中查看控制台日志*

*要查看通过 Console API 输出的日志，需要打开 Web 浏览器的开发者工具。 大多数浏览器都内置了开发者工具，但本章将使用 Firefox 进行演示。 打开开发者工具的**控制台**选项卡，可以查看通过 Console API 输出的日志。

可以通过以下任一方法打开 Firefox 的开发者工具。

+   Firefox メニュー（メニューバーがある場合や macOS では、ツールメニュー）の "ブラウザーツール"のサブメニューから "ウェブ開発ツール" を選択する

+   キーボードショートカット`Ctrl+Shift+K`（macOSでは`Command+Option+K`）を押下する

詳細は「[浏览器开发者工具是什么？](https://developer.mozilla.org/ja/docs/Learn/Common_questions/What_are_browser_developer_tools)」を参照してください。

### [](#error-not-display-console-log)*控制台日志不显示*

*HTMLは表示されるがコンソールログに`index.js: loaded`が表示されない場合は、次のような問題に該当してないかを確認してください。

#### [](#fail-to-load-javascript-module)*[错误示例] `index.js`的读取失败*

*`script`要素の`src`属性に指定した`index.js`のパスにファイルが存在しているかを確認してください。 `<script type="module" src="index.js">`とした場合は`index.html`と`index.js`は同じディレクトリに配置する必要があります。

另外，如果控制台显示类似`CORS policy Invalid`的错误，则是因为[同源策略](https://developer.mozilla.org/ja/docs/Web/Security/Same-origin_policy)导致`index.js`的读取失败。正如之前所介绍的，从以`file:`开头的页面上无法正确运行 JavaScript 模块。因此，请启动本地服务器，并确认正在访问以`http:`开头的本地服务器（URL）。

#### [](#unsupport-javascript-module)*[错误示例] 正在使用的浏览器不支持 JavaScript 模块*

*由于 JavaScript 模块是较新的功能，因此需要版本 60 以上的 Firefox。版本 60 以下的 Firefox 无法读取作为 JavaScript 模块的`index.js`，因此不会在控制台输出日志。

在本次 Todo 应用程序中，需要使用支持原生 JavaScript 模块的浏览器。[Can I Use](https://caniuse.com/#feat=es6-module)汇总了支持原生 JavaScript 模块的浏览器。即使在不支持的浏览器中，也可以使用名为 Bundler 的工具来支持，但本章中省略了这部分内容。

* * *

## [](#module-entry-point)*创建模块的入口点*

*最后，从作为入口点的`index.js`中尝试导入其他 JavaScript 文件作为模块。在这个应用程序中，JavaScript 模块多次出现，因此创建了一个名为`src/`的目录，并在`src/`目录下编写 JavaScript 模块。这次创建了一个名为`src/App.js`的文件，并将其作为模块从`index.js`中导入。

按照`src/App.js`的以下配置创建文件。

```
todoapp
├── index.html
├── index.js
└── src
    └── App.js 
```

创建`src/App.js`文件，并编写以下内容的 JavaScript 模块。`App.js`是一个按名称导出`App`类的模块。此外，在`App`类的构造函数中，为了验证，写入输出控制台日志的代码。

src/App.js

```
console.log("App.js: loaded");
export class App {
    constructor() {
        console.log("App initialized");
    }
} 
```

接下来，为了从`index.js`中使用`import`，将`src/App.js`导入进来。将`index.js`修改如下，从`App.js`导入`App`类并实例化。

index.js

```
import { App } from "./src/App.js";
const app = new App(); 
```

再次使用浏览器访问本地服务器的 URL（`http://localhost:3000`），并重新加载。控制台日志将按以下顺序输出处理过程。

```
App.js: loaded
App initialized 
```

首先从`index.js`中按名称导入了名为`App`的类，该类位于`src/App.js`。接着，从日志中可以确认`App`类已被实例化。

这样，就确认了 HTML 和 JavaScript 各自的入口点创建和运行情况。

### [](#error-import-app-js)*`App.js`的读取失败*

*如果在此之前的 JavaScript 模块读取过程中出现错误且无法运行，请检查以下事项。

如果目录结构或`import`语句中指定的文件路径不同，则无法读取文件并出现错误。在这种情况下，请打开开发者工具，并检查控制台是否有错误输出。

下面总结了使用`import`语句进行 JavaScript 模块读取时可能遇到的典型错误及其处理方法。

#### [](#syntax-error-import-declarations)*[错误示例] 语法错误：导入声明只能出现在模块的最顶层*

*“`import`声明只能在模块的顶层使用”这个错误表明，`import`语句没有满足可使用的条件。换句话说，`import`语句不在顶层，或者是在非模块执行上下文中执行的。

如果在函数中声明了`import`，由于`import`声明不在顶层，因此会引发错误。在这种情况下，请尝试将`import`语句移动到顶层（程序的最底部）。

在非模块执行上下文中执行意味着执行上下文是 Script。JavaScript 有两种执行上下文：Script 和 Module。`import`语句只能在执行上下文为 Module 时使用。因此，请检查`script`元素的`type`属性是否指定了`module`。

若要将执行上下文作为模块执行，需要指定`type=module`，例如`<script type="module" src="index.js">`（由于从`index.js`中的`import`语句引入的`App.js`将继承执行上下文，因此将在模块的执行上下文中处理）。

#### [](#fail-to-load-src-app)*[错误示例] 无法加载模块源 "[`localhost:3000/src/App”](http://localhost:3000/src/App”)。*

*出现了无法加载`App.js`的错误。仔细查看错误消息，发现错误在于`App`而不是`App.js`。

在`import`语句中，不要省略要加载的文件的扩展名。因此，如果省略了扩展名（`.js`），则会出现此错误。

```
// エラーとなる例
import { App } from "./src/App"; 
```

应正确编写路径，包括文件扩展名。还要确保指定的路径（`./src/App.js`）中存在文件。

```
// 正しい例
import { App } from "./src/App.js"; 
```

## [](#conclusion)*结论*

*在此部分中，我们创建了作为 HTML 入口点的文件，并加载了 JavaScript 模块的入口点 JavaScript 文件。

## [](#section-checklist)*此部分的检查清单*

创建了名为**`todoapp`**的项目目录。

+   创建了入口点`index.html`。

+   创建了 JavaScript 的入口点`index.js`，并从`index.html`中加载了它。

+   使用本地服务器显示了`index.html`。

+   `src/App.js`文件已创建，并确保可以通过`import`语句从`index.js`中进行引用。

到目前为止的 Todo 应用程序可以在以下 URL 中找到。

+   [`jsprimer.net/use-case/todoapp/entrypoint/module-entry/`](https://jsprimer.net/use-case/todoapp/entrypoint/module-entry/)*************
