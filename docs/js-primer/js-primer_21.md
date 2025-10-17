# 文字列

> 原文：[`jsprimer.net/basic/string/`](https://jsprimer.net/basic/string/)

この章ではJavaScriptにおける文字列について学んでいきます。 まずは、文字列の作成方法や文字列の操作方法について見ていきます。 そして、文字列を編集して自由に文字列を作れるようになることがこの章の目的です。

## *文字列を作成する*

*文字列を作成するには、文字列リテラルを利用します。 「データ型とリテラル」の章でも紹介しましたが、文字列リテラルには`"`（ダブルクォート）、`'`（シングルクォート）、```（バッククォート）の3 種類があります。

まずは`"`（ダブルクォート）と`'`（シングルクォート）について見ていきます。

`"`（ダブルクォート）と`'`（シングルクォート）に意味的な違いはありません。 そのため、どちらを使うかは好みやプロジェクトごとのコーディング規約によって異なります。 この書籍では、`"`（ダブルクォート）を主な文字列リテラルとして利用します。

```

const double = "文字列";

console.log(double); // => "文字列"

const single = '文字列';

console.log(single); // => '文字列'

// どちらも同じ文字列

console.log(double === single);// => true

```

ES2015では、テンプレートリテラル ```（バッククォート）が追加されました。 ```（バッククォート）を利用することで文字列を作成できる点は、他の文字列リテラルと同じです。

これに加えてテンプレートリテラルでは、文字列中に改行を入力できます。 次のコードでは、テンプレートリテラルを使って複数行の文字列を見た目どおりに定義しています。

```

const multiline = `1 行目

2 行目

3 行目`;

// \n は改行を意味する

console.log(multiline); // => "1 行目\n2 行目\n3 行目"

```

どの文字列リテラルでも共通ですが、文字列リテラルは同じ記号が対になります。 そのため、文字列の中にリテラルと同じ記号が出現した場合は、`\`（バックスラッシュ）を使いエスケープする必要があります。 次のコードでは、文字列中の`"`を`\"`のようにエスケープしています。

```

const str = "This book is \"js-primer\"";

console.log(str); // => 'This book is "js-primer"'

```

## *エスケープシーケンス*

 *文字列リテラル中にはそのままでは入力できない特殊な文字もあります。 改行もそのひとつで、`"`（ダブルクォート）と`'`（シングルクォート）の文字列リテラルには改行をそのまま入力できません （テンプレートリテラル中には例外的に改行をそのまま入力できます）。

次のコードは、JavaScriptの構文として正しくないため、構文エラー（SyntaxError）となります。

```

// JavaScriptエンジンが構文として解釈できないため、SyntaxErrorとなる

const invalidString = "1 行目

2 行目

3 行目";

```

この問題を回避するためには、改行のような特殊な文字をエスケープシーケンスとして書く必要があります。 エスケープシーケンスは、`\`と特定の文字を組み合わせることで、特殊文字を表現します。

次の表では、代表的な[エスケープシーケンス](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/String#%E3%82%A8%E3%82%B9%E3%82%B1%E3%83%BC%E3%83%97%E3%82%B7%E3%83%BC%E3%82%B1%E3%83%B3%E3%82%B9)を紹介しています。 エスケープシーケンスは、`"`（ダブルクォート）、`'`（シングルクォート）、```（バッククォート）すべての文字列リテラルの中で利用できます。

| エスケープシーケンス | 意味 |
| --- | --- |
| `\'` | シングルクォート |
| `\"` | ダブルクォート |
| `\`` | バッククォート |
| `\\` | バックスラッシュ(`\`そのものを表示する) |
| `\n` | 改行 |
| `\t` | タブ |
| `\uXXXX` | Code Unit(`\u`と4 桁のHexDigit) |
| `\u{X}` ... `\u{XXXXXX}` | Code Point（`\u{}`のカッコ中にHexDigit） |

このエスケープシーケンスを利用することで、先ほどの`"`（ダブルクォート）の中に改行（`\n`）を入力できます。

```
// 改行を\nのエスケープシーケンスとして入力している
const multiline = "1 行目\n2 行目\n3 行目";
console.log(multiline); 
/* 改行した結果が出力される
1 行目
2 行目
3 行目
*/ 
```

また、`\`からはじまる文字は自動的にエスケープシーケンスとして扱われます。 しかし、`\a`のように定義されていないエスケープシーケンスは、`\`が単に無視され`a`という文字列として扱われます。 これにより、`\`（バックスラッシュ）そのものを入力していたつもりが、その文字がエスケープシーケンスとして扱われてしまう問題があります。

次のコードでは、`\_`という組み合わせのエスケープシーケンスはないため、`\`が無視された文字列として評価されます。

```
console.log("¯\_(ツ)_/¯");
// ¯_(ツ)_/¯ のように\が無視されて表示される 
```

`\`（バックスラッシュ）そのものを入力したい場合は、`\\`のようにエスケープする必要があります。

```
console.log("¯\\_(ツ)_/¯");
//　¯\_(ツ)_/¯ と表示される 
```

## *文字列を結合する*

*文字列を結合する簡単な方法は文字列結合演算子（`+`）を使う方法です。

```
const str = "a" + "b";
console.log(str); // => "ab" 
```

変数と文字列を結合したい場合も文字列結合演算子で行えます。

```
const name = "JavaScript";
console.log("Hello " + name + "!");// => "Hello JavaScript!" 
```

特定の書式に文字列を埋め込むには、テンプレートリテラルを使うとより宣言的に書けます。

テンプレートリテラル中に`${変数名}`で書かれた変数は評価時に展開されます。 つまり、先ほどの文字列結合は次のように書けます。

```
const name = "JavaScript";
console.log(`Hello ${name}!`);// => "Hello JavaScript!" 
```

## *文字へのアクセス*

*文字列の特定の位置にある文字にはインデックスを指定してアクセスできます。 これは、配列における要素へのアクセスにインデックスを指定するのと同じです。

文字列では`文字列[インデックス]`のように指定した位置（インデックス）の文字へアクセスできます。 インデックスの値は`0`以上`2⁵³ - 1`未満の整数が指定できます。

```
const str = "文字列";
// 配列と同じようにインデックスでアクセスできる
console.log(str[0]); // => "文"
console.log(str[1]); // => "字"
console.log(str[2]); // => "列" 
```

また、存在しないインデックスへのアクセスでは配列やオブジェクトと同じように`undefined`を返します。

```
const str = "文字列";
// 42 番目の要素は存在しない
console.log(str[42]); // => undefined 
```

### *[ES2022] `String.prototype.at`*

*ES2022から`String.prototype.at`メソッドが追加されています。 Stringの`at`メソッドは、Arrayの`at`メソッドと同じく、相対的なインデックスを渡してその位置の文字へアクセスできます。 `at`メソッドへ`-1`のようにマイナスのインデックスを渡した場合は、末尾から数えた位置の文字へアクセスできます。

```
const str = "文字列";
console.log(str.at(0)); // => "文"
console.log(str.at(1)); // => "字"
console.log(str.at(2)); // => "列"
console.log(str.at(-1)); // => "列" 
```

## *文字列とは*

*今まで何気なく「文字列」という言葉を利用していましたが、ここでいう文字列とはどのようなものでしょうか？　コンピュータのメモリ上に文字列の「ア」といった文字をそのまま保存できないため、0と1からなるビット列へ変換する必要があります。 この文字からビット列へ変換することを符号化（エンコード）と呼びます。

一方で、変換後のビット列が何の文字なのかを管理する表が必要になります。 この文字に対応するIDの一覧表のことを符号化文字集合と呼びます。

次の表は、Unicodeという文字コードにおける符号化文字集合からカタカナの一部分を取り出したものです。¹ Unicodeはすべての文字に対してID（Code Point）を振ることを目的に作成されている仕様です。

|  | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | A | B | C | D | E | F |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 30A0 | ら | り | り | り | り | り | り | り | り | り | り | り | り | り | り | り |
| 30B0 | れ | む | め | ろ | ろ | ろ | ろ | ろ | ろ | ろ | ろ | ろ | ろ | ろ | ろ | ろ |
| 30C0 | ダ | チ | ヂ | ッ | ��� | ヅ | テ | デ | �� | ド | ナ | ニ | ヌ | ネ | ノ | ハ |

JavaScript（ECMAScript）采用 Unicode 作为字符编码，并采用 UTF-16 作为编码文字的方式。UTF-16 是将每个字符转换为 16 位位序列的编码方式。在 Unicode 中，用于表示 1 个字符的最小位组合称为**Code Unit**（符号单元），UTF-16 中每个 Code Unit 的大小为 16 位（2 字节）。

下面的代码是显示构成字符串的 Code Unit 的 hex 值（十六进制）的示例。String 的`charCodeAt`方法返回字符串指定索引的 Code Unit 的整数。然后使用 Number 的`toString`方法将这个 Code Unit 的整数值转换为 hex 值（十六进制）。

```
const str = "アオイ";
// それぞれの文字をCode Unitのhex 値（16 進数）に変換する
// toStringの引数に16を渡すと16 進数に変換される
console.log(str.charCodeAt(0).toString(16)); // => "30a2"
console.log(str.charCodeAt(1).toString(16)); // => "30aa"
console.log(str.charCodeAt(2).toString(16));  // => "30a4" 
```

相反，要将 Code Unit 转换为 hex 值（十六进制）并转换为字符，可以使用`String.fromCharCode`方法。下面的代码展示了如何将用`0x`表示的十六进制整数字面量转换为字符串（有关`0x`字面量，请参阅“数据类型和字面量”章节）。

```
const str = String.fromCharCode(
    0x30a2, // アのCode Unit
    0x30aa, // オのCode Unit
    0x30a4  // イのCode Unit
);
console.log(str); // => "アオイ" 
```

总结这些结果，这个字符串与构成它的 UTF-16 的 Code Unit 之间的关系如下。

| 索引 | 0 | 1 | 2 |
| --- | --- | --- | --- |
| 文字串 | ア | オ | イ |
| UTF-16 的 Code Unit（十六进制） | 0x30A2 | 0x30AA | 0x30A4 |

这样，JavaScript 中的字符串在内部以 16 位的 Code Unit 顺序排列。这仅仅是 ECMAScript 的内部表示采用 UTF-16，与 JavaScript 文件（编写的源代码文件）的编码无关。因此，即使 JavaScript 文件的编码不是 UTF-16，也没有问题。

UTF-16 的使用似乎是 JavaScript 的内部表示，因此似乎没有必要担心。然而，JavaScript 使用 UTF-16 的事实对 String 的 API 也有影响。关于 UTF-16 和字符串，我们将在下一章“字符串与 Unicode”中详细探讨。

在这里，只需记住“JavaScript 中的字符串由 UTF-16 的 Code Unit 组成”这一点就足够了。

## *字符串的分解与连接*

*要将字符串分解为数组，可以使用 String 的`split`方法。另一方面，要将数组元素连接成字符串，可以使用 Array 的`join`方法。

这两个经常一起使用，所以我们一起来了解一下。

String 的`split`方法返回一个使用指定的分隔符分割的字符串的数组。下面的代码展示了如何创建一个用`・`分割的数组。

```
const strings = "赤・青・緑".split("・");
console.log(strings); // => ["赤", "青", "緑"] 
```

在将分解后的字符串数组组合成字符串时，可以使用 Array 的`join`方法。Array 的`join`方法的第一个参数是分隔符，它将使用该分隔符将字符串连接起来并返回。

将这两个结合起来，可以将分隔符从`・`转换为`、`的处理过程写成如下。首先用`・`分割字符串（`split`），然后用`、`连接（`join`）即可完成转换。

```
const str = "赤・青・緑".split("・").join("、");
console.log(str); // => "赤、青、緑" 
```

String 的`split`方法的第一个参数可以是正则表达式。利用这一点，可以轻松地编写如下分割字符串的处理过程。`/\s+/`是一个匹配一个或多个空格的正则表达式字面量，它创建了一个匹配该正则表达式的正则表达式对象。

```
// 文字間に1つ以上のスペースがある
const str = "a     b    c      d";
// 1つ以上のスペースにマッチして分解する
const strings = str.split(/\s+/);
console.log(strings); // => ["a", "b", "c", "d"] 
```

## *字符串的长度*

*String 的`length`属性返回字符串的元素数。由于字符串的构成元素是 Code Unit，因此`length`属性返回 Code Unit 的数量。

由于以下字符串由 3 个 Code Unit 组成，因此`length`属性返回`3`。

```
console.log("文字列".length); // => 3 
```

另外，由于空字符串的元素数量为`0`，因此`length`属性的结果也是`0`。

```
console.log("".length); // => 0 
```

## *字符串的比较*

*使用`===`（严格比较运算符）进行字符串比较。如果满足以下条件，则字符串相同。

+   字符串的元素 Code Unit 是否按相同的顺序排列

+   文字的长度（length）是否相同

虽然写得复杂，但如果字符串相同，使用`===`（严格比较运算符）的结果将是`true`。

```
console.log("文字列" === "文字列"); // => true
// 一致しなければfalseとなる
console.log("JS" === "ES"); // => false
// 文字列の長さが異なるのでfalseとなる
console.log("文字列" === "文字"); // => false 
```

此外，除了`===`等比较运算符之外，还可以使用`>`、`<`、`>=`、`<=`等大小关系运算符来比较字符串。

这些关系运算符也按顺序比较字符串的元素 Code Unit。要从字符串中获取 Code Unit 的数值，可以使用 String 的`charCodeAt`方法。

在下面的代码中，比较了`ABC`和`ABD`哪个更大（Code Unit 的值更大）。

```
// "A"と"B"のCode Unitは65と66
console.log("A".charCodeAt(0)); // => 65
console.log("B".charCodeAt(0)); // => 66
// "A"（65）は"B"（66）よりCode Unitの値が小さい
console.log("A" > "B"); // => false
// 先頭から順番に比較し C > D が falseであるため
console.log("ABC" > "ABD"); // => false 
```

这样，关系运算符中的字符串比较是针对 Code Unit 进行的。预测这个结果很困难，而且产生的结果往往不直观。因为字符的顺序可能因国家或语言而异，所以也需要国际化（Internationalization）相关的知识。

在 JavaScript 中，也有一个与 ECMAScript 相关的 ECMA-402 这样的 ECMAScript 相关规范，它规定了国际化。还有一个名为[Intl](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Intl)的内置对象定义了这些国际化 API，但关于这些 API 的详细内容将省略。

## *获取字符串的一部分*

*要从字符串中提取一部分，可以使用 String 的`slice`方法或`substring`方法。

`slice`方法已经在数组中学习过，但基本操作在字符串中也是相同的。首先来看一下`slice`方法。

Stringの`slice`メソッドは、第一引数の開始位置から第二引数の終了位置（終了位置の要素は含まない）までの範囲を取り出した新しい文字列を返します。 第二引数を省略した場合の挙動も同様で、省略した場合は文字列の末尾まで含んだ新しい文字列を返します。

位置にマイナスの値を指定した場合は文字列の末尾から数えた位置となります。 また、第一引数の位置が第二引数の位置より大きい場合、常に空の文字列を返します。

そのため、メソッドの引数の扱い方は配列の`slice`メソッドと同様です。

```
const str = "ABCDE";
console.log(str.slice(1)); // => "BCDE"
console.log(str.slice(1, 5)); // => "BCDE"
// マイナスを指定すると後ろからの位置となる
console.log(str.slice(-1)); // => "E"
// インデックスが1から4の範囲を取り出す
console.log(str.slice(1, 4)); // => "BCD"
// 第一引数 > 第二引数の場合、常に空文字列を返す
console.log(str.slice(4, 1)); // => "" 
```

Stringの`substring`メソッドは、`slice`メソッドと同じく第一引数に開始位置、第二引数に終了位置を指定し、その範囲を取り出して新しい文字列を返します。 第二引数を省略した場合の挙動も同様で、省略した場合は文字列の末尾が終了位置となります。

`slice`メソッドとは異なる点として、位置にマイナスの値を指定した場合は常に`0`として扱われます。 また、第一引数の位置が第二引数の位置より大きい場合、第一引数と第二引数が入れ替わるという予想しにくい挙動となります。

```
const str = "ABCDE";
console.log(str.substring(1)); // => "BCDE"
console.log(str.substring(1, 5)); // => "BCDE"
// マイナスを指定すると0として扱われる
console.log(str.substring(-1)); // => "ABCDE"
// 位置:1から4の範囲を取り出す
console.log(str.substring(1, 4)); // => "BCD"
// 第一引数 > 第二引数の場合、引数が入れ替わる
// str.substring(1, 4)と同じ結果になる
console.log(str.substring(4, 1)); // => "BCD" 
```

このように、マイナスの位置や引数が交換される挙動はわかりやすいものとは言えません。 そのため、`slice`メソッドと`substring`メソッドに指定する引数は、どちらとも同じ結果となる範囲に限定したほうが直感的な挙動となります。 つまり、指定するインデックスは0 以上にして、第二引数を指定する場合は`第一引数の位置 < 第二引数の位置`にするということです。

Stringの`slice`メソッドは、`indexOf`メソッドなどの位置を取得するものと組み合わせて使うことが多いでしょう。 次のコードでは、`?`の位置を`indexOf`メソッドで取得し、それ以降の文字列を`slice`メソッドで切り出しています。

```
const url = "https://example.com?param=1";
const indexOfQuery = url.indexOf("?");
const queryString = url.slice(indexOfQuery);
console.log(queryString); // => "?param=1" 
```

また、配列とは異なりプリミティブ型の値である文字列は、`slice`メソッドと`substring`メソッド共に非破壊的です。 機能的な違いがほとんどないため、どちらを利用するかは好みの問題となります。

## *文字列の検索*

*文字列の検索方法として、大きく分けて文字列による検索と正規表現による検索があります。*

指定した文字列が文字列中に含まれているかを検索する方法として、Stringメソッドには取得したい結果ごとにメソッドが用意���れています。 ここでは、次の3 種類の結果を取得する方法について文字列と正規表現それぞれの検索方法を見ていきます。

+   マッチした箇所のインデックスを取得

+   マッチした文字列の取得

+   マッチしたかどうかの真偽値を取得

### *文字列による検索*

*`String`オブジェクトには、指定した文字列で検索するメソッドが用意されています。*

#### *文字列によるインデックスの取得*

*Stringの`indexOf`メソッドと`lastIndexOf`メソッドは、指定した文字列で検索し、その文字列が最初に現れたインデックスを返します。 これらは配列のArrayの`indexOf`メソッドと同じで、厳密等価演算子（`===`）で文字列を検索します。 一致する文字列がない場合は`-1`を返します。*

+   `文字列.indexOf("検索文字列")`: 先頭から検索し、指定された文字列が最初に現れたインデックスを返す

+   `文字列.lastIndexOf("検索文字列")`: 末尾から検索し、指定された文字列が最初に現れたインデックスを返す

どちらのメソッドも一致する文字列が複数個ある場合でも、指定した検索文字列を最初に見つけた時点で検索は��了します。

```
// 検索対象となる文字列
const str = "にわにはにわにわとりがいる";
// indexOfは先頭から検索しインデックスを返す - "**にわ**にはにわにわとりがいる"
// "にわ"の先頭のインデックスを返すため 0 となる
console.log(str.indexOf("にわ")); // => 0
// lastIndexOfは末尾から検索しインデックスを返す- "にわにはにわ**にわ**とりがいる"
console.log(str.lastIndexOf("にわ")); // => 6
// 指定した文字列が見つからない場合は -1 を返す
console.log(str.indexOf("未知のキーワード")); // => -1 
```

### *文字列にマッチした文字列の取得*

*文字列を検索してマッチした文字列は、検索文字列そのものになるので自明です。*

次のコードでは`"Script"`という文字列で検索していますが、その検索文字列にマッチする文字列はもちろん`"Script"`になります。

```
const str = "JavaScript";
const searchWord = "Script";
const index = str.indexOf(searchWord);
if (index !== -1) {
    console.log(`${searchWord}が見つかりました`);
} else {
    console.log(`${searchWord}は見つかりませんでした`);
} 
```

#### *真偽値の取得*

*「文字列」に「検索文字列」が含まれているかを検索する方法がいくつか用意されています。 次の3つのメソッドはES2015で導入されました。*

+   `String.prototype.startsWith(検索文字列)`^([ES2015]): 検索文字列が先頭にあるかの真偽値を返す

+   `String.prototype.endsWith(検索文字列)`^([ES2015]): 検索文字列が末尾にあるかの真偽値を返す

+   `String.prototype.includes(検索文字列)`^([ES2015]): 検索文字列を含むかの真偽値を返す

具体的な例をいくつか見てみましょう。

```
// 検索対象となる文字列
const str = "にわにはにわにわとりがいる";
// startsWith - 検索文字列が先頭ならtrue
console.log(str.startsWith("にわ")); // => true
console.log(str.startsWith("いる")); // => false
// endsWith - 検索文字列が末尾ならtrue
console.log(str.endsWith("にわ")); // => false
console.log(str.endsWith("いる")); // => true
// includes - 検索文字列が含まれるならtrue
console.log(str.includes("にわ")); // => true
console.log(str.includes("いる")); // => true 
```

## *正規表現オブジェクト*

*文字列による検索では、固定の文字列にマッチするものしか検索できません。 一方で正規表現による検索では、あるパターン（規則性）にマッチするという柔軟な検索ができます。*

正規表現は正規表現オブジェクト（`RegExp`オブジェクト）として表現されます。 正規表現オブジェクトはマッチする範囲を決める`パターン`と正規表現の検索モードを指定する`フラグ`の2つで構成されます。 正規表現のパターン内では、次の文字は**特殊文字**と呼ばれ、特別な意味を持ちます。特殊文字として解釈されないように入力する場合には`\`（バックスラッシュ）でエスケープする必要があります。

```
\ ^ $ . * + ? ( ) [ ] { } | 
```

正規表現オブジェクトを作成するには、正規表現リテラルと`RegExp`コンストラクタを使う2つの方法があります。

```
// 正規表現リテラルで正規表現オブジェクトを作成
const patternA = /パターン/フラグ;
// `RegExp`コンストラクタで正規表現オブジェクトを作成
const patternB = new RegExp("パターン文字列", "フラグ"); 
```

正規表現リテラルは、`/`と`/`のリテラル内に正規表現のパターンを書くことで、正規表現オブジェクトを作成できます。 次のコードでは、`+`という1 回以上の繰り返しを意味する特殊文字を使い、`a`が1 回以上連続する文字列にマッチする正規表現オブジェクトを作成しています。

```
const pattern = /a+/; 
```

正規表現オブジェクトを作成するもうひとつの方法として`RegExp`コンストラクタがあります。 `RegExp`コンストラクタでは、文字列から正規表現オブジェクトを作成できます。

次のコードでは、`RegExp`コンストラクタを使って`a`が1 文字以上連続している文字列にマッチする正規表現オブジェクトを作成しています。 これは先ほどの正規表現リテラルで作成した正規表現オブジェクトと同じ意味になります。

```
const pattern = new RegExp("a+"); 
```

### *正規表現リテラルと`RegExp`コンストラクタの違い*

*正则表达式字面量和`RegExp`构造函数的区别在于正则表达式模式的评估时机不同。正则表达式字面量在源代码加载（解析）阶段评估正则表达式模式。另一方面，`RegExp`构造函数与普通函数一样，只有调用`RegExp`构造函数时才会评估正则表达式模式。*

以单独的 `[` 为例，这是一个无效的模式正则表达式，我们来看看评估时机的不同之处。由于 `[` 需要与 `]` 配对使用，因此在正则表达式模式中单独使用会导致语法错误异常。

正则表达式字面量在代码加载时会评估正则表达式模式，因此，即使没有调用 `main` 函数，也会导致语法错误（`SyntaxError`）。

```
// 正規表現リテラルはロード時にパターンが評価され、例外が発生する
function main() {
    // `[`は対となる`]`を組み合わせる特殊文字であるため、単独で書けない
    const invalidPattern = /[/;
}

// `main`関数を呼び出さなくても例外が発生する 
```

另一方面，`RegExp` 构造函数在运行时评估正则表达式模式，因此只有调用 `main` 函数时才会出现语法错误（`SyntaxError`）。

```
// `RegExp`コンストラクタは実行時にパターンが評価され、例外が発生する
function main() {
    // `[`は対となる`]`を組み合わせる特殊文字であるため、単独で書けない
    const invalidPattern = new RegExp("");
}

// `main`関数を呼び出すことで初めて例外が発生する
main(); 
```

这意味着，正则表达式字面量是在编写代码时创建固定模式的正则表达式对象。另一方面，`RegExp` 构造函数可以创建在运行时可能变化的模式的正则表达式对象，例如与变量结合使用。

例如，我们将比较使用正则表达式对象匹配连续空格的情况。

在下面的代码中，使用正则表达式字面量创建一个匹配连续 3 个空格的正则表达式对象。`\s` 匹配空格或制表符等空白字符。另外，`{number}` 表示重复指定次数的特殊字符。

```
// 3つの連続するスペースなどにマッチする正規表現
const pattern = /\s{3}/; 
```

正则表达式字面量在加载时评估正则表达式模式，因此无法动态更改 `\s` 连续出现的次数。另一方面，`RegExp` 构造函数在运行时评估正则表达式模式，因此可以创建包含变量的正则表达式对象。

在下面的代码中，使用 `RegExp` 构造函数根据变量 `spaceCount` 的数量创建一个匹配连续空格的正则表达式对象。需要注意的是，`\`（反斜杠）本身在字符串中是转义字符，请注意在 `RegExp` 构造函数的模式字符串中，以 `\`（反斜杠）开头的特殊字符需要转义。

```
const spaceCount = 3;
// `/\s{3}/`の正規表現を文字列から作成する
// "\"がエスケープ文字であるため、"\"自身を文字列として書くには、"\\"のように2つ書く
const pattern = new RegExp(`\\s{${spaceCount}}`); 
```

因此，`RegExp` 构造函数可以从字符串创建正则表达式对象，但需要对特殊字符进行转义。因此，在可以使用正则表达式字面量的情况下，最好使用字面量，因为它更简洁且性能更好。当需要在正则表达式模式中使用变量时，可以使用 `RegExp` 构造函数。

### *通过正则表达式搜索*

*通过正则表达式搜索，使用与 String 对象或 RegExp 对象对应的方法。

#### [*通过正则表达式获取索引*

*String 的 `indexOf` 方法的正则表达式版本是 String 的 `search` 方法。`search` 方法返回与正则表达式模式匹配的位置的索引，如果没有匹配的字符串，则返回 `-1`。

+   `String.prototype.indexOf(searchString)`：返回匹配的字符串的索引

+   `String.prototype.search(/pattern/`)：返回与指定正则表达式模式匹配的位置的索引

在下面的代码中，搜索连续 3 个数字，并返回匹配位置的索引。`\d` 是匹配单个数字（`0` 到 `9`）的特殊字符。

```
const str = "ABC123EFG";
const searchPattern = /\d{3}/;
console.log(str.search(searchPattern)); // => 3 
```

#### *通过正则表达式匹配的字符串获取*

*通过字符串搜索，搜索到的字符串本身就是匹配的字符串。然而，使用 `search` 方法的正则表达式搜索是基于正则表达式模式的搜索，因此搜索到的匹配字符串的长度是不固定的。换句话说，即使只获取 `search` 方法返回的匹配索引，实际匹配的字符串也无法得知。

```
const str = "abc123def";
// 連続した数字にマッチする正規表現
const searchPattern = /\d+/;
const index = str.search(searchPattern); // => 3
// `index` だけではマッチした文字列の長さがわからない
str.slice(index, index + マッチした文字列の長さ); // マッチした文字列は取得できない 
```

因此，提供了获取匹配字符串的 String 的 `match` 方法和 `matchAll` 方法。此外，这些方法通过正则表达式的全局匹配 `g` 标志来改变行为。

##### *匹配的字符串获取*

*首先，让我们从 String 的 `match` 方法开始查看获取匹配的字符串。`match` 方法用于在正则表达式 `/pattern/` 与 `"string"` 匹配时返回有关匹配字符串的信息。

```
"文字列".match(/パターン/); 
```

如果使用 `match` 方法搜索，但没有匹配正则表达式的字符串，则返回 `null`。

```
console.log("文字列".match(/マッチしないパターン/)); // => null 
```

当使用不带 `g` 标志的正则表达式模式进行搜索时，`match` 方法会在找到第一个匹配项时结束搜索。此时，`match` 方法的返回值是一个特殊数组，包含 `index` 属性和 `input` 属性。`index` 属性包含匹配字符串的起始索引，`input` 属性包含整个搜索字符串。

下面的代码中的 `/[a-zA-Z]+/` 正则表达式将匹配一个或多个连续的 `a` 到 `Z` 之间的字符。这个正则表达式匹配的字符串可以通过索引访问返回的数组中获取。没有 `g` 标志时，只有一个匹配项时搜索会结束，因此返回的数组只包含一个元素。

```
const str = "ABC あいう DE えお";
const alphabetsPattern = /[a-zA-Z]+/;
// gフラグなしでは、最初の結果のみを含んだ特殊な配列を返す
const results = str.match(alphabetsPattern);
console.log(results.length); // => 1
// マッチした文字列はインデックスでアクセスできる
console.log(results[0]); // => "ABC"
// マッチした文字列の先頭のインデックス
console.log(results.index); // => 0
// 検索対象となった文字列全体
console.log(results.input); // => "ABC あいう DE えお" 
```

`match` 方法在使用带有 `g` 标志的正则表达式模式进行搜索时，将返回包含所有匹配字符串的数组。

下面的代码中的 `/[a-zA-Z]+/g` 正则表达式将匹配一个或多个连续的 `a` 到 `Z` 之间的字符。这个正则表达式匹配的位置是 "ABC" 和 "DE"，因此 `match` 方法返回的数组中包含 2 个元素。

```
const str = "ABC あいう DE えお";
const alphabetsPattern = /[a-zA-Z]+/g;
// gフラグありでは、すべての検索結果を含む配列を返す
const resultsWithG = str.match(alphabetsPattern);
console.log(resultsWithG.length); // => 2
console.log(resultsWithG[0]); // => "ABC"
console.log(resultsWithG[1]); // => "DE"
// indexとinputはgフラグありの場合は追加されない
console.log(resultsWithG.index); // => undefined
console.log(resultsWithG.input); // => undefined 
```

在这种情况下，`match` 方法返回的数组不包含 `index` 和 `input` 属性。这是因为在多个匹配位置的情况下，单个 `index` 属性无法唯一确定意义。

总结 String 的 `match` 方法的行为如下。

+   如果没有匹配，则返回 `null`

+   如果匹配成功，则返回包含匹配字符串的特殊数组

+   当使用正则表达式的 `g` 标志时，返回包含所有匹配结果的普通数组

在 ES2020 中，引入了 String 的 `matchAll` 方法，用于使用正则表达式的 `g` 标志进行重复匹配。`matchAll` 方法返回一个 Iterator，其中包含匹配的结果。

下面的代码获取了与字母匹配的结果的 Iterator 对象。Iterator 对象可以通过 `for...of` 语法进行迭代处理，从而逐个获取 Iterator 中的值进行处理（有关详细信息，请参阅“循环和迭代”章节）。在这种迭代处理中，可以获取每个匹配的字符串以及具有 `index` 和 `input` 属性的特殊数组。

```
const str = "ABC あいう DE えお";
const alphabetsPattern = /[a-zA-Z]+/g;
// matchAllはIteratorを返す
const matchesIterator = str.matchAll(alphabetsPattern);
for (const match of matchesIterator) {
    // マッチした要素ごとの情報を含んでいる
    console.log(`match: "${match[0]}", index: ${match.index}, input: "${match.input}"`);
}
// 次の順番でコンソールに出力される
// match: "ABC", index: 0, input: "ABC あいう DE えお"
// match: "DE", index: 8, input: "ABC あいう DE えお" 
```

因此，当使用正则表达式的 `g` 标志进行重复匹配时，应该使用 `matchAll` 方法，而不是 `match` 方法。另外，由于 `matchAll` 方法不支持没有 `g` 标志的正则表达式，因此如果传递没有 `g` 标志的正则表达式，则会引发异常。

#### *获取匹配字符串的一部分*

*String 的 `match` 方法和 `matchAll` 方法都支持正则表达式的捕获。捕获是指在正则表达式中使用 `/pattern1(pattern2)/` 这样的括号括起来的部分。通过这种捕获，可以提取正则表达式匹配的部分。

`match` 方法和 `matchAll` 方法都将匹配结果作为数组返回。

如果匹配的模式中包含捕获，则返回的数组将逐渐添加捕获的部分。数组的开头包含整个匹配的字符串，然后按顺序包含捕获（用 `(` 和 `)` 括起来的部分）。

```
const [マッチした全体の文字列, キャプチャ1, キャプチャ2] = 文字列.match(/パターン(キャプチャ1)と(キャプチャ2)/); 
```

下面的代码尝试提取 `ECMAScript 数字` 中的 `数字` 部分。通过 String 的 `match` 方法和捕获，提取了与数字 (`\d+`) 匹配的部分。

```
// "ECMAScript (数字+)"にマッチするが、欲しい文字列は数字の部分のみ
const pattern = /ECMAScript (\d+)/;
// 返り値は0 番目がマッチした全体、1 番目がキャプチャの1 番目というように対応している
// [マッチした全部の文字列, キャプチャの1 番目, キャプチャの2 番目 ....]
const [all, capture1] = "ECMAScript 6".match(pattern);
console.log(all); // => "ECMAScript 6"
console.log(capture1); // => "6" 
```

当使用正则表达式的 `g` 标志重复匹配字符串时，应该使用 `matchAll` 方法。正如前面介绍的那样，`match` 方法无法在重复匹配时获取各自独立的匹配信息。

下面的代码尝试提取 `ES 数字` 中的数字 (`\d+`)。通过迭代处理 `matchAll` 的返回值，可以提取每个匹配的捕获。

```
// "ES(数字+)"にマッチするが、欲しい文字列は数字の部分のみ
const pattern = /ES(\d+)/g;
// iteratorを返す
const matchesIterator = "ES2015、ES2016、ES2017".matchAll(pattern);
for (const match of matchesIterator) {
    // マッチした要素ごとの情報を含んでいる
    // 0 番目はマッチした文字列全体、1 番目がキャプチャの1 番目である数字
    console.log(`match: "${match[0]}", capture1: ${match[1]}, index: ${match.index}, input: "${match.input}"`);
}
// 次の順番でコンソールに出力される
// match: "ES2015", capture1: 2015, index: 0, input: "ES2015、ES2016、ES2017"
// match: "ES2016", capture1: 2016, index: 7, input: "ES2015、ES2016、ES2017"
// match: "ES2017", capture1: 2017, index: 14, input: "ES2015、ES2016、ES2017" 
```

#### *[专栏] 使用 RegExp.prototype.exec 进行 String.prototype.matchAll*

*String 的`matchAll`方法是在 ES2020 中引入的方法。 在那之前，我们使用类似于 String 的`match`方法的 RegExp 的`exec`方法来实现类似于 String 的`matchAll`方法的表达。

RegExp 的`exec`方法是接受字符串参数的方法。

```
/pattern/.exec("文字列"); 
```

当使用没有`g`标志的模式进行搜索时，RegExp 的`exec`方法将返回一个包含仅匹配的第一个结果的特殊数组。 在这种情况下，返回的数组具有添加了`index`属性和`input`属性的特殊数组，与 String 的`match`方法类似。

```
const str = "ABC あいう DE えお";
const alphabetsPattern = /[a-zA-Z]+/;
// gフラグなしでは、最初の結果のみを持つ配列を返す
const results = alphabetsPattern.exec(str);
console.log(results.length); // => 1
console.log(results[0]); // => "ABC"
// マッチした文字列の先頭のインデックス
console.log(results.index); // => 0
// 検索対象となった文字列全体
console.log(results.input); // => "ABC あいう DE えお" 
```

即使在使用具有`g`标志的模式进行搜索时，RegExp 的`exec`方法也将返回一个包含仅匹配的第一个结果的特殊数组。 这一点与 String 的`match`方法不同。 此外，它还将匹配的字符串末尾索引记录在正则表达式对象的`lastIndex`属性中。 然后再次调用`exec`方法时，将从最后一个匹配的末尾索引（`lastIndex`属性位置）开始搜索。

```
const str = "ABC あいう DE えお";
const alphabetsPattern = /[a-zA-Z]+/g;
// まだ一度も検索していないので、lastIndexは0となり先頭から検索が開始される
console.log(alphabetsPattern.lastIndex); // => 0
// gフラグありでも、一回目の結果は同じだが、`lastIndex`プロパティが更新される
const result1 = alphabetsPattern.exec(str);
console.log(result1[0]); // => "ABC"
console.log(alphabetsPattern.lastIndex); // => 3
// 2 回目の検索が、`lastIndex`の値のインデックスから開始される
const result2 = alphabetsPattern.exec(str);
console.log(result2[0]); // => "DE"
console.log(alphabetsPattern.lastIndex); // => 10
// 検索結果が見つからない場合はnullを返し、`lastIndex`プロパティは0にリセットされる
const result3 = alphabetsPattern.exec(str);
console.log(result3); // => null
console.log(alphabetsPattern.lastIndex); // => 0 
```

总结 RegExp 的`exec`方法的行为如下。 如果正则表达式没有`g`标志，则结果与 String 的`match`方法相同。 另一方面，如果正则表达式有`g`标志，则行为与 String 的`match`方法不同。

+   如果没有匹配，则返回`null`

+   如果匹配成功，则返回包含匹配字符串的特殊数组

+   当使用正则表达式的`g`标志时，返回包含匹配字符串的特殊数组，并将匹配的末尾索引记录在正则表达式对象的`lastIndex`属性中

利用正则表达式的`g`标志和`exec`方法进行搜索时，可以利用`lastIndex`属性在每次搜索时更新，从而获取所有匹配的结果。

下面的代码使用了 RegExp 的`exec`方法，将匹配字母的结果存储在`matches`中。 在具有`g`标志的情况下，`exec`方法会记录最后一个匹配位置，因此在`while`循环中进行迭代处理并从上次匹配的位置继续搜索。 此外，由于`exec`方法在没有匹配时会返回`null`，因此一旦没有匹配项，就会自动退出 while 循环。

```
const str = "ABC あいう DE えお";
const alphabetsPattern = /[a-zA-Z]+/g;
let matches;
while (matches = alphabetsPattern.exec(str)) {
    // RegExpの`exec`メソッドの返り値は`index`プロパティなどを含む特殊な配列
    console.log(`match: ${matches[0]}, index: ${matches.index}, lastIndex: ${alphabetsPattern.lastIndex}`);
}
// 次の順番でコンソールに出力される
// match: ABC, index: 0, lastIndex: 3
// match: DE, index: 8, lastIndex: 10 
```

使用 RegExp 的`exec`方法和正则表达式的`g`标志，实现了类似于 String 的`matchAll`方法的迭代处理。 RegExp 的`exec`方法是一种在引入迭代处理对象之前就存在的方法。

与 String 的`matchAll`方法处理 Iterator 的直观迭代相比，RegExp 的`exec`方法需要手动编写`while`循环等迭代处理，因此不够直观。 因此，在可以使用 String 的`matchAll`方法的情况下，就不需要使用 RegExp 的`exec`方法。

#### *获取布尔值*

*使用正则表达式对象，可以使用 RegExp 的`test`方法来测试是否匹配该模式。

正则表达式的模式中有指定位置的特殊字符。因此，“基于字符串的搜索”中提到的方法可以使用特殊字符和 RegExp 的`test`方法来表示。

+   String 的`startsWith`等效于：`/^pattern/.test(string)`

    +   `^`表示匹配开头的特殊字符

+   String 的`endsWith`等效于：`/pattern$/.test(string)`

    +   `$`表示匹配末尾的特殊字符

+   String 的`includes`等效于：`/pattern/.test(string)`

让我们看一个具体的例子。

```
// 検索対象となる文字列
const str = "にわにはにわにわとりがいる";
// ^ - 検索文字列が先頭ならtrue
console.log(/^にわ/.test(str)); // => true
console.log(/^いる/.test(str)); // => false
// $ - 検索文字列が末尾ならtrue
console.log(/にわ$/.test(str)); // => false
console.log(/いる$/.test(str)); // => true
// 検索文字列が含まれるならtrue
console.log(/にわ/.test(str)); // => true
console.log(/いる/.test(str)); // => true 
```

此外，正则表达式还可以使用重复和字符集等特殊字符来实现灵活的搜索。

### *使用字符串还是正则表达式*

*我们已经知道，正则表达式和 String 方法可以实现相同的搜索。当 String 方法和正则表达式可以获得相同结果时，应该选择哪一个呢？

正则表达式对模糊搜索有很强的支持，并且可以使用特殊字符获得灵活的搜索结果。另一方面，由于模糊性，有时很难从正则表达式的模式本身看出代码正在搜索什么。

下面的例子尝试判断从`/`开始到`/`结束的字符串。这个判断分别使用了正则表达式和 String 方法来实现（这是一个故意不利于正则表达式的例子）。

在正则表达式的情况下，即使看到`/^\/.*\/$/`这样的模式，也很难一眼看出想要做什么。而在 String 方法的情况下，可以直接将判断是否以`/`开头并以`/`结尾的内容表达为代码。

```
const str = "/正規表現のような文字列/";
// 正規表現で`/`からはじまり`/`で終わる文字列のパターン
const regExpLikePattern = /^\/.*\/$/;
// RegExpの`test`メソッドでパターンにマッチするかを判定
console.log(regExpLikePattern.test(str)); // => true
// Stringメソッドで、`/`からはじまり`/`で終わる文字列かを判定する関数
const isRegExpLikeString = (str) => {
    return str.startsWith("/") && str.endsWith("/");
};
console.log(isRegExpLikeString(str)); // => true 
```

正则表达式灵活且方便，但很容易在代码中失去意图。因此，在处理正则表达式时，最好使用注释或变量名来说明具体意图。

回到“使用 String 方法和正则表达式获得相同结果时应该选择哪个？”的疑问。建议在可以使用 String 方法表达的情况下使用 String 方法，而在需要灵活性或模糊搜索时，建议使用带有注释和变量名的正则表达式。

要了解更多关于正则表达式的信息，请参考[MDN 的正则表达式文档](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Regular_Expressions "正则表达式 - JavaScript | MDN")或者可以在控制台中尝试的[regex101](https://regex101.com/ "Online regex tester and debugger: PHP, PCRE, Python, Golang and JavaScript")等网站。

## *替换/删除字符串*

*要替换或删除字符串的一部分，可以使用 String 的`replace`方法。正如在“数据类型和字面量”中所述，作为不可变的原始类型，字符串无法删除部分字符。

换句话说，`delete`运算符不能用于字符串。在严格模式下，如果尝试删除无法删除的属性，将会导致错误。在非严格模式下，将会被忽略而不会报错（详细信息请参考“JavaScript 是什么”中的严格模式）。

```
"use strict";
const str = "文字列";
// 文字列の0 番目の削除を試みるがStrict modeでは例外が発生する
delete str[0]; // => TypeError: property 0 is non-configurable and can't be deleted 
```

相反，String 的`replace`方法可以通过返回删除所需字符的新字符串来表示删除。`replace`方法会将**字符串**中与第一个参数`搜索字符串`或正则表达式匹配的部分替换为第二个参数`替换字符串`。一个参数可以是字符串也可以是正则表达式。

```
文字列.replace("検索文字列", "置換文字列");
文字列.replace(/パターン/, "置換文字列"); 
```

如下所示，通过将要删除的部分替换为空字符串，可以删除字符串。

```
const str = "文字列";
// "文字"を""（空文字列）へ置換することで"削除"を表現
const newStr = str.replace("文字", "");
console.log(newStr); // => "列" 
```

`replace`方法也可以指定正则表达式。通过传递启用`g`标志的正则表达式，可以替换字符串中与模式匹配的所有内容。

```
// 検索対象となる文字列
const str = "にわにはにわにわとりがいる";
// 文字列を指定した場合は、最初に一致したものだけが置換される
console.log(str.replace("にわ", "niwa")); // => "niwaにはにわにわとりがいる"
// `g`フラグなし正規表現の場合は、最初に一致したものだけが置換される
console.log(str.replace(/にわ/, "niwa")); // => "niwaにはにわにわとりがいる"
// `g`フラグあり正規表現の場合は、繰り返し置換を行う
console.log(str.replace(/にわ/g, "niwa")); // => "niwaにはniwaniwaとりがいる" 
```

当需要将字符串中所有匹配搜索字符串的内容全部替换时，可以使用 ES2021 中新增的 String 的`replaceAll`方法。在`replace`方法中，只会替换第一个匹配的内容，而在`replaceAll`方法中，会替换所有匹配的内容。

与使用 String 的`replace`方法和带有`g`标志的正则表达式的区别在于，String 的`replaceAll`方法可以使用字符串而不是正则表达式来替换所有内容。因此，在`replaceAll`方法中，甚至可以直接将具有特殊含义的字符（如`?`）作为搜索字符串写入并进行替换。

```
// 検索対象となる文字列
const str = "???";
// replaceメソッドに文字列を指定した場合は、最初に一致したものだけが置換される
console.log(str.replace("?", "!")); // => "!??"
// replaceAllメソッドに文字列を指定した場合は、一致したものがすべて置換される
console.log(str.replaceAll("?", "!")); // => "!!!"
// replaceメソッドの場合は、正規表現の特殊文字はエスケープが必要となる
console.log(str.replace(/\?/g, "!")); // => "!!!"
// replaceAllメソッドにも正規表現を渡せるが、この場合はエスケープが必要となるためreplaceと同じ
console.log(str.replaceAll(/\?/g, "!")); // => "!!!" 
```

在`replace`方法和`replaceAll`方法中，可以使用捕获的字符串执行复杂的替换操作。

可以将回调函数传递给`replace`方法和`replaceAll`方法的第二个参数。与第一个参数`模式`匹配的部分将被回调函数的返回值替换。回调函数的第一个参数是整个匹配的字符串，后续参数是按顺序捕获的字符串。

```
const 置換した結果の文字列 = 文字列.replace(/(パターン)/, (all, ...captures) => {
    return 置換したい文字列;
}); 
```

举例来说，让我们写一个将`2017-03-01`替换为`2017 年 03 月 01 日`的过程。

正则表达式`/(\d{4})-(\d{2})-(\d{2})/g`将匹配字符串`"2017-03-01"`。`year`、`month`、`day`分别包含捕获的字符串，整个匹配的字符串将被回调函数的返回值替换。

```
function toDateJa(dateString) {
    // パターンにマッチしたときのみ、コールバック関数で置換処理が行われる
    return dateString.replace(/(\d{4})-(\d{2})-(\d{2})/g, (all, year, month, day) => {
        // `all`には、マッチした文字列全体が入っているが今回は利用しない
        // `all`が次の返す値で置換されるイメージ
        return `${year}年${month}月${day}日`;
    });
}
// マッチしない文字列の場合は、そのままの文字列が返る
console.log(toDateJa("本日ハ晴天ナリ")); // => "本日ハ晴天ナリ"
// マッチした場合は置換した結果を返す
console.log(toDateJa("今日は2017-03-01です")); // => "今日は2017 年 03 月 01 日です" 
```

## *构建字符串*

*最后让我们来看看如何构建字符串。正如本章开头所述，本章的目的是“学会自由地构建字符串”。

通过简单地连接或替换字符串，我们可以创建新的字符串。然而，在处理结构化字符串时，仅仅简单地连接可能会导致意义不同。

在这里，所说的结构化字符串是指字符串中包含有上下文的字符串，例如 URL 字符串具有以下结构，其中每个元素进入的字符串类型等都有规定（详细信息请参考「[URL 标准](https://url.spec.whatwg.org/ "URL Standard")」）。

```
"https://example.com/index.html"
 ^^^^^   ^^^^^^^^^^^ ^^^^^^^^^^
   |          |     　　　|
 scheme      host     pathname 
```

在创建这些字符串时，与使用简单的字符串连接运算符（`+`）相比，使用专门的函数会更安全。

例如，假设存在一个`getResource`函数，该函数通过传递`baseURL`和`pathname`来获取连接后的 URL 中的资源。这个`getResource`函数接受基础 URL(`baseURL`)和路径（`pathname`）作为参数。

```
// `baseURL`と`pathname`にあるリソースを取得する
function getResource(baseURL, pathname) {
    const url = baseURL + pathname;
    console.log(url); // => "http://example.com/resouces/example.js"
    // 省略) リソースを取得する処理...
}
const baseURL = "http://example.com/resouces";
const pathname = "/example.js";
getResource(baseURL, pathname); 
```

但是，有些人可能会认为`baseURL`的末尾应该包含`/`。`getResource`函数没有考虑到`baseURL`末尾包含`/`的情况。因此，可能会出现从意外的 URL 获取资源的问题。

```
// `baseURL`と`pathname`にあるリソースを取得する
function getResource(baseURL, pathname) {
    const url = baseURL + pathname;
    // `/` と `/` が２つ重なってしまっている
    console.log(url); // => "http://example.com/resouces//example.js"
    // 省略) リソースを取得する処理...
}
const baseURL = "http://example.com/resouces/";
const pathname = "/example.js";
getResource(baseURL, pathname); 
```

这个问题的难点在于，连接生成的`url`在字符串层面上是正确的，因此不会产生错误。也就是说，表面上看起来没有问题，但只有在实际运行时才会发现的问题。

因此，在处理此类结构化字符串时，通过创建专门的函数或对象来安全地处理字符串。

为了安全地执行类似之前的 URL 文字串连接，需要创建一个能够吸收输入的`baseURL`文字串表示差异的机制。接下来的`baseJoin`函数会返回基础 URL 和路径连接后的字符串，同时吸收基础 URL 末尾`/`的差异。

```
// ベースURLとパスを結合した文字列を返す
function baseJoin(baseURL, pathname) {
    // 末尾に / がある場合は、/ を削除してから結合する
    const stripSlashBaseURL = baseURL.replace(/\/$/, "");
    return stripSlashBaseURL + pathname;
}
// `baseURL`と`pathname`にあるリソースを取得する
function getResource(baseURL, pathname) {
    const url = baseJoin(baseURL, pathname);
    // baseURLの末尾に / があってもなくても同じ結果となる
    console.log(url); // => "http://example.com/resouces/example.js"
    // 省略) リソースを取得する処理...
}
const baseURL = "http://example.com/resouces/";
const pathname = "/example.js";
getResource(baseURL, pathname); 
```

虽然 ECMAScript 范围之外，但对于 URL 或文件路径等典型情况，已经存在专用的处理方式。例如，处理 URL 的 Web 标准 API 对象[URL](https://developer.mozilla.org/ja/docs/Web/API/URL "URL - Web API インターフェイス | MDN")、处理文件路径的 Node.js 核心模块[Path](https://nodejs.org/api/path.html "Path | Node.js v7.9.0 Documentation")等。如果存在专用机制，应避免直接使用`+`运算符进行字符串连接。

### *[ES2015] 标签模板函数*

*JavaScript 中，对于模板字符串，可以通过标签模板函数来执行只更改一部分的处理。标签模板函数是函数和模板字面量结合的表达式。请注意，在调用函数时使用的是`関数(`模板`)`而不是`関数`模板`関数`的形式。

通常作为普通函数调用时，函数的参数只是简单的字符串。

```
function tag(str) {
    // 引数`str`にはただの文字列が渡ってくる
    console.log(str); // => "template 0 literal 1"
}
// ()をつけて関数を呼び出す
tag(`template ${0} literal ${1}`); 
```

但是，通过使用`関数`模板`関数`而不是`()`来调用函数，`関数`接收的参数是针对标签模板的值。这时，函数的第一个参数是模板中`${}`分隔的字符串数组，后续参数是按顺序传递的`${}`中表达式的计算结果。

```
// 呼び出し方によって受け取る引数の形式が変わる
function tag(strings, ...values) {
    // stringsは文字列のパーツが${}で区切られた配列となる
    console.log(strings); // => ["template "," literal ",""]
    // valuesには${}の評価値が順番に入る
    console.log(values); // => [0, 1]
}
// ()をつけずにテンプレートを呼び出す
tag`template ${0} literal ${1}`; 
```

这两个都是相同的函数，但是以`関数`模板`関数`的形式调用时，传递的参数形式是特殊的。因此，将用于标签模板的函数称为**标签函数**（Tag function）。

首先，为了了解如何处理参数，实现了一个名为`stringRaw`的标签函数，该函数将标签模板的内容直接连接并返回。使用 Array 的`reduce`方法可以按顺序连接模板字符串和变量（关于`reduce`方法，请参考“数组”章节）。

```
// テンプレートを順番どおりに結合した文字列を返すタグ関数
function stringRaw(strings, ...values) {
    // 配列から文字列を返すためにreduceメソッドを利用する
    // resultの初期値はstrings[0]の値となる
    return strings.reduce((result, str, i) => {
        console.log([result, values[i - 1], str]);
        // それぞれループで次のような出力となる
        // 1 度目: ["template ", 0, " literal "]
        // 2 度目: ["template 0 literal ", 1, ""]
        return result + values[i - 1] + str;
    });
}
// 関数`テンプレートリテラル` という形で呼び出す
console.log(stringRaw`template ${0} literal ${1}`); // => "template 0 literal 1" 
```

这里实现的`stringRaw`标签函数与`String.raw`方法([ES2015])类似。

```
console.log(String.raw`template ${0} literal ${1}`); // => "template 0 literal 1" 
```

通过使用标签模板函数，可以对模板字符串进行特定形式的转换并嵌入数据，从而执行模板处理。

在下面的代码中，定义了一个将模板中的变量 URL 编码后嵌入的标签模板函数。`encodeURIComponent`函数是用于 URL 编码的函数。`escapeURL`函数则是使用`encodeURIComponent`函数对传入的变量进行 URL 编码后嵌入。

```
// 変数をURLエスケープするタグ関数
function escapeURL(strings, ...values) {
    return strings.reduce((result, str, i) => {
        return result + encodeURIComponent(values[i - 1]) + str;
    });
}

const input = "A&B";
// escapeURLタグ関数を使ったタグつきテンプレート
const escapedURL = escapeURL`https://example.com/search?q=${input}&sort=desc`;
console.log(escapedURL); // => "https://example.com/search?q=A%26B&sort=desc" 
```

通过使用标签模板字面量，可以添加与上下文相关的处理。这个功能在 JavaScript 内嵌入 HTML 等其他语言或 DSL（领域特定语言）时经常被使用。

## *终わりに*

*本章介绍了 JavaScript 中的字符串（`String`对象）。用于处理字符串的 String 方法有很多，其中也包括与正则表达式结合使用的方法。

正则表达式是 JavaScript 语言内的一种独立语言，几乎可以用它来编写一本书。有关详细信息，请参考[MDN 的正则表达式文档](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Regular_Expressions "正規表現 - JavaScript | MDN")等。

文字列看起来可能很简单，但也有一些包含 URL 或路径等上下文信息的。为了安全地处理这些字符串，需要根据上下文进行相应的处理。此外，通过使用带标签的模板字面量，可以实现自动转义模板中的变量等功能。

> ¹. 这是从[Unicode 片假名列表](https://unicode-table.com/jp/#katakana)中提取的表格。 ↩*****************************
