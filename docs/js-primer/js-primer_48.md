# ユニットテストを記述する

> 原文：[`jsprimer.net/use-case/nodecli/refactor-and-unittest/`](https://jsprimer.net/use-case/nodecli/refactor-and-unittest/)

このセクションでは、これまで作成したCLIアプリケーションにユニットテストを導入します。 ユニットテストの導入と合わせて、ソースコードを整理してテストがしやすくなるようにモジュール化します。

前のセクションまでは、すべての処理をひとつのJavaScriptファイルに記述していました。 ユニットテストを行うためにはテスト対象がモジュールとして分割されていなければいけません。 今回のアプリケーションでは、CLIアプリケーションとしてコマンドライン引数を処理する部分と、MarkdownをHTMLへ変換する部分に分割します。

## [](#split-script)*アプリケーションをモジュールに分割する*

*実際にアプリケーションのモジュール化をする前に、ECMAScriptモジュールにおけるエクスポートについて簡単に振り返ります。

ECMAScriptモジュールでは`export`文を使って変数や関数などのオブジェクトをエクスポートし、他のスクリプトから利用できるようにします。 次の`greet.js`というファイルは、`greet`関数をエクスポートするモジュールの例です。

greet.js

```
// greet.js
export function greet(name) {
    return `Hello ${name}!`;
}; 
```

このモジュールを利用する側では、`import`文を使って指定したファイルパスのJavaScriptファイルをインポートできます。 次のコードでは先ほどの`greet.js`のパスを指定してモジュールとしてインポートして、エクスポートされた`greet`関数を利用しています。

greet-main.js

```
import { greet } from "./greet.js";
greet("World"); // => "Hello World!" 
```

これから行うアプリケーションのモジュール化とは、このようにアプリケーションの一部分を別のファイルに切り出した上で、必要なオブジェクトをエクスポートして外部から利用可能にするということです。 機能をモジュールとして切り出すことで、アプリケーションとユニットテストの両方から利用できるようになります。

それではCLIアプリケーションのソースコードをモジュールに分割してみましょう。 `md2html.js`という名前のJavaScriptファイルを作成し、次のようにmarkedを使ったMarkdownの変換処理を記述します。

md2html.js

```
import { marked } from "marked";

export function md2html(markdown, cliOptions) {
    return marked.parse(markdown, {
        gfm: cliOptions.gfm,
    });
}; 
```

このモジュールがエクスポートするのは、与えられたオプションを元にMarkdown 文字列をHTMLに変換する関数です。 アプリケーションのエントリーポイントである`main.js`では、次のようにこのモジュールをインポートして使用します。

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

markedパッケージや、そのオプションに関する記述がひとつの`md2html`関数に隠蔽され、`main.js`がシンプルになりました。 そして`md2html.js`はアプリケーションから独立したひとつのモジュールとして切り出され、ユニットテストが可能になりました。

## [](#create-env)*ユニットテスト実行環境を作る*

*ユニットテストの実行にはさまざまな方法があります。 このセクションではテスティングフレームワークとして[Mocha](https://mochajs.org/)を使って、ユニットテストの実行環境を作成します。 Mochaが提供するテスト実行環境では、グローバルに`it`や`describe`などの関数が定義されます。 `it`関数はその内部でエラーが発生したとき、そのテストを失敗として扱います。 つまり、期待する結果と異なるならエラーを投げ、期待どおりならエラーを投げないというテストコードを書くことになります。

今回はNode.jsの標準モジュールのひとつである[assertモジュール](https://nodejs.org/api/assert.html)から提供される`assert.strictEqual`メソッドを利用します。 `assert.strictEqual`メソッドは第一引数と第二引数の評価結果が`===`で比較して異なる場合に、例外を投げる関数です。

Mochaによるテスト環境を作るために、まずは次のコマンドで`mocha`パッケージをインストールします。

```
$ npm install --save-dev mocha@10 
```

`--save-dev`オプションは、パッケージを`devDependencies`としてインストールするためのものです。 `package.json`の`devDependencies`には、そのパッケージを開発するときだけ必要な依存ライブラリを記述します。

ユニットテストを実行するには、Mochaが提供する`mocha`コマンドを使います。 Mochaをインストールした後、`package.json`の`scripts`プロパティを次のように記述します。

```
{
    ...
    "scripts": {
        "test": "mocha test/"
    },
    ...
} 
```

この記述により、`npm test`コマンドを実行すると、`mocha`コマンドで`test/`ディレクトリにあるテストファイルを実行します。 試しに`npm test`コマンドを実行し、Mochaによるテストが行われることを確認しましょう。 まだテストファイルを作っていないので、`Error: No test files found`というエラーが表示されます。

```
$ npm test
> mocha

 Error: No test files found 
```

## [](#write-unit-test)*��ニットテストを記述する*

*テストの実行環境ができたので、実際にユニットテストを記述します。 Mochaのユニットテストは`test`ディレクトリの中にJavaScriptファイルを配置して記述します。 `test/md2html-test.js`ファイルを作成し、`md2html.js`に対するユニットテストを次のように記述します。

`it`関数は第一引数にテストのタイトルを入れ、第二引数にテストの内容を記述します。

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

`it`関数で定義したユニットテストは、`md2html`関数の変換結果が期待するものになっているかをテストしています。 `test/fixtures`ディレクトリにはユニットテストで用いるファイルを配置しています。 今回は変換元のMarkdownファイルと、期待する変換結果のHTMLファイルが存在します。

次のように変換元のMarkdownファイルを`test/fixtures/sample.md`に配置します。

test/fixtures/sample.md

```
# サンプルファイル

これはサンプルです。
https://jsprimer.net/

- サンプル1
- サンプル2 
```

そして、期待する変換結果のHTMLファイルも`test/fixtures`ディレクトリに配置します。 `gfm`オプションの有無にあわせて、`expected.html`と`expected-gfm.html`の2つを次のように作成しましょう。

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

ユニットテストの準備ができたら、もう一度`npm test`コマンドを実行しましょう。2 件のテストが通れば成功です。

```
$ npm test
> mocha

  ✓ converts Markdown to HTML (GFM=false)
  ✓ converts Markdown to HTML (GFM=true)

  2 passing (31ms) 
```

ユニットテストが通らなかった場合は、次のことを確認してみましょう。

+   `test/fixtures`ディレクトリに`sample.md`と`expected.html`、`expected-gfm.html`というファイルを作成したか

+   それぞれのファイルは文字コードがUTF-8で、改行コードがLFになっているか

+   それぞれのファイルに余計な文字が入っていないか

たとえば、`npm test`を実行して次のようにテストが失敗している場合のエラーメッセージを見てみましょう。

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

このテスト結果では `converts Markdown to HTML (GFM=true)` というタイトルのテストが1つ失敗していることがわかります。 また、`+ actual - expected`には、`assert.strictEqual`で比較した結果が一致していない部分が表示されています。 この場合は、expected（期待する結果）の末尾に`;;;`という不要な文字列が入ってしまっているのが、テストが失敗している理由です。 そのため、`expected-gfm.html`ファイルを確認し不要な`;;;`という文字列を取り除けば、テストが通るようになるはずです。

## [](#reason-for-unit-test)*なぜユニットテストを行うのか*

*ユニットテストを実施することには多くの利点があります。 早期にバグが発見できることや、安心してリファクタリングを行えるようになるのはもちろんですが、 ユニットテストが可能な状態を保つこと自体に意味があります。 実際にテストを行わなくてもテストしやすいコードになるよう心がけることが、アプリケーションを適切にモジュール化する指針になります。

またユニットテストには生きたドキュメントとしての側面もあります。 ドキュメントはこまめにメンテナンスされないとすぐに実際のコードと齟齬が生まれてしまいますが、 ユニットテストはそのモジュールが満たすべき仕様を表すドキュメントとして機能します。

ユニットテストの記述は手間がかかるだけのようにも思えますが、 中長期的にアプリケーションをメンテナンスする場合にはかかせないものです。 そしてよいテストを書くためには、日頃からテストを書く習慣をつけておくことが重要です。

## [](#unit-test-summary)*まとめ*

*このユースケースの目標であるNode.jsを使ったCLIアプリケーションの作成と、ユニットテストの導入ができました。 npmを使ったパッケージ管理や外部モジュールの利用、`fs`モジュールを使ったファイル操作など、多くの要素が登場しました。 これらはNode.jsアプリケーション開発においてほとんどのユースケースで応用されるものなので、よく理解しておきましょう。

## [](#section-checklist)*このセクションのチェックリスト*

**   Markdownの変換処理をECMAScriptモジュールとして`md2html.js`に切り出し、`main.js`から読み込んだ

+   mochaパッケージをインストールし、`npm test`コマンドで`mocha`コマンドを実行できることを確認した

+   `md2html`関数のユニットテストを作成し、テストの実行結果を確認した******
