# 再次编写单元测试

> 原文：[`jsprimer.net/use-case/nodecli/refactor-and-unittest/`](https://jsprimer.net/use-case/nodecli/refactor-and-unittest/)

在本节中，我们将向之前创建的 CLI 应用程序引入单元测试。在引入单元测试的同时，整理源代码，以便更容易进行测试。

到目前为止，所有处理都在一个 JavaScript 文件中描述。为了进行单元测试，测试目标必须作为模块分割。在本应用程序中，将命令行参数处理部分和 Markdown 到 HTML 转换部分分割。

## 将应用程序分割成模块

在实际进行应用程序的模块化之前，简单回顾一下 ECMAScript 模块中的导出。

在 ECMAScript 模块中，使用`export`语句导出变量、函数等对象，以便其他脚本可以使用。以下名为`greet.js`的文件是一个导出`greet`函数的模块示例。

greet.js

```
// greet.js
export function greet(name) {
    return `Hello ${name}!`;
}; 
```

对于使用此模块的一方来说，可以使用`import`语句导入指定文件路径的 JavaScript 文件。以下代码中，指定了之前提到的`greet.js`的路径，将其作为模块导入，并使用导出的`greet`函数。

greet-main.js

```
import { greet } from "./greet.js";
greet("World"); // => "Hello World!" 
```

现在进行应用程序的模块化，即这样将应用程序的一部分切分到其他文件中，并导出必要的对象，使其可以从外部使用。将功能作为模块切分，可以使应用程序和单元测试双方都可以使用。

那么让我们将 CLI 应用程序的源代码分割成模块。创建一个名为`md2html.js`的 JavaScript 文件，并按照以下方式使用 marked 进行 Markdown 的转换处理。

md2html.js

```
import { marked } from "marked";

export function md2html(markdown, cliOptions) {
    return marked.parse(markdown, {
        gfm: cliOptions.gfm,
    });
}; 
```

此模块导出的是一个函数，根据提供的选项将 Markdown 文本转换为 HTML。在应用程序的入口点`main.js`中，如下所示导入并使用此模块。

main.js

```
import { program } from "commander";
import * as fs from "node:fs/promises";
// md2htmlモジュールからmd2html 関数をインポートする
import { md2html } from "./md2html.js";

program.option("--gfm", "GFMを有効にする");
program.parse(process.argv);
const filePath = program.args[0];

const cliOptions = {
    gfm: false,
    ...program.opts(),
};

fs.readFile(filePath, { encoding: "utf8" }).then(file => {
    // md2htmlモジュールを使ってHTMLに変換する
    const html = md2html(file, cliOptions);
    console.log(html);
}).catch(err => {
    console.error(err.message);
    process.exit(1);
}); 
```

marked 包及其选项的描述被隐藏在一个`md2html`函数中，`main.js`变得简单。然后，`md2html.js`被作为一个独立的模块从应用程序中切分出来，使得单元测试成为可能。

## 创建单元测试执行环境

*单元测试的执行有多种方法。在本节中，我们将使用[Mocha](https://mochajs.org/)作为测试框架来创建单元测试的执行环境。Mocha 提供的测试执行环境中，全局定义了`it`、`describe`等函数。`it`函数在其内部发生错误时，将测试视为失败。也就是说，如果结果与预期不符，则抛出错误，如果符合预期则不抛出错误，这是编写测试代码的方式。

这次利用 Node.js 的标准模块之一[assert 模块](https://nodejs.org/api/assert.html)提供的`assert.strictEqual`方法。`assert.strictEqual`方法是一个函数，当第一和第二个参数的评估结果不相等时，会抛出异常。

使用 Mocha 创建测试环境，首先使用以下命令安装`mocha`包。

```
$ npm install --save-dev mocha@10 
```

`--save-dev`选项是用于将包作为`devDependencies`安装的。在`package.json`的`devDependencies`中，描述了在开发过程中所需的依赖库。

执行单元测试需要使用 Mocha 提供的`mocha`命令。安装 Mocha 后，在`package.json`的`scripts`属性中按以下方式描述。

```
{
    ...
    "scripts": {
        "test": "mocha test/"
    },
    ...
} 
```

通过这种方式，执行`npm test`命令时，将使用 Mocha 命令执行`test/`目录中的测试文件。尝试执行`npm test`命令，确认 Mocha 测试正在执行。由于还没有创建测试文件，所以会显示`Error: No test files found`错误。

```
$ npm test
> mocha

 Error: No test files found 
```

## 编写单元测试

*创建单元测试执行环境*

`it`函数将测试标题放入第一参数，将测试内容放入第二参数。

test/md2html-test.js

```
import * as assert from "node:assert";
import * as fs from "node:fs/promises";
import { md2html } from "../md2html.js";

it("converts Markdown to HTML (GFM=false)", async() => {
    // fs.readFileはPromiseを返すので、`await`式で読み込みが完了するまで待って内容を取得する
    const sample = await fs.readFile("test/fixtures/sample.md", { encoding: "utf8" });
    const expected = await fs.readFile("test/fixtures/expected.html", { encoding: "utf8" });
    // 末尾の改行の有無の違いを無視するため、変換後のHTMLのスペースをtrimメソッドで削除してから比較しています
    assert.strictEqual(md2html(sample, { gfm: false }).trimEnd(), expected.trimEnd());
});

it("converts Markdown to HTML (GFM=true)", async() => {
    const sample = await fs.readFile("test/fixtures/sample.md", { encoding: "utf8" });
    const expected = await fs.readFile("test/fixtures/expected-gfm.html", { encoding: "utf8" });
    // 末尾の改行の有無の違いを無視するため、変換後のHTMLのスペースをtrimメソッドで削除してから比較しています
    assert.strictEqual(md2html(sample, { gfm: true }).trimEnd(), expected.trimEnd());
}); 
```

使用`it`函数定义的单元测试正在测试`md2html`函数的转换结果是否符合预期。`test/fixtures`目录中配置了用于单元测试的文件。这次，转换源 Markdown 文件和预期的转换结果 HTML 文件都存在。

按照以下方式将转换源 Markdown 文件配置到`test/fixtures/sample.md`。

test/fixtures/sample.md

```
# サンプルファイル

これはサンプルです。
https://jsprimer.net/

- サンプル1
- サンプル2 
```

然后，将预期的转换结果 HTML 文件也配置到`test/fixtures`目录中。根据`gfm`选项的有无，创建`expected.html`和`expected-gfm.html`两个文件，如下所示。

test/fixtures/expected.html

```
<h1 id="サンプルファイル">サンプルファイル</h1>
<p>これはサンプルです。
https://jsprimer.net/</p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul> 
```

test/fixtures/expected-gfm.html

```
<h1 id="サンプルファイル">サンプルファイル</h1>
<p>これはサンプルです。
<a href="https://jsprimer.net/">https://jsprimer.net/</a></p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul> 
```

单元测试准备就绪后，再次执行`npm test`命令。如果 2 个测试都通过，则表示成功。

```
$ npm test
> mocha

  ✓ converts Markdown to HTML (GFM=false)
  ✓ converts Markdown to HTML (GFM=true)

  2 passing (31ms) 
```

如果单元测试未通过，可以检查以下事项。

+   是否在`test/fixtures`目录中创建了`sample.md`、`expected.html`和`expected-gfm.html`文件

+   每个文件是否使用 UTF-8 编码，并且换行符为 LF

+   每个文件中是否有额外的文字

例如，执行`npm test`后，如果测试失败，可以查看以下错误消息。

```
$ npm test
> mocha test/

  ✔ converts Markdown to HTML (GFM=false)
  1) converts Markdown to HTML (GFM=true)

  1 passing (17ms)
  1 failing

  1) converts Markdown to HTML (GFM=true):

      AssertionError [ERR_ASSERTION]: Expected values to be strictly equal:
+ actual - expected ... Lines skipped

  '<h1 id="サンプルファイル">サンプルファイル</h1>\n' +
    '<p>これはサンプルです。\n' +
...
    '<li>サンプル1</li>\n' +
    '<li>サンプル2</li>\n' +
+   '</ul>'
-   '</ul>\n' +
-   ';;;'
      + expected - actual

       <a href="https://jsprimer.net/">https://jsprimer.net/</a></p>
       <ul>
       <li>サンプル1</li>
       <li>サンプル2</li>
      -</ul>
      +</ul>
      +;;; 
```

在这个测试结果中，可以看到标题为`converts Markdown to HTML (GFM=true)`的测试失败了一个。此外，在`+ actual - expected`中，显示的是使用`assert.strictEqual`比较结果不一致的部分。在这种情况下，测试失败的原因是 expected（期望的结果）的末尾多了一个不必要的字符串`;;;`。因此，通过检查`expected-gfm.html`文件并删除不必要的`;;;`字符串，测试应该可以通过。

## *为什么进行单元测试*

*执行单元测试有许多优点。能够早期发现错误，以及能够安心地进行重构当然很重要，但保持单元测试可行本身就是有意义的。努力使代码易于测试，这是将应用程序正确模块化的指导方针。*

此外，单元测试还具有活文档的一面。文档如果不经常维护，很快就会与实际代码产生差异，但单元测试作为表示模块应满足的规格的文档发挥作用。

虽然单元测试的编写可能看起来很费时，但在中长期的维护中，它是不可或缺的。而且，为了编写好的测试，重要的是要养成日常编写测试的习惯。

## *总结*

*我们已经完成了 Node.js CLI 应用程序的创建和单元测试的引入，这个用例的目标。npm 包管理、外部模块的利用、使用`fs`模块进行文件操作等许多元素都出现了。这些在 Node.js 应用程序开发中的大多数用例中都会用到，所以请务必理解。*

## *本节检查清单*

**   将 Markdown 的转换处理切出为 ECMAScript 模块`md2html.js`，并从`main.js`中读取**

+   确认已安装 mocha 包，并可以通过`npm test`命令执行 mocha 命令

+   创建了`md2html`函数的单元测试，并确认了测试执行结果******
