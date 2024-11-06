# 读取文件

> 原文：[`jsprimer.net/use-case/nodecli/read-file/`](https://jsprimer.net/use-case/nodecli/read-file/)

前のセクションではコマンドライン引数からファイルパスを取得して利用できるようになりました。 このセクションでは渡されたファイルパスを元にMarkdownファイルを読み込んで、標準出力に表示してみましょう。

## [](#read-file-by-fs)*使用`fs`模块读取文件*

*利用前一节获取的文件路径，读取文件。 要在 Node.js 中进行文件读写，可以使用标准模块[`fs`模块](https://nodejs.org/api/fs.html)。 首先创建要读取的文件。 将其命名为`sample.md`，并放置在与`main.js`相同的`nodecli`目录中。

sample.md

```
# sample 
```

### [](#fs-module)*`fs`模块*

*`fs`模块提供了 Node.js 中进行文件读写的基本函数。

`fs`模块提供了同步和异步两种形式。 虽然`fs`模块提供了同步和异步 API，但异步形式的 API 也可以使用`fs/promises`模块名来引用。 本书中为了易于理解，将仅使用提供异步形式 API 的`fs/promises`模块。

Node.js 的标准模块可以通过添加`node:`前缀（例如`node:fs`）来导入。 尽管可以省略前缀导入`fs`，但建议保留前缀以明确区分来自 npm 安装的第三方模块。

下面的代码使用 ECMAScript 模块的`import * as`语法，将`fs/promises`模块作为`fs`对象导入。

```
// fs/promisesモジュール全体を読み込む
import * as fs from "node:fs/promises"; 
```

当然，也可以使用命名导入，只使用部分 API 而不是导入整个`fs/promises`模块。

```
// fs/promisesモジュールからreadFile 関数を読み込む
import { readFile } from "node:fs/promises"; 
```

`fs/promises`的异步 API 根据模块名称就可以理解，返回 Promise。 当文件读写等异步操作成功时，返回的 Promise 实例将被解析。 另一方面，当文件读写等异步操作失败时，返回的 Promise 实例将被拒绝。

下面的示例代码是使用`fs/promises`的`readFile`方法读取指定文件的示例。

```
// 非同期 APIを提供するfs/promisesモジュールを読み込む
import * as fs from "node:fs/promises";

fs.readFile("sample.md").then(file => {
    console.log(file);
}).catch(err => {
    console.error(err);
}); 
```

然后，以下示例代码是使用`fs`模块的`readFileSync`方法读取指定文件的示例。 在 Node.js 中，具有异步 API 和同步 API 的 API 都包含在方法名称的末尾明确表示为`Sync`。

```
// 同期 APIを提供するfsモジュールを読み込む
import * as fs from "node:fs";

try {
    const file = fs.readFileSync("sample.md");
} catch (err) {
    // ファイルが読み込めないなどのエラーが発生したときに呼ばれる
} 
```

Node.js 是单线程的，因此通常选择不阻塞其他处理的异步形式 API。 除了`fs/promises`模块外，Node.js 还提供了许多其他异步 API，因此要熟悉异步处理。

### [](#use-readFile)*使用 readFile 函数*

*现在让我们使用`fs/promises`模块的`readFile`方法来读取`sample.md`文件。 修改`main.js`如下，并基于从命令行参数获取的文件路径来读取文件并将其输出到控制台。

main.js

```
import { program } from "commander";
// fs/promisesモジュールをfsオブジェクトとしてインポートする
import * as fs from "node:fs/promises";

// コマンドライン引数からファイルパスを取得する
program.parse(process.argv);
const filePath = program.args[0];

// ファイルを非同期で読み込む
fs.readFile(filePath).then(file => {
    console.log(file);
}); 
```

传递`sample.md`作为参数的执行结果如下。 第二个参数不是字符串，这是因为回调函数的第二个参数是代表文件内容的`Buffer`实例。 `Buffer`实例将文件内容保存为字节序列。 因此，直接传递给`console.log`方法并不会变成可读的字符串。

```
$ node main.js sample.md
<Buffer 23 20 73 61 6d 70 6c 65> 
```

`fs.readFile`函数可以通过参数指定文件的读取方式。 如果预先指定文件的编码方式作为第二个参数，则它将自动转换为字符串并传递给回调函数。 修改`main.js`如下，将读取的文件转换为 UTF-8。

main.js

```
import { program } from "commander";
import * as fs from "node:fs/promises";

program.parse(process.argv);
const filePath = program.args[0];

// ファイルをUTF-8として非同期で読み込む
fs.readFile(filePath, { encoding: "utf8" }).then(file => {
    console.log(file);
}); 
```

再次执行相同的命令，执行结果如下。 可以将`sample.md`文件的内容作为字符串输出。

```
$ node main.js sample.md
# sample 
```

### [](#error-handling)*错误处理*

*由于文件的存在与否、权限和文件系统的差异等原因，文件读写操作容易引发异常，因此务必编写错误处理代码。

接下来的代码很简单，只是在`readFile`返回的 Promise 对象上添加了`catch`方法。 当发生错误时，它会显示错误消息并使用`process.exit`函数指定退出状态来终止进程。 在这里，我们使用表示一般错误的退出状态`1`来终止进程。

main.js

```
import { program } from "commander";
import * as fs from "node:fs/promises";

program.parse(process.argv);
const filePath = program.args[0];

// ファイルを非同期で読み込む
fs.readFile(filePath, { encoding: "utf8" }).then(file => {
    console.log(file);
}).catch(err => {
    console.error(err.message);
    // 終了ステータス 1（一般的なエラー）としてプロセスを終了する
    process.exit(1);
}); 
```

当传递一个不存在的文件`notfound.md`作为命令行参数运行时，将会出现以下错误并退出。

```
$ node main.js notfound.md
ENOENT: no such file or directory, open 'notfound.md' 
```

现在，我们已经可以读取并输出了通过命令行参数指定的文件。 在下一节中，我们将添加将读取的 Markdown 文件转换为 HTML 的功能。

## [](#node-error-first-callbak)*[专栏] Node.js 的错误优先回调*

*我们介绍了`fs`模块提供同步和异步 API 的事实。 出于历史原因，在 Node.js 中也存在提供 Promise 和错误优先回调两种类型的异步 API 的情况。

`fs/promises`模块的`readFile`方法是一个返回 Promise 的异步 API。 另一方面，`fs`模块也有一个`readFile`方法，该 API 是一个处理错误优先回调的异步 API。

```
// fsモジュールにはエラーファーストコールバックを扱う非同期 APIも含まれる
import * as fs from "node:fs/promises";

// エラーファーストコールバックの第 1 引数にはエラー、第 2 引数 には結果が入るというルール
fs.readFile("sample.md", (err, file) => {
    if (err) {
        console.error(err.message);
        process.exit(1);
        return;
    }
    console.log(file);
}); 
```

错误优先回调也在异步章节中介绍。在 ES2015 之前，错误优先回调被广泛用作处理异步操作的方法，因为 Promises 在 ES2015 之前进入 ECMAScript 之前，许多 Node.js 模块是在此之前创建的，因此像`fs`模块一样处理错误优先回调的 API 也存在。

然而，由于 Promise 成为异步 API 的主流，Node.js 也添加了用于处理 Promise 的 API。然而，由于已经存在提供错误优先回调的同名方法，因此可以将其拆分为模块，例如对于`fs`，可以使用`fs/promises`。此外，Node.js 还提供了一个名为`util.promisify`的方法，用于将接受错误优先回调的异步 API 包装为返回 Promise 的异步 API。

由于历史原因，Node.js 可能同时提供错误优先回调和 Promise 的 API。然而，如果两者都提供，则应使用 Promise 的 API。处理 Promise 的 API 具有许多优点，如易于与其他处理 Promise 的操作协作、对 Async Function 的语法支持、简洁的错误处理等。

## [](#section-checklist)*本节的检查清单*

**   使用`fs/promises`模块的`readFile`函数读取文件

+   将 UTF-8 格式文件的内容输出到控制台

+   `readFile`函数的调用中添加了错误处理逻辑
