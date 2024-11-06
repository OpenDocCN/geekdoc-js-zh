# 条件分岐

> 原文：[`jsprimer.net/basic/condition/`](https://jsprimer.net/basic/condition/)

この章ではif 文やswitch 文を使った条件分岐について学んでいきます。 条件分岐を使うことで、特定の条件を満たすかどうかで行う処理を変更できます。

## [](#if-statement)*if 文*

*if 文を使うことで、プログラム内に条件分岐を書けます。

if 文は次のような構文が基本形となります。 `条件式`の評価結果が`true`であるならば、`実行する文`が実行されます。

```
if (条件式) {
    実行する文;
} 
```

次のコードでは`条件式`が`true`であるため、ifの中身が実行されます。

```
if (true) {
    console.log("この行は実行されます");
} 
```

`実行する文`が1つのみの場合は、`{` と `}` のブロックを省略できます。 しかし、どこまでがif 文かがわかりにくくなるため、常にブロックで囲むことを推奨します。

```
if (true)
    console.log("この行は実行されます"); 
```

if 文は`条件式`に比較演算子などを使い、その比較結果によって処理を分岐するためによく使われます。 次のコードでは、`x`が`10`よりも大きな値である場合に、if 文の中身が実行されます。

```
const x = 42;
if (x > 10) {
    console.log("xは10より大きな値です");
} 
```

if 文の`条件式`には`true`または`false`といった真偽値以外の値も指定できます。 真偽値以外の値の場合、その値を暗黙的に真偽値へ変換してから、条件式として判定します。

真偽値へ変換すると`true`となる値の種類は多いため、逆に変換した結果が`false`となる値を覚えるのが簡単です。 次の値は真偽値へと変換すると`false`となるため、これらは**falsy**な値と呼ばれます（「暗黙的な型変換」の章を参照）。

+   `false`

+   `undefined`

+   `null`

+   `0`

+   `0n`

+   `NaN`

+   `""`（空文字列）

falsyではない値は、`true`へと変換されます。 そのため、`"文字列"`や0 以外の数値などを`条件式`に指定した場合は、`true`へと変換してから条件式として判定します。

次のコードは、条件式が`true`へと変換されるため、if 文の中身が実行されます。

```
if (true) {
    console.log("この行は実行されます");
}
if ("文字列") {
    console.log("この行は実行されます");
}
if (42) {
    console.log("この行は実行されます");
}
if (["配列"]) {
    console.log("この行は実行されます");
}
if ({ name: "オブジェクト" }) {
    console.log("この行は実行されます");
} 
```

falsyな値を`条件式`に指定した場合は、`false`へと変換されます。 次のコードは、条件式が`false`へと変換されるため、if 文の中身は実行されません。

```
if (false) {
    // この行は実行されません
}
if ("") {
    // この行は実行されません
}
if (0) {
    // この行は実行されません
}
if (undefined) {
    // この行は実行されません
}
if (null) {
    // この行は実行されません
} 
```

### [](#else-if-statement)*else if 文*

*複数の条件分岐を書く場合は、if 文に続けてelse if 文を使うことで書けます。 たとえば、次の3つの条件分岐するプログラムを考えます。

+   `version` が `"ES5"` ならば `"ECMAScript 5"` と出力

+   `version` が `"ES6"` ならば `"ECMAScript 2015"` と出力

+   `version` が `"ES7"` ならば `"ECMAScript 2016"` と出力

次のコードでは、if 文とelse if 文を使うことで3つの条件を書いています。 変数`version`の値が`"ES6"`であるため、コンソールには`"ECMAScript 2015"`が出力されます。

```
const version = "ES6";
if (version === "ES5") {
    console.log("ECMAScript 5");
} else if (version === "ES6") {
    console.log("ECMAScript 2015");
} else if (version === "ES7") {
    console.log("ECMAScript 2016");
} 
```

### [](#else-statement)*else 文*

*if 文とelse if 文では、条件に一致した場合の処理をブロック内に書いていました。 一方で条件に一致しなかった場合の処理は、else 文を使うことで書けます。

次のコードでは、変数`num`の数値が10より大きいかを判定しています。 `num`の値は10 以下であるため、else 文で書いた処理が実行されます。

```
const num = 1;
if (num > 10) {
    console.log(`numは10より大きいです: ${num}`);
} else {
    console.log(`numは10 以下です: ${num}`);
} 
```

#### [](#nested-if-statement)*ネストしたif 文*

*if 文、else if 文、else 文はネストして書けます。 次のように複数の条件を満たすかどうかをif 文のネストとして表現できます。

```
if (条件式 A) {
    if (条件式 B) {
        // 条件式 Aと条件式 Bがtrueならば実行される文
    }
} 
```

ネストしたif 文���例として、今年がうるう年かを判定してみましょう。

うるう年の条件は次のとおりです。

+   西暦で示した年が4で割り切れる年はうるう年です

+   ただし、西暦で示した年が100で割り切れる年はうるう年ではありません

+   ただし、西暦で示した年が400で割り切れる年はうるう年です

西暦での現在の年は `new Date().getFullYear();` で取得できます。 このうるう年の条件をif 文で表現すると次のように書けます。

```
const year = new Date().getFullYear();
if (year % 4 === 0) { // 4で割り切れる
    if (year % 100 === 0) { // 100で割り切れる
        if (year % 400 === 0) { // 400で割り切れる
            console.log(`${year}年はうるう年です`);
        } else {
            console.log(`${year}年はうるう年ではありません`);
        }
    } else {
        console.log(`${year}年はうるう年です`);
    }
} else {
    console.log(`${year}年はうるう年ではありません`);
} 
```

条件を上から順に書き下したため、ネストが深い文となってしまっています。 一般的にはネストは浅いほうが、読みやすいコードとなります。

条件を少し読み解くと、400で割り切れる年は無条件にうるう年であることがわかります。 そのため、条件を並び替えることで、ネストするif 文なしに書くことができます。

```
const year = new Date().getFullYear();
if (year % 400 === 0) { // 400で割り切れる
    console.log(`${year}年はうるう年です`);
} else if (year % 100 === 0) { // 100で割り切れる
    console.log(`${year}年はうるう年ではありません`);
} else if (year % 4 === 0) { // 4で割り切れる
    console.log(`${year}年はうるう年です`);
} else { // それ以外
    console.log(`${year}年はうるう年ではありません`);
} 
```

## [](#switch-statement)*switch 文*

*switch 文は、次のような構文で`式`の評価結果が指定した値である場合に行う処理を並べて書きます。

```
switch (式) {
    case ラベル1:
        // `式`の評価結果が`ラベル1`と一致する場合に実行する文
        break;
    case ラベル2:
        // `式`の評価結果が`ラベル2`と一致する場合に実行する文
        break;
    default:
        // どのcaseにも該当しない場合の処理
        break;
}
// break; 後はここから実行される 
```

switch 文はif 文と同様に`式`の評価結果に基づく条件分岐を扱います。 またbreak 文は、switch 文から抜けてswitch 文の次の文から実行するためのものです。 次のコードでは、`version`の評価結果は`"ES6"`となるため、`case "ES6":`に続く文が実行されます。

```
const version = "ES6";
switch (version) {
    case "ES5":
        console.log("ECMAScript 5");
        break;
    case "ES6":
        console.log("ECMAScript 2015");
        break;
    case "ES7":
        console.log("ECMAScript 2016");
        break;
    default:
        console.log("しらないバージョンです");
        break;
}
// "ECMAScript 2015" と出力される 
```

これはif 文で次のように書いた場合と同じ結果になります。

```
const version = "ES6";
if (version === "ES5") {
    console.log("ECMAScript 5");
} else if (version === "ES6") {
    console.log("ECMAScript 2015");
} else if (version === "ES7") {
    console.log("ECMAScript 2016");
} else {
    console.log("しらないバージョンです");
} 
```

switch 文はやや複雑な仕組みであるため、どのように処理されているかを見ていきます。 まず `switch (式)` の`式`を評価します。

```
switch (式) {
    // case
} 
```

次に`式`の評価結果が厳密等価演算子（`===`）で一致するラベルを探索します。 一致するラベルが存在する場合は、そのcase 節を実行します。 一致する`ラベル`が存在しない場合は、default 節が実行されます。

```
switch (式) {
    // if (式 === "ラベル1")
    case "ラベル1":
        break;
    // else if (式 === "ラベル2")
    case "ラベル2":
        break;
    // else
    default:
        break;
} 
```

### [](#break-statement)*break 文*

*switch 文のcase 節では基本的に`break;`を使ってswitch 文を抜けるようにします。 この`break;`は省略が可能ですが、省略した場合、後ろに続くcase 節が条件に関係なく実行されます。

```
const version = "ES6";
switch (version) {
    case "ES5":
        console.log("ECMAScript 5");
    case "ES6": // 一致するケース
        console.log("ECMAScript 2015");
    case "ES7": // breakされないため条件無視して実行
        console.log("ECMAScript 2016");
    default:    // breakされないため条件無視して実行
        console.log("しらないバージョンです");
}
/*
 "ECMAScript 2015"
 "ECMAScript 2016"
 "しらないバージョンです"
 と出力される
 */ 
```

このように`break;`を忘れてしまうと意図しないcase 節が実行されてしまいます。 そのため、case 節とbreak 文が多用されているswitch 文が出てきた場合、 別の方法で書けないかを考えるべきサインとなります。

switch 文はif 文の代用として使うのではなく、次のように関数と組み合わせて条件に対する値を返すパターンとして使うことが多いです。 関数については「関数と宣言」の章を参照してください。

```
function getECMAScriptName(version) {
    switch (version) {
        case "ES5":
            return "ECMAScript 5";
        case "ES6":
            return "ECMAScript 2015";
        case "ES7":
            return "ECMAScript 2016";
        default:
            return "しらないバージョンです";
    }
}
// 関数を実行して`return`された値を得る
getECMAScriptName("ES6"); // => "ECMAScript 2015" 
```

## [](#conclusion)*まとめ*

*この章では条件分岐について学びました。

+   if 文、else if 文、else 文で条件分岐した処理を扱える

+   条件式に指定した値は真偽値へと変換してから判定される

+   真偽値に変換すると`false`となる値をfalsyと呼ぶ

+   switch 文とcase 節、default 節を組み合わせて条件分岐した処理を扱える

+   case 節でbreak 文を省略した場合は、後ろに続くcase 節が実行される。

条件分岐にはif 文やswitch 文を利用します。複雑な条件を定義する場合には、if 文のネストが深くなりやすいです。そのような場合には、条件式自体を見直してよりシンプルな条件にできないかを考えてみることも重要です。
