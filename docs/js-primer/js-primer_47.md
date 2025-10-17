# 将 Markdown 转换为 HTML

> 原文：[`jsprimer.net/use-case/nodecli/md-to-html/`](https://jsprimer.net/use-case/nodecli/md-to-html/)

在前面的章节中，我们通过命令行参数读取文件，并将其显示在标准输出中。接下来，让我们尝试将读取的 Markdown 文件转换为 HTML，并将结果输出到标准输出。

## [](#use-marked-package)*使用 marked 包*

*为了将 Markdown 转换为 HTML，这次我们使用了[marked](https://github.com/markedjs/marked)这个库。marked 的包通过 npm 进行分发，所以像 commander 一样，使用`npm install`命令来安装包。

```
$ npm install [[email protected]](/cdn-cgi/l/email-protection) 
```

安装完成后，从 Node.js 脚本中读取。在前面的章节的最后，添加`marked`模块的读取处理到脚本中。以下是如何修改`main.js`，使用 marked 将 Markdown 文件转换为 HTML。

main.js

```
import { program } from "commander";
import * as fs from "node:fs/promises";
// markedモジュールからmarkedオブジェクトをインポートする
import { marked } from "marked";

program.parse(process.argv);
const filePath = program.args[0];

fs.readFile(filePath, { encoding: "utf8" }).then(file => {
    // MarkdownファイルをHTML 文字列に変換する
    const html = marked.parse(file);
    console.log(html);
}).catch(err => {
    console.error(err.message);
    process.exit(1);
}); 
```

## [](#create-convert-option)*创建转换选项*

marked 有 Markdown 的[转换选项](https://marked.js.org/#/USING_ADVANCED.md#options)，根据选项的设置，转换后的 HTML 会有所不同。因此，在应用程序中，我们可以设置选项的默认值，并允许通过命令行参数进行切换。

在本次应用程序中，我们以`gfm`为例处理 marked 的选项。

### [](#gfm-option)*gfm 选项*

*`gfm`选项是决定是否按照 GitHub 的 Markdown 规范（[GitHub Flavored Markdown](https://github.github.com/gfm/), GFM）进行转换的选项。marked 中，这个`gfm`选项默认设置为`true`。GFM 在标准 Markdown 的基础上添加了一些扩展，其中最著名的扩展是 URL 的自动链接化。以下是如何修改`sample.md`，并使用之前脚本和将`gfm`选项设置为`false`的脚本查看结果差异。

sample.md

```
# サンプルファイル

これはサンプルです。
https://jsprimer.net/

- サンプル1
- サンプル2 
```

当`gfm`选项有效时，URL 字符串将自动转换为`<a>`标签的链接。

```
<h1 id="サンプルファイル">サンプルファイル</h1>
<p>これはサンプルです。
<a href="https://jsprimer.net/">https://jsprimer.net/</a></p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul> 
```

另一方面，将`gfm`选项设置为`false`时，它将被简单地作为文本处理，不会转换为链接。

main.js

```
import { program } from "commander";
import * as fs from "node:fs/promises";
import { marked } from "marked";

program.parse(process.argv);
const filePath = program.args[0];

fs.readFile(filePath, { encoding: "utf8" }).then(file => {
    // gfmオプションを無効にする
    const html = marked.parse(file, {
        gfm: false
    });
    console.log(html);
}).catch(err => {
    console.error(err.message);
    process.exit(1);
}); 
```

```
<h1 id="サンプルファイル">サンプルファイル</h1>
<p>これはサンプルです。
https://jsprimer.net/</p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul> 
```

除了自动链接之外，还有一些其他扩展，但更详细的内容请参考[GitHub Flavored Markdown](https://github.github.com/gfm/)的文档。

### [](#receive-option)*从命令行参数接收选项*

*接下来，让我们将`gfm`选项通过命令行参数进行控制。在应用程序的默认设置中，将`gfm`选项设置为无效，然后通过添加`--gfm`选项来执行命令。

```
$ node main.js --gfm sample.md 
```

当你想处理像`--gfm`这样的命令行参数时，可以使用 commander 的`option`方法。首先定义所需的选项，然后解析命令行参数，就可以通过`program.opts`方法获取解析结果的对象。

```
// gfmオプションを定義する
program.option("--gfm", "GFMを有効にする");
// コマンドライン引数をパースする
program.parse(process.argv);
// オプションのパース結果をオブジェクトとして取得する
const options = program.opts();
console.log(options.gfm); 
```

`--gfm`选项可以在指定`sample.md`文件路径的前后使用。这是因为`program.args`数组中不包含通过`program.option`方法定义的选项。直接使用`process.argv`数组会使得处理此类选项变得复杂，因此使用像 commander 这样的解析处理是常见的做法。

### [](#declare-default)*定义默认设置*

在应用程序中保留默认设置可以减少在 marked 行为发生变化时的影响。以下是如何创建表示选项的`cliOptions`对象，并使用`program.opts`方法获取的值进行设置。对于未指定的选项，使用`??`（空合并运算符）来设置默认值。空合并运算符仅在左边的值为 nullish 时才返回右边的值，因此它在需要区分值未指定和显式设置为`false`的情况时很有用。

```
// コマンドライン引数のオプションを取得する
const options = program.opts();

// コマンドライン引数で指定されなかったオプションにデフォルト値を上書きする
const cliOptions = {
    gfm: options.gfm ?? false,
}; 
```

将这样创建的 cliOptions 对象作为选项传递给 marked 的`parse`函数。`main.js`的完整内容如下。

main.js

```
import { program } from "commander";
import * as fs from "node:fs/promises";
import { marked } from "marked";

// gfmオプションを定義する
program.option("--gfm", "GFMを有効にする");
program.parse(process.argv);
const filePath = program.args[0];

// コマンドライン引数のオプションを取得する
const options = program.opts();

// コマンドライン引数で指定されなかったオプションにデフォルト値を上書きする
const cliOptions = {
    gfm: options.gfm ?? false,
};

fs.readFile(filePath, { encoding: "utf8" }).then(file => {
    const html = marked.parse(file, {
        // オプションの値を使用する
        gfm: cliOptions.gfm,
    });
    console.log(html);
}).catch(err => {
    console.error(err.message);
    process.exit(1);
}); 
```

使用定义的命令行参数尝试将 Markdown 文件进行转换。

```
$ node main.js sample.md
<h1 id="サンプルファイル">サンプルファイル</h1>
<p>これはサンプルです。
https://jsprimer.net/</p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul> 
```

再次，添加`gfm`选项执行时，输出应该是这样的。

```
$ node main.js --gfm sample.md
<h1 id="サンプルファイル">サンプルファイル</h1>
<p>これはサンプルです。
<a href="https://jsprimer.net/">https://jsprimer.net/</a></p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul> 
```

这样，我们就可以通过命令行参数将 Markdown 转换的设置作为选项提供了。在下一节中，我们将整理应用程序的代码，并在最后引入单元测试。

## [](#section-checklist)*本节检查清单*

**使用 marked 包将 Markdown 文本转换为 HTML 文本**

+   在命令行参数中设置了 marked 的转换选项

+   定义了默认选项，并允许通过命令行参数进行覆盖******
