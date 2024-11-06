# 处理命令行参数

> 原文：[`jsprimer.net/use-case/nodecli/argument-parse/`](https://jsprimer.net/use-case/nodecli/argument-parse/)

此使用案例中创建的 CLI 应用程序的目标是将给定的 Markdown 文件转换为 HTML。本节将介绍如何在使用`node`命令执行脚本时传递参数并解析命令行参数。

## [](#process-object-and-commandline-args)*`process`对象和命令行参数*

*在处理命令行参数之前，首先让我们了解一下`process`对象。`process`对象是 Node.js 执行环境的全局变量之一。`process`对象提供了有关当前 Node.js 执行进程的信息获取和操作 API。有关详细信息，请参阅[官方文档](https://nodejs.org/docs/latest-v20.x/api/process.html#process_process)。

访问命令行参数的方法是通过`process`对象的`argv`属性，它是一个字符串数组。让我们修改`main.js`，并输出`process.argv`到控制台。

main.js

```
// コンソールにコマンドライン引数を出力する
console.log(process.argv); 
```

使用以下命令以带有命令行参数的形式执行此脚本。

```
$ node main.js one two=three four 
```

此命令的执行结果如下：

```
[
  '/usr/local/bin/node', // Node.jsの実行プロセスのパス
  '/Users/laco/nodecli/main.js', // 実行したスクリプトファイルのパス
  'one', // 1 番目の引数
  'two=three', // 2 番目
  'four'  // 3 番目
] 
```

第 1 个和第 2 个元素始终是`node`命令和执行的脚本文件路径。换句话说，应用程序使用的是第 3 个元素及以后的元素作为命令行参数。

## [](#parse-args)*解析命令行参数*

*通过使用`process.argv`数组，您可以获取命令行参数，但获取的信息可能包含应用程序不需要的内容。此外，由于以字符串数组的形式传递，因此在接收布尔值等真值时也不方便。因此，在处理命令行参数时，通常会对其进行解析并格式化为更易于处理的值。

在本例中，我们将使用名为[commander](https://github.com/tj/commander.js/)的库来解析命令行参数。虽然您可以手动进行字符串处理，但对于此类常见操作，使用现有库会更加简单。

### [](#install-commander)*安装`commander`包*

*可以使用[npm](https://www.npmjs.com/)的`npm install`命令来安装`commander`。如果您尚未准备好 npm 运行环境，请先参考“设置本地环境”章节。

在使用 npm 安装包之前，首先要创建一个名为`package.json`的文件。`package.json`记录了应用程序依赖的包的类型和版本等信息，是一个 JSON 格式的文件。可以使用`npm init`命令生成`package.json`文件的模板。通常会通过交互式提示来设置信息，但在这里我们将使用`--yes`选项以默认值创建`package.json`。

在`nodecli`目录中，执行`npm init --yes`命令以创建`package.json`。

```
$ npm init --yes 
```

生成的`package.json`文件如下所示：

package.json

```
{
  "name": "nodecli",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
} 
```

当`package.json`文件准备好后，使用`npm install`命令安装`commander`包。此命令的参数是要安装的包的名称和版本，用`@`符号连接。如果不指定版本，则自动选择当前的最新稳定版本。执行以下命令以安装 commander 的 9.0 版本。^(1)

```
$ npm install [[email protected]](/cdn-cgi/l/email-protection) 
```

安装完成后，`package.json`文件如下所示：

package.json

```
{
  "name": "nodecli",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "commander": "⁹.0.0"
  }
} 
```

此外，执行`npm install`时会同时生成`package-lock.json`文件。该文件记录了 npm 安装的包的实际版本。虽然我们将 commander 的版本设置为`9.0`，但实际安装的将是与`9.0.x`匹配的最新版本。`package-lock.json`文件记录了实际安装的版本，这样再次执行`npm install`时就可以防止安装不同的版本。

### [](#esmodule)*使用 ECMAScript 模块*

*在这个使用案例中，我们将使用安装的`commander`包，并结合我们学到的基本语法 ECMAScript 模块。`commander`包支持 ECMAScript 模块，因此可以使用`import`语句来导入变量和函数等。

```
import { program } from "commander"; 
```

但是，要导入 ECMAScript 模块的包，导入源文件本身也必须是 ECMAScript 模块。因为[Node.js](https://nodejs.org/)支持另一种模块形式，即[CommonJS 模块](https://nodejs.org/docs/latest/api/modules.html)，而 CommonJS 模块形式不支持`import`语句。因此，需要告诉 Node.js 即将执行的 JavaScript 文件使用哪种形式。

Node.js 通过最接近的上级目录的`package.json`的`type`字段值来确定 JavaScript 文件的模块格式。 如果`type`字段为`module`，则将其视为 ECMAScript 模块；如果`type`字段为`commonjs`，则将其视为 CommonJS 模块。^(2) 此外，还可以通过 JavaScript 文件的扩展名明确指定。如果扩展名为`.mjs`，则被识别为 ECMAScript 模块；如果为`.cjs`，则被识别为 CommonJS 模块。

为了将`main.js`识别为 ECMAScript 模块，可以将`package.json`中添加`type`字段如下。

```
# npm pkg コマンドで type フィールドの値をセットする
$ npm pkg set type=module 
```

package.json

```
{
  "name": "nodecli",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "commander": "⁹.0.0"
  }
} 
```

#### [](#commonjs-module)*[专栏] CommonJS 模块*

*[CommonJS 模块](https://nodejs.org/docs/latest/api/modules.html)是 Node.js 环境中使用的 JavaScript 模块化机制。 CommonJS 模块在 ECMAScript 模块规范制定之前就已经在 Node.js 中使用。

尽管 Node.js 现在也支持 ECMAScript 模块，但诸如`fs`之类的标准模块仍以 CommonJS 模块的形式提供。 此外，即使在第三方库或长期开发的项目源代码中，也经常使用 CommonJS 模块。 因此，在这两种模块格式共存的情况下，开发人员需要注意模块格式之间的互操作性（组合时的行为）。^(3)

Node.js 支持从 ECMAScript 模块导入 CommonJS 模块的双向互操作性。 例如，使用 CommonJS 模块中的`exports`对象导出的对象可以通过 ECMAScript 模块中的`import`语句导入。 Node.js 的标准模块可以从 ECMAScript 模块的 JavaScript 文件中使用，但这是由于这种互操作性造成的。

```
// lib.cjs
exports.key = "value";

// app.mjs
import { key } from "./lib.cjs"; 
```

另一方面，不支持从 CommonJS 模块导入 ECMAScript 模块的双向互操作性。 如果来自现有库的模块是 ECMAScript 模块，则使用该库的应用程序也必须是 ECMAScript 模块。 在开发 Node.js 应用程序时使用多个包时，需要注意互操作性。

### [](#get-file-path)*从命令行参数获取文件路径*

*使用刚刚安装的`commander`包，获取通过命令行参数传递的文件路径。 在这个 CLI 应用程序中，将以以下命令格式接收要处理的文件路径。

```
$ node main.js ./sample.md 
```

要使用 commander 解析命令行参数，需要将命令行参数传递给导入的`program`对象的`parse`方法。

```
// commanderモジュールからprogramオブジェクトをインポートする
import { program } from "commander";
// コマンドライン引数をcommanderでパースする
program.parse(process.argv); 
```

调用`parse`方法后，可以从`program`对象中提取解析后的命令行参数结果。 在本例中，文件路径存储在`program.args`数组中。 `program.args`数组按顺序存储了除了选项和标志之外的剩余命令行参数。

然后，将`main.js`修改如下，以获取通过命令行参数传递的文件路径。

main.js

```
// commanderモジュールからprogramオブジェクトをインポートする
import { program } from "commander";

// コマンドライン引数をcommanderでパースする
program.parse(process.argv);

// ファイルパスをprogram.args 配列から取り出す
const filePath = program.args[0];
console.log(filePath); 
```

执行以下命令后，将获取存储在`program.args`数组中的`./sample.md`字符串并输出到控制台。 `./sample.md`在`process.argv`数组中是第 3 个元素，但在解析后的`program.args`数组中变为第 1 个元素，更易于处理。

```
$ node main.js ./sample.md
./sample.md 
```

通过使用类似 commander 的库来声明性地定义和处理命令行参数，而不是直接处理`process.argv`数组，可以更好地处理命令行参数。 在下一节中，将添加处理基于命令行参数获取的文件路径的过程。

#### [](#syntax-error-import-statement)*[错误示例] SyntaxError: Cannot use import statement outside a module*

*如果出现“无法在模块外部使用 import 语��”的错误，则表示 Node.js 无法识别`main.js`文件为 ECMAScript 模块。

```
import { program } from "commander";
^^^^^^

SyntaxError: Cannot use import statement outside a module 
```

如使用 ECMAScript 模块所述，将`package.json`的`type`字段设置为`module`。

## [](#section-checklist)*本节的检查清单*

**   确认`process.argv`数组中包含`node`命令的命令行参数

+   理解使用 npm 安装包的方法

+   确认使用 ECMAScript 模块加载包

+   确认可以使用 commander 解析命令行参数

+   可以获取通过命令行参数传递的文件路径并输出到控制台

> ¹. 使用--save 选项安装的意义相同。从 npm 5.0.0 开始，--save 成为默认选项。 ↩
> 
> ². [package.json 和文件扩展名](https://nodejs.org/api/packages.html#packagejson-and-file-extensions) ↩
> 
> ³. [与 CommonJS 的互操作性](https://nodejs.org/api/esm.html#interoperability-with-commonjs) ↩********
