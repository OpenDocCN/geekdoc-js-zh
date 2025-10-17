# 数据类型与字面量

> 原文：[`jsprimer.net/basic/data-type/`](https://jsprimer.net/basic/data-type/)

## [](#data-type)*数据类型*

*JavaScript 被归类为动态类型语言，因此没有像静态类型语言那样的**变量类型**。然而，存在诸如字符串、数值、真值等**值类型**。这些值类型被称为**数据类型**。

数据类型大致分为**原始类型**和**对象**两种。

原始类型（基本类型）是指诸如真值或数值等基本值的类型。原始类型的值一旦创建就不能更改，具有不可变（immutable）的特性。在 JavaScript 中，字符串也具有不可变的特性，一旦创建就不能更改，被视为原始类型之一。

另一方面，不是原始类型的被称为对象（复合类型），对象是由多个原始类型的值或对象组成的集合。对象在创建后可以改变其值，因此具有可变（mutable）的特性。由于对象是通过引用值来操作的，因此也被称为引用类型的数据。

数据类型细分后，由 7 个原始类型和对象组成。

+   原始类型（基本类型）

    +   真值（Boolean）: `true` 或 `false` 的数据类型

    +   数值（Number）: `42` 或 `3.14159` 等数值的数据类型

    +   大整数（BigInt）: 从 ES2020 开始添加的 `9007199254740992n` 等任意精度的整数的数据类型

    +   字符串（String）: `"JavaScript"` 等字符串的数据类型

    +   undefined：表示值未定义的数据类型

    +   null：表示值不存在的数据类型

    +   符号（Symbol）: 从 ES2015 开始添加的唯一且不可变的值的数据类型

+   对象（复合类型）

    +   非原始类型的数据

    +   对象、数组、函数、类、正则表达式、Date 等

非原始类型的是对象，记住这一点就不会有问题。

使用 `typeof` 操作符可以像这样检查数据类型。

```
console.log(typeof true);// => "boolean"
console.log(typeof 42); // => "number"
console.log(typeof 9007199254740992n); // => "bigint"
console.log(typeof "JavaScript"); // => "string"
console.log(typeof Symbol("シンボル"));// => "symbol"
console.log(typeof undefined); // => "undefined"
console.log(typeof null); // => "object"
console.log(typeof ["配列"]); // => "object"
console.log(typeof { "key": "value" }); // => "object"
console.log(typeof function() {}); // => "function" 
```

原始类型的值分别通过 `typeof` 操作符的评估结果返回其值的类型。另一方面，被归类为对象的值是 `"object"`。

数组（`[]`）和对象（`{}`）都返回 `"object"` 这个判定结果。因此，`typeof` 操作符不能正确地判定对象的详细类型。但是，函数在对象中是特殊处理的，因此 `typeof` 操作符的评估结果是 `"function"`。另外，`typeof null` 返回 `"object"` 是由于历史原因的规范错误¹。

从这个例子中可以看出，`typeof` 操作符用于区分原始类型或对象。记住，`typeof` 操作符不能确定对象的详细类型。关于每个对象的判定方法，将在各自的章节中介绍。

## [](#literal)*字面量*

*原始类型的值或一些对象可以通过**字面量**轻松定义。

字面量是在程序中直接以语法形式定义的数值或字符串等数据类型值。例如，用引号 `"` 和 `"` 包围的范围是字符串字面量，它表示字符串类型的值。

在下面的代码中，定义了一个名为 `str` 的变量，其初始值为具有字符串类型的 `"こんにちは"` 数据。

```
// "と"で囲んだ範囲が文字列リテラル
const str = "こんにちは"; 
```

如果没有字面量表示，则通过将值作为参数传递给创建该值的函数来创建该值。为了避免这种冗长的表示，为常用的主要数据类型提供了字面量。

下面的 5 个原始类型各自都有字面量表示。

+   真值

+   数值

+   BigInt

+   字符串

+   null

此外，对于在对象中经常使用的，也提供了字面量表示。

+   对象

+   数组

+   正则表达式

关于这些字面量，首先从原始类型开始依次介绍。

### [](#boolean)*真值（Boolean）*

*真值有 `true` 和 `false` 的字面量。它们分别返回 `true` 和 `false` 的值，具有直观的含义。

```
true; // => true
false; // => false 
```

### [](#number)*数值（Number）*

*数值包括像 `42` 这样的整数字面量以及像 `3.14159` 这样的浮点数字面量。

这些字面量表示的数值被视为 IEEE 754 的双精度浮点数。双精度浮点数使用 64 位来表示数值。在 64 位中，52 位用于存储数字，11 位用于小数点的位置，剩下的 1 位是正负符号。因此，可以精确处理的数值的最大值是 `2⁵³-1`（2 的 53 次方减 1）。

#### [](#integer-literal)*整数字面量*

*整数字面量有以下 4 种。

+   十进制：数字的组合

    +   但是，当组合多个数字时，如果以 `0` 开头，则可能被视为八进制。

    +   例）`0`、`2`、`10`

+   二进制：紧跟在 `0b`（或 `0B`）之后的是由 `0` 或 `1` 组成的数字组合

    +   例）`0b0`、`0b10`、`0b1010`

+   八进制：紧跟在 `0o`（或 `0O`）之后的是由 `0` 到 `7` 组成的数字组合

    +   `0o` 是数字的零和小写字母 `o`

    +   例）`0o644`、`0o777`

+   十六进制：紧跟在 `0x`（或 `0X`）之后的是由 `0` 到 `9` 的数字和 `a` 到 `f` 或 `A` 到 `F` 的字母组成的组合

    +   字母的大小写对值没有影响

    +   例）`0x30A2`、`0xEEFF`

仅由 0 到 9 的数字组成的数值被视为十进制。

```
console.log(1); // => 1
console.log(10); // => 10
console.log(255); // => 255 
```

以 `0b` 开头的二进制数字面量常用于表示位。`b` 代表二进制（binary）。

```
console.log(0b1111); // => 15
console.log(0b10000000000); // => 1024 
```

以 `0o` 开头的八进制数字面量常用于表示文件权限。`o` 代表八进制（octal）。

```
console.log(0o644);  // => 420
console.log(0o777);  // => 511 
```

次のように、`0`からはじまり、`0`から`7`の数字を組み合わせた場合も8 進数として扱われます。 しかし、この表記は10 進数と紛らわしいものであったため、ES2015で`0o`の8 進数リテラルが新たに導入されました。 また、strict modeではこの書き方は例外が発生するため、次のような8 進数の書き方は避けるべきです（詳細は「JavaScriptとは」のstrict modeを参照）。

```
// 非推奨な8 進数の書き方
// strict modeは例外が発生
console.log(0644);  // => 420
console.log(0777);  // => 511 
```

`0x`からはじまる16 進数リテラルは、文字のコードポイントやRGB 値の表現などに利用されています。 `x`は16 進数を表すhexを意味しています。

```
console.log(0xFF); // => 255
// 小文字で書いても意味は同じ
console.log(0xff); // => 255
console.log(0x30A2); // => 12450 
```

| 名前 | 表記例 | 用途 |
| --- | --- | --- |
| 10 進数 | 42 | 数値 |
| 2 進数 | 0b0001 | ビット演算など |
| 8 進数 | 0o777 | ファイルのパーミッションなど |
| 16 進数 | 0xEEFF | 文字のコードポイント、RGB 値など |

#### [](#floating-point-number-literal)*浮動小数点数リテラル*

*When writing a floating-point number as a literal, the following two types of notation can be used.

+   `3.14159` のような `.`（ドット）を含んだ数値

+   `2e8`のような`e`または`E`を含んだ数値

`0`からはじまる浮動小数点数は、`0`を省略して書くことができます。

```
.123; // => 0.123 
```

しかし、JavaScriptでは`.`をオブジェクトにおいて利用する機会が多いため、 `0`からはじまる場合でも省略せずに書いたほうが意図しない挙動を減らせるでしょう。

> **Note** 変数名を数字からはじめることができないのは、数値リテラルと衝突してしまうからです。

`e`は指数（exponent）を意味する記号で、`e`のあとには指数部の値を書きます。 たとえば、`2e8`は2×10の8 乗となるので、10 進数で表すと`200000000`となります。

```
2e8; // => 200000000 
```

### [](#bigint-literal)*[ES2020] BigInt*

*JavaScriptでは、`1`や`3.14159`などの数値リテラルは[IEEE 754](https://ja.wikipedia.org/wiki/IEEE_754)で定義された倍精度浮動小数となります。 倍精度浮動小数で正確に扱える数値の最大値は`2⁵³-1`（2の53 乗から1 引いた値である`9007199254740991`）です。 この数値リテラルで安全に表せる最大の数値は`Number.MAX_SAFE_INTEGER`として定義されています。

```
console.log(Number.MAX_SAFE_INTEGER); // => 9007199254740991 
```

When expressing or calculating a value larger than `2⁵³-1` (`9007199254740991`) with a number literal, there may be cases where incorrect results occur.

この問題を解決するために、ES2020では`BigInt`という新しい整数型のデータ型とリテラルが追加されました。 数値リテラルは倍精度浮動小数（64ビット）で数値を扱うのに対して、BigIntでは任意の精度の整数を扱えます。 そのため、BigIntでは`2⁵³-1`（`9007199254740991`）よりも大きな整数を正しく表現できます。

BigIntリテラルは、数値の後ろに`n`をつけます。

```
console.log(1n); // => 1n
// 2⁵³-1より大きな値も扱える
console.log(9007199254740992n); // => 9007199254740992n 
```

BigInt is a data type that handles integers, so if it contains a decimal point, a syntax error occurs.

```
1.2n; // => SyntaxError 
```

### [](#numeric-separators)*[ES2021] Numeric Separators*

*The more the number becomes large, the more likely it is to make mistakes in the number of digits. The following code is to write 1 billion as a number literal, but it is difficult to read the number of digits.

```
1000000000000; 
```

ES2021から、数値リテラル内の区切り文字として`_`を追加できるNumeric Separatorsがサポートされています。 Numeric Separatorsは、数値リテラル内では区切り文字として`_`が追加できます。 次のコードも、1 億を数値リテラルで書いています。数値リテラルを評価する際に`_`は単純に無視されるため同じ意味となります。

```
1_000_000_000_000; 
```

Numeric Separators can only be used in the literals of integers, floating-point numbers, and BigInt. Additionally, `_` cannot be added at the beginning or end of the literal.

```
_123; // 変数として評価される
3._14; // => SyntaxError
0x52_; // => SyntaxError
1234n_; // => SyntaxError 
```

### [](#string)*文字列（String）*

*As a common rule for string literals, the content enclosed with the same symbol is treated as a string. There are three types of literals as string literals, but their evaluation results are all the same `"string"`.

```
console.log("文字列"); // => "文字列"
console.log('文字列'); // => "文字列"
console.log(`文字列`); // => "文字列" 
```

#### [](#double-quote-and-single-quote)*ダブルクォートとシングルクォート*

*`"`（ダブルクォート）と`'`（シングルクォート）はまったく同じ意味となります。 PHPやRubyなどとは違い、どちらのリテラルでも評価結果は同じとなります。

文字列リテラルは同じ記号で囲む必要があるため、次のように文字列の中に同じ記号が出現した場合は、 `\'`のように`\`（バックスラッシュ）を使ってエスケープしなければなりません。

```
'8 o\'clock'; // => "8 o'clock" 
```

また、文字列内部に出現しない別のクォート記号を使うことで、エスケープをせずに書くこともできます。

```
"8 o'clock"; // => "8 o'clock" 
```

ダブルクォートとシングルクォートどちらも、改行をそのままでは入力できません。 次のように改行を含んだ文字列は定義できないため、構文エラー（`SyntaxError`）となります。

```
"複数行の
文字列を
入れたい"; // => SyntaxError: "" string literal contains an unescaped line break 
```

改行の代わりに改行記号のエスケープシーケンス（`\n`）を使うことで複数行の文字列を書くことができます。

```
"複数行の\n 文字列を\n 入れたい"; 
```

シングルクォートとダブルクォートの文字列リテラルに改行を入れるには、エスケープシーケンスを使わないといけません。 これに対してES2015から導入されたテンプレートリテラルでは、複数行の文字列を直感的に書くことができます。

#### [](#template-literal)*[ES2015] Template Literals*

*テンプレートリテラルは、```（バッククォート）で囲んだ範囲を文字列とするリテラルです。 テンプレートリテラルでは、複数行の文字列を改行記号のエスケープシーケンス（`\n`）を使わずにそのまま書くことができます。

複数行の文字列も```で囲めば、そのまま書くことができます。

```
`複数行の
文字列を
入れたい`; // => "複数行の\n 文字列を\n 入れたい" 
```

また、名前のとおりテンプレートのような機能も持っています。 テンプレートリテラル内で`${変数名}`と書いた場合に、その変数の値を埋め込むことができます。

```
const str = "文字列";
console.log(`これは${str}です`); // => "これは文字列です" 
```

テンプレートリテラルも他の文字列リテラルと同様に同じリテラル記号を内包したい場合は、`\`を使ってエスケープする必要があります。

```
`This is \`code\``;// => "This is `code`" 
```

### [](#null-literal)*nullリテラル*

*nullリテラルは`null`値を返すリテラルです。 `null`は「値がない」ということを表現する値です。

次のように、未定義の変数を参照した場合は、参照できないため`ReferenceError`の例外が投げられます。

```
foo;// "ReferenceError: foo is not defined" 
```

如果想表示 `foo` has no value, you can define a `foo` variable with a `null` value by assignment. In this way, you can define `foo` as a variable without a value and reference it.

```
const foo = null;
console.log(foo); // => null 
```

## [](#undefined-is-not-literal)*[专栏] undefined is not a literal* 

*作为原始类型介绍的`undefined`并不是字面量。 `undefined`只是一个全局变量，它只有一个名为`undefined`的值。

可以声明具有相同名称的局部变量，因为`undefined`只是一个全局变量。

```
function fn(){
    const undefined = "独自の未定義値"; // undefinedという名前の変数をエラーなく定義できる
    console.log(undefined); // => "独自の未定義値"
}
fn(); 
```

相比之下，像`true`、`false`、`null`等则不是全局变量，而是字面量，因此无法定义具有相同名称的变量。 字面量类似于保留字，因此如果尝试重新定义它们，将会产生语法错误（SyntaxError）。

```
let null; // => SyntaxError 
```

在这里，我们声明了名为`undefined`的局部变量用于解释，但是不推荐重新定义`undefined`。 因为这样只会带来不必要的混乱，所以应该避免。

### [](#object)*对象字面量*

*在 JavaScript 中，对象是一切的基础。 创建这些对象的方法之一是使用对象字面量。 通过编写`{}`（大括号），可以创建新对象。

```
const obj = {}; // 中身が空のオブジェクトを作成 
```

对象字面量允许在创建对象的同时定义其内容。 使用`{}`内的键和值以`:`分隔，即可创建并初始化对象。

下面的代码创建了一个具有`key`键和值为`"value"`字符串的对象。 键名可以是字符串或 Symbol，值可以是从原始类型到对象的任何内容。

```
const obj = {
    "key": "value"
}; 
```

在这种情况下，对象具有的键称为属性名。 在这种情况下，`obj`对象具有一个名为`key`的属性。

要引用`obj`的`key`属性，可以使用`.`（点）连接或`[]`（方括号）引用的方式。

```
const obj = {
    "key": "value"
};
// ドット記法
console.log(obj.key); // => "value"
// ブラケット記法
console.log(obj["key"]); // => "value" 
```

在点表示法中，属性名必须是标识符。 因此，无法使用类似下面的无法作为标识符的属性名来使用点表示法。

```
// プロパティ名は文字列の"123"
const object = {
    "123": "value"
};
// OK: ブラケット記法では、文字列として書くことができる
console.log(object["123"]); // => "value"
// NG: ドット記法では、数値からはじまる識別子は利用できない
object.123 
```

对象非常重要，因为接下来将介绍的数组和正则表达式都是基于此对象。 详细信息请参见“对象”章节。 在这里，当出现对象字面量（`{`和`}`）时，请理解为正在创建新对象。

### [](#array)*数组字面量*

*除了对象字面量之外，数组字面量也是一种常用的字面量。 数组字面量使用`[`和`]`将值括起来，用逗号分隔，创建一个具有这些值的 Array 对象。 数组（Array 对象）是一种可以存储多个值并且具有顺序的对象。

```
const emptyArray = []; // 空の配列を作成
const array = [1, 2, 3]; // 値を持った配列を作成 
```

数组从`0`开始的索引（下标）保持与相应值的关系。 要获取创建的数组的元素，请使用`array[index]`语法指定的索引。

```
const array = ["index:0", "index:1", "index:2"];
// 0 番目の要素を参照
console.log(array[0]); // => "index:0"
// 1 番目の要素を参照
console.log(array[1]); // => "index:1" 
```

关于数组的详细信息将在“数组”章节中进行解释。

### [](#regexp-literal)*正则表达式字面量*

*JavaScript 允许使用字面量编写正则表达式。 正则表达式字面量使用`/`（斜杠）将正则表达式模式字符串括起来。 在正则表达式模式中，特定字符和以`\`（反斜杠）开头的特殊字符具有特殊含义。

在下面的代码中，我们使用匹配数字的特殊字符`\d`，并且使用正则表达式字面量表示匹配一个或多个数字的正则表达式。

```
const numberRegExp = /\d+/; // 1 文字以上の数字にマッチする正規表現
// `numberRegExp`の正規表現が文字列"123"にマッチするかをテストする
console.log(numberRegExp.test("123")); // => true 
```

通过使用`RegExp`构造函数，可以从字符串创建正则表达式对象。 但是，这样做会导致特殊字符的双重转义，使得编写起来不够直观。

关于正则表达式对象的详细信息将在“字符串”章节中介绍。

## [](#primitive-and-wrapper-object)*原始类型和包装对象*

*原始类型通常以字面量形式表示，但是布尔值（Boolean）、数值（Number）、字符串（String）也可以作为对象来表示。 这些对象称为**包装对象**，它们包装了原始类型的值。

使用`new`操作符和相应的构造函数可以创建包装对象。 例如，用于原始字符串的构造函数是`String`。

下面的代码创建了`String`的包装对象。 由于包装对象本质上也是对象，因此`typeof`运算符的结果也是`"object"`。 此外，由于它是对象，因此可以访问对象具有的属性，例如`length`属性。

```
// 文字列をラップしたStringラッパーオブジェクト
const str = new String("文字列");
// ラッパーオブジェクトは"object"型のデータ
console.log(typeof str); // => "object"
// Stringオブジェクトの`length`プロパティは文字列の長さを返す
console.log(str.length); // => 3 
```

但是，没有明确使用包装对象的理由。 因为 JavaScript 允许像对象一样引用原始类型的数据，这是因为存在隐式类型转换到包装对象的机制。 在下面的代码中，我们可以访问原始类型的字符串数据的`length`属性。

```
// プリミティブ型の文字列データ
const str = "文字列";
// プリミティブ型の文字列は"string"型のデータ
console.log(typeof str); // => "string"
// プリミティブ型の文字列も`length`プロパティを参照できる
console.log(str.length); // => 3 
```

这是因为在访问

关于这种隐式类型转换到包装对象的机制将在“包装对象”章节中进行解释。 目前，只需知道即使是原始类型的数据也可以像对象一样访问属性（包括方法）即可。

## [](#data-type-summary)*摘要*

*在本章中，我们学习了有关数据类型和字面量的知识。

+   有 7 种原始类型和对象

+   字面量是一种可以直接描述数据类型值的语法。

+   +   プリミティブ型的布尔值、数字、BigInt、字符串、null 都有字面量表示。

+   +   对象类型的对象、数组、正则表达式都有字面量表示。

+   +   即使是原始数据类型也可以访问属性。

> +   ¹. JavaScript 最初在 Netscape 上实现时有一个 bug，`typeof null === "object"`。 由于修复此 bug 会破坏已经依赖此行为的代码，因此修复被推迟，当前行为成为规范。 有关详细信息，请参阅[`2ality.com/2013/10/typeof-null.html`](https://2ality.com/2013/10/typeof-null.html)。 ↩******************
