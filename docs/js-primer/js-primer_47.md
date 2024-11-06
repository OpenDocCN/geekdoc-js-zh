# MarkdownをHTMLに変換する

> 原文：[`jsprimer.net/use-case/nodecli/md-to-html/`](https://jsprimer.net/use-case/nodecli/md-to-html/)

前のセクションではコマンドライン引数で受け取ったファイルを読み込み、標準出力に表示しました。 次は読み込んだMarkdownファイルをHTMLに変換して、その結果を標準出力に表示してみましょう。

## [](#use-marked-package)*markedパッケージを使う*

*JavaScriptでMarkdownをHTMLへ変換するために、今回は[marked](https://github.com/markedjs/marked)というライブラリを使用します。 markedのパッケージはnpmで配布されているので、commanderと同様に`npm install`コマンドでパッケージをインストールしましょう。

```
$ npm install [[email protected]](/cdn-cgi/l/email-protection) 
```

インストールが完了したら、Node.jsのスクリプトから読み込みます。 前のセクションの最後で書いたスクリプトに、`marked`モジュールの読み込み処理を追加しましょう。 次のように`main.js`を変更し、読み込んだMarkdownファイルをmarkedを使ってHTMLに変換します。 `marked`モジュールからインポートした`marked.parse`関数は、Markdown 文字列を引数にとり、HTML 文字列に変換して返します。

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

## [](#create-convert-option)*変換オプションを作成する*

*markedにはMarkdownの[変換オプション](https://marked.js.org/#/USING_ADVANCED.md#options)があり、オプションの設定によって変換後のHTMLが変化します。 そこで、アプリケーション中でオプションのデフォルト値を決め、さらにコマンドライン引数から設定を切り替えられるようにしてみましょう。

今回のアプリケーションでは、例として`gfm`というmarkedのオプションを扱います。

### [](#gfm-option)*gfmオプション*

*`gfm`オプションは、GitHubにおけるMarkdownの仕様（[GitHub Flavored Markdown](https://github.github.com/gfm/), GFM）に合わせて変換するかを決めるオプションです。 markedではこの`gfm`オプションがデフォルトで`true`になっています。GFMは標準的なMarkdownにいくつかの拡張を加えたもので、代表的な拡張がURLの自動リンク化です。 次のように`sample.md`を変更し、先ほどのスクリプトと`gfm`オプションを`false`にしたスクリプトで結果の違いを見てみましょう。

sample.md

```
# サンプルファイル

これはサンプルです。
https://jsprimer.net/

- サンプル1
- サンプル2 
```

`gfm`オプションが有効のときは、URLの文字列が自動的に`<a>`タグのリンクに置き換わります。

```
<h1 id="サンプルファイル">サンプルファイル</h1>
<p>これはサンプルです。
<a href="https://jsprimer.net/">https://jsprimer.net/</a></p>
<ul>
<li>サンプル1</li>
<li>サンプル2</li>
</ul> 
```

一方、次のように`gfm`オプションを`false`にすると、単なる文字列として扱われ、リンクには置き換わりません。

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

自動リンクのほかにもいくつかの拡張がありますが、詳しくは[GitHub Flavored Markdown](https://github.github.com/gfm/)のドキュメントを参照してください。

### [](#receive-option)*コマンドライン引数からオプションを受け取る*

*次に、`gfm`オプションをコマンドライン引数で制御できるようにしましょう。 アプリケーションのデフォルトでは`gfm`オプションを無効にした上で、次のように`--gfm`オプションを付与してコマンドを実行できるようにします。

```
$ node main.js --gfm sample.md 
```

コマンドライン引数で`--gfm`のようなオプションを扱いたいときには、commanderの`option`メソッドを使います。 次のように必要なオプションを定義してからコマンドライン引数をパースすると、`program.opts`メソッドでパース結果のオブジェクトを取得できます。

```
// gfmオプションを定義する
program.option("--gfm", "GFMを有効にする");
// コマンドライン引数をパースする
program.parse(process.argv);
// オプションのパース結果をオブジェクトとして取得する
const options = program.opts();
console.log(options.gfm); 
```

`--gfm`オプションはファイルパスを指定する`sample.md`の前後のどちらについていても動作します。 なぜなら`program.args`配列に��`program.option`メソッドで定義したオプションが含まれないためです。 `process.argv`配列を直接使っているとこのようなオプションの処理が面倒なので、commanderのようなパース処理を挟むのが一般的です。

### [](#declare-default)*デフォルト設定を定義する*

*アプリケーション側でデフォルト設定を持っておくことで、将来的にmarkedの挙動が変わったときにも影響を受けにくくなります。 次のようにオプションを表現した`cliOptions`オブジェクトを作成し、`program.opts`メソッドの返り値から取得した値をセットします。 コマンドライン引数で指定されなかったオプションには`??`（Nullish coalescing 演算子）を使ってデフォルトの値をセットします。 Nullish coalescing 演算子は左辺がnullishであるときにだけ右辺の値を返すため、値が指定されなかった状態と明示的に`false`が与えられた状態を区別したいときに便利です。

```
// コマンドライン引数のオプションを取得する
const options = program.opts();

// コマンドライン引数で指定されなかったオプションにデフォルト値を上書きする
const cliOptions = {
    gfm: options.gfm ?? false,
}; 
```

こうして作成したcliOptionsオブジェクトを、markedの`parse`関数へオプションとして渡しましょう。`main.js`の全体は次のようになります。

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

定義したコマンドライン引数を使って、Markdownファイルを変換してみましょう。

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

また、`gfm`オプションを付与して実行すると次のように出力されるはずです。

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

これでMarkdown 変換の設定をコマンドライン引数でオプションとして与えられるようになりました。 次のセクションではアプリケーションのコードを整理し、最後にユニットテストを導入します。

## [](#section-checklist)*このセクションのチェックリスト*

**   markedパッケージを使ってMarkdown 文字列をHTML 文字列に変換した

+   コマンドライン引数でmarkedの変換オプションを設定した

+   デフォルトオプションを定義し、コマンドライン引数で上書きできるようにした******
