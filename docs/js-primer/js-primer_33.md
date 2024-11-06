# [ES2015] ECMAScriptモジュール

> 原文：[`jsprimer.net/basic/module/`](https://jsprimer.net/basic/module/)

モジュールとは、変数や関数などをまとめたものです。 JavaScriptにおいては、1つのモジュールは1つのJavaScriptファイルに対応します。

モジュールについてはNode.jsでCLIアプリのユースケースやTodoアプリのユースケースで実際に動かしながら学ぶため、ここでは構文の説明とモジュールのイメージをつかむのが目的です。 この章のサンプルコードを実際に動かすためにはローカルサーバーなどの準備が必要です。 そのため、ユースケースの章を先に読んでから戻ってきてもかまいません。

モジュールは、保守性・名前空間・再利用性のために使われます。

+   保守性: 依存性の高いコードの集合を一箇所にまとめ、それ以外のモジュールへの依存性を減らせます

+   名前空間: モジュールごとに分かれたスコープがあり、グローバルの名前空間を汚染しません

+   再利用性: 便利な変数や関数を複数の場所にコピーアンドペーストせず、モジュールとして再利用できます

モジュールは変数や関数などをモジュール外部にエクスポートできます。また、モジュールからエクスポートされた変数や関数などをインポートして利用できます。 モジュールに処理を分けることで、コードの見通しが良くなったり、特定のことに関する処理をモジュールにまとめたり、処理を再利用できるようになります。 それによって、コードの行数が増えてきた場合にも、一度にみるコードの量をモジュールで分割できるようになり、メンテナンス性がよくなります。

この章では、**ECMAScriptモジュール（ESモジュール、JavaScriptモジュールとも呼ばれる）** について見ていきます。 ECMAScriptモジュールは、ES2015��導入されたJavaScriptファイルをモジュール化する言語標準の機能です。

## [](#es-module-syntax)*ECMAScriptモジュールの構文*

*ECMAScriptモジュールは、[export 文](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export)によって変数や関数などをエクスポートできます。 また、[import 文](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import)を使って別のモジュールからエクスポートされたものをインポートできます。 インポートとエクスポートはそれぞれに **名前つき** と **デフォルト** という2 種類の方法があります。

まずは名前つきエクスポート／インポート文について見ていきましょう。

### [](#named-export-import)*名前つきエクスポート／インポート*

***名前つきエクスポート**は、モジュールごとに複数の変数や関数などをエクスポートできます。 次の例では、`foo`変数と`bar`関数をそれぞれ名前つきエクスポートしています。 `export`文のあとに続けて`{}`を書き、その中にエクスポートする変数を入れることで、宣言済みの変数を名前つきエクスポートできます。

named-export.js

```
const foo = "foo";
// 宣言済みのオブジェクトを名前つきエクスポートする
export { foo }; 
```

また、名前つきエクスポートでは`export`文を宣言の前につけると、宣言と同時に名前つきエクスポートできます。

named-export-declare.js

```
// 宣言と同時に名前つきエクスポートする
export function bar() { }; 
```

**名前つきインポート**は、指定したモジュールから名前を指定して選択的にインポートできます。 次の例では `my-module.js`から名前つきエクスポートされたオブジェクトの名前を指定して名前つきインポートしています。 `import`文のあとに続けて`{}`を書き、その中にインポートしたい名前つきエクスポートの名前を入れます。 複数の値をインポートしたい場合は、それぞれの名前をカンマで区切ります。

my-module.js

```
export const foo = "foo";
export function bar() { } 
```

named-import.js

```
// 名前つきエクスポートされたfooとbarをインポートする
import { foo, bar } from "./my-module.js";
console.log(foo); // => "foo"
console.log(bar); // => function bar() 
```

#### [](#named-export-import-alias)*名前つきエクスポート／インポートのエイリアス*

*名前つきエクスポート／インポートには**エイリアス**の仕組みがあります。 エイリアスを使うと、宣言済みの変数を違う名前で名前つきエクスポートできます。 エイリアスをつけるには、次のように`as`のあとにエクスポートしたい名前を記述します。

named-export-alias.js

```
const internalFoo = "foo";
// internalFoo 変数をfooとして名前つきエクスポートする
export { internalFoo as foo }; 
```

また、名前つきインポートしたオブジェクトにも別名をつけることができます。 インポートでも同様に、`as`のあとに別名を記述します。

named-import-alias.js

```
// fooとして名前つきエクスポートされた変数をmyFooとしてインポートする
import { foo as myFoo } from "./named-export-alias.js";
console.log(myFoo); // => "foo" 
```

### [](#default-export-import)*デフォルトエクスポート／インポート*

*次に、デフォルトエクスポート／インポートについて見ていきましょう。 **デフォルトエクスポート**は、モジュールごとに1つしかエクスポートできない特殊なエクスポートです。 次の例は、すでに宣言されている変数をデフォルトエクスポートしています。 `export default`文で、後に続く式の評価結果をデフォルトエクスポートします。

default-export.js

```
const foo = "foo";
// foo 変数の値をデフォルトエクスポートする
export default foo; 
```

また、`export`文を宣言の前につけると、宣言と同時にデフォルトエクスポートできます。 このとき関数やクラスの名前を省略できます。

```
// 宣言と同時に関数をデフォルトエクスポートする
export default function() {} 
```

ただし、変数宣言は宣言とデフォルトエクスポートを同時に行うことはできません。 なぜなら、変数宣言はカンマ区切りで複数の変数を定義できてしまうためです。 次の例は実行できない不正なコードです。

```
// 変数宣言と同時にデフォルトエクスポートはできない
export default const foo = "foo", bar = "bar"; 
```

**デフォルトインポート**は、指定したモジュールのデフォルトエクスポートに名前をつけてインポートします。 次の例では `my-module.js`のデフォルトエクスポートに`myModule`という名前をつけてインポートしています。 `import`文のあとに任意の名前をつけることで、デフォルトエクスポートをインポートできます。

my-module.js

```
export default {
    baz: "baz"
}; 
```

default-import.js

```
// デフォルトエクスポートをmyModuleとしてインポートする
import myModule from "./my-module.js";
console.log(myModule); // => { baz: "baz" } 
```

実はデフォルトエクスポートは、`default`という固有の名前による名前つきエクスポートと同じものです。 そのため、名前つきエクスポートで`as default`とエイリアスをつけることでデフォルトエクスポートすることもできます。

default-export-alias.js

```
const foo = "foo";
// foo 変数の値をデフォルトエクスポートする
export { foo as default }; 
```

类似地，在命名导入中，名称`default`对应默认导入。通过以下方式，在命名导入中指定`default`，您可以进行默认导入。但请注意，`default`是保留字，因此您必须使用`as`语法为其指定别名。

default-import-alias.js

```
// デフォルトエクスポートをmyModuleとしてインポートする
import { default as myModule } from "./my-module.js";
console.log(myModule); // => { baz: "baz" } 
```

此外，命名导入和默认导入的语法可以同时编写。您可以用逗号连接两种语法，如下所示。

default-import-with-named.js

```
// myModuleとしてデフォルトインポートし、
// fooを名前つきインポートする
import myModule, { foo } from "./my-module.js";
console.log(foo); // => "foo"
console.log(myModule); // => { baz: "baz" } 
```

在 ECMAScript 模块中，未导出的内容无法被导入。这是因为在 JavaScript 的解析阶段解析 ECMAScript 模块时，如果无法解析出要导入的内容，则会导致解析错误。默认导入要求导入目标模块导出默认导出。类似地，命名导入要求导入目标模块导出指定的命名导出。*

### [](#other-syntax)*其他语法*

*ECMAScript 模块还具有除了命名导入和默认导入之外的其他一些语法。*

#### [](#re-export)*再导出*

*再导出是指，重新从另一个模块导入的内容，再次从自身导出。它通常用于创建包含从多个模块导出的内容的模块。*

再导出的语法如下，在`export`关键字后面跟着`from`，然后指定另一个模块名。

```
// ./my-module.jsのすべての名前つきエクスポートを再エクスポートする
export * from "./my-module.js";
// [ES2020] ./my-module.jsのすべての名前つきエクスポートを名前空間オブジェクトとして再エクスポートする
export * as myNameSpace from "./my-module.js";
// ./my-module.jsの名前つきエクスポートを選んで再エクスポートする
export { foo, bar } from "./my-module.js";
// ./my-module.jsの名前つきエクスポートにエイリアスをつけて再エクスポートする
export { foo as myModuleFoo, bar as myModuleBar } from "./my-module.js";
// ./my-module.jsのデフォルトエクスポートをデフォルトエクスポートとして再エクスポートする
export { default } from "./my-module.js";
// ./my-module.jsのデフォルトエクスポートを名前つきエクスポートとして再エクスポートする
export { default as myModuleDefault } from "./my-module.js";
// ./my-module.jsの名前つきエクスポートをデフォルトエクスポートとして再エクスポートする
export { foo as default } from "./my-module.js"; 
```

#### [](#namespace-import)*导入所有内容*

*`import * as`语法会导入所有命名导出。使用这种方法，你声明了一个对象，该对象成为了模块的 **命名空间** 。要访问导出的变量或函数等，您需要使用该命名空间对象的属性。此外，正如前文所述，使用名为 `default` 的关键字，您还可以访问默认导出。*

my-module.js

```
export const foo = "foo";
export function bar() { }
export default {
    baz: "baz"
}; 
```

namespace-import.js

```
// すべての名前つきエクスポートをmyModuleオブジェクトとしてまとめてインポートする
import * as myModule from "./my-module.js";
// fooとして名前つきエクスポートされた値にアクセスする
console.log(myModule.foo); // => "foo"
// defaultとしてデフォルトエクスポートされた値にアクセスする
console.log(myModule.default); // => { baz: "baz" } 
```

#### [](#import-for-side-effect)*用于副作用的导入*

*有些模块仅执行全局代码，而不导出任何内容。例如，用于操作全局变量的模块等。*

side-effects.js

```
// グローバル変数を操作する(副作用)
window.foo = "foo"; 
```

要导入此类模块，您需要使用用于副作用的导入语法。使用此语法，您只需加载并执行指定的模块，而不导入任何内容。

```
// ./side-effects.jsのグローバルコードが実行される
import "./side-effects.js"; 
```

## [](#run-es-modules)*执行 ECMAScript 模块*

*要执行创建的 ECMAScript 模块，您需要将起始 JavaScript 文件作为 ECMAScript 模块加载到 Web 浏览器中。Web 浏览器会使用`script`元素加载和执行 JavaScript 文件。通过为`script`元素添加`type="module"`属性，Web 浏览器会将 JavaScript 文件视为 ECMAScript 模块加载。*

```
<!-- my-module.jsをECMAScriptモジュールとして読み込む -->
<script type="module" src="./my-module.js"></script>
<!-- インラインでも同じ -->
<script type="module"> import { foo } from "./my-module.js"; </script> 
```

如果没有添加`type="module"`属性，则会被视为普通脚本，不会启用 ECMAScript 模块的功能。如果在加载的 JavaScript 中使用了`import`或`export`语句，将会导致语法错误。

在 Web 浏览器环境中，导入的模块是通过网络解析的。因此，模块名称应该指定 JavaScript 文件的绝对 URL 或相对 URL。有关详细信息，请参阅 Todo 应用程序用例。*
