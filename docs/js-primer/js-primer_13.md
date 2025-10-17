# 隐式类型转换

> 原文：[`jsprimer.net/basic/implicit-coercion/`](https://jsprimer.net/basic/implicit-coercion/)

本章将学习显式类型转换和隐式类型转换。

在“运算符”章节中，我推荐使用严格的等价运算符（`===`）而不是等价运算符（`==`），因为严格的等价运算符（`===`）不会进行隐式类型转换，可以直接比较值。

严格的等价运算符（`===`）在比较不同数据类型时，其比较结果始终为`false`。以下代码正在比较数值的`1`和字符串的`"1"`这两种不同的数据类型，因此结果为`false`。

```
// `===`では、異なるデータ型の比較結果はfalse
console.log(1 === "1"); // => false 
```

然而，等价运算符（`==`）在比较不同数据类型时，会先进行隐式类型转换，使其具有相同的类型后再进行比较。以下代码中，数值的`1`和字符串的`"1"`的比较结果为`true`。这是因为等价运算符（`==`）将右边的字符串`"1"`隐式地转换为数值的`1`，然后进行比较。

```
// `==`では、異なるデータ型は暗黙的な型変換をしてから比較される
// 暗黙的な型変換によって 1 == 1 のように変換されてから比較される
console.log(1 == "1"); // => true 
```

这样，由于隐式类型转换可能导致意外结果，因此在比较时应使用严格的等价运算符（`===`）。

作为另一个隐式类型转换的例子，让我们看看数值和布尔值的加法。在许多语言中，数值和布尔值的加法等不同数据类型的加法会导致错误。然而，在 JavaScript 中，由于隐式类型转换，加法操作可以无错误地处理。

在以下代码中，布尔值的`true`被隐式地转换为数值的`1`，然后进行加法处理。

```
// 暗黙的な型変換が行われ、数値の加算として計算される
1 + true; // => 2
// 次のように暗黙的に変換されてから計算される
1 + 1; // => 2 
```

JavaScript 中，不会发生错误，而是进行隐式类型转换。当隐式转换发生时，程序不会抛出异常，处理会继续进行，这使得错误难以发现。因此，应尽可能避免隐式类型转换。

本章将学习以下内容。

+   隐式类型转换是什么样的？

+   非隐式的显式类型转换方法

+   仅显式转换无法解决问题

## [](#what-is-implicit-coercion)*隐式类型转换是什么*

*隐式类型转换是指以下内容。

+   在某个处理过程中，指在处理过程中进行的非显式类型转换

隐式类型转换是在运算符的运算或函数的处理过程中进行的。在这里，我们将主要关注运算符中的隐式类型转换。

### [](#implicit-coercion-of-equal-operator)*等价运算符的隐式类型转换*

*最著名的隐式类型转换是之前提到的等价运算符（`==`）。等价运算符在比较操作数之前，会进行隐式类型转换，使其具有相同的类型。

如下使用等价运算符（`==`）进行的比较会产生令人惊讶的结果。

```
// 異なる型である場合に暗黙的な型変換が行われる
console.log(1 == "1"); // => true
console.log(0 == false); // => true
console.log(10 == ["10"]); // => true 
```

此外，由于等价运算符导致的不可预测的结果数量与比较值和类型的组合数量相同，因此记住等价运算符的比较结果在现实中是不切实际的。

![等价运算符比较结果汇总表。绿色表示为 true 的组合](https://dorey.github.io/JavaScript-Equality-Table/)

但是，避免等价运算符的隐式类型转换有一个简单的方法。

那就是始终使用严格的等价运算符（`===`）。在比较值时，始终使用严格的等价运算符可以避免隐式类型转换，从而直接比较值。

```
console.log(1 === "1"); // => false
console.log(0 === false); // => false
console.log(10 === ["10"]); // => false 
```

使用严格的等价运算符（`===`）可以避免意外的比较结果。因此，建议在比较时使用严格的等价运算符（`===`）而不是等价运算符（`==`）。

### [](#various-implicit-coercion)*各种隐式类型转换*

*其他运算符的具体例子也让我们看看。

在以下代码中，使用加号运算符处理数值的`1`和字符串的`"2"`。加号运算符（`+`）被双重定义，既可以执行数值的加法，也可以执行字符串的连接。在这种情况下，JavaScript 优先执行字符串连接。因此，数值的`1`被隐式地转换为字符串的`"1"`，然后进行字符串连接。

```
1 + "2"; // => "12"
// 演算過程で次のように暗黙的な型変換が行われる
"1" + "2"; // => "12" 
```

另一个例子，让我们看看数值和字符串之间的隐式类型转换。以下代码中，从数值的`1`减去字符串的`"2"`。

JavaScript 中没有对字符串的减号运算符（`-`）的定义。因此，对减号运算符的目标数值进行隐式类型转换。因此，将字符串的`"2"`隐式地转换为数值的`2`，然后进行减法。

```
1 - "2"; // => -1
// 演算過程で次のように暗黙的な型変換が行われる
1 - 2; // => -1 
```

对于两个值，还可以预测结果的类型。但是，当处理三个以上的值时，预测结果变得困难。

当使用`+`运算符进行三个以上值的运算时，如果值的数据类型混合，则结果会因运算顺序而异。

```
const x = 1, y = "2", z = 3;
console.log(x + y + z); // => "123"
console.log(y + x + z); // => "213"
console.log(x + z + y); // => "42" 
```

这样，在处理过程中，根据操作数的类型，会自动进行转换，这被称为**隐式类型转换**。

在隐式类型转换中，结果的类型依赖于操作数的类型。为了避免这种情况，需要使用非隐式转换，即显式类型转换。

## [](#explicit-coercion)*显式类型转换*

*查看将原始类型转换为显式类型的方法。

### [](#any-to-boolean)*任意值 → 布尔值*

*在 JavaScript 中，可以使用`Boolean`构造函数将任意值转换为`true`或`false`的布尔值。

```
Boolean("string"); // => true
Boolean(1); // => true
Boolean({}); // => true
Boolean(0); // => false
Boolean(""); // => false
Boolean(null); // => false 
```

在 JavaScript 中，以下值会被转换为`false`。

+   `false`

+   `undefined`

+   `null`

+   `0`

+   `0n`

+   `NaN`

+   `""`（空字符串）

由于隐式类型转换，以下这些值会被汇总为**falsy**值。非 falsy 值会被转换为`true`。

这种转换规则与 if 语句的条件表达式评估相同。当向 if 语句传递非布尔值时，会将其隐式地转换为布尔值进行判断。

```
// x は undefined
let x; 
if (!x) {
    console.log("falsyな値なら表示", x);
} 
```

关于布尔值，由于隐式类型转换的规则较少，因此经常直接处理而不进行显式转换。但是，为了更精确地判断布尔值，可以使用如下严格的等价运算符（`===`）进行比较。

```
// x は undefined
let x;
if (x === undefined) {
    console.log("xがundefinedなら表示", x);
} 
```

### [](#number-to-string)*数值 → 字符串*

*要明确从数字转换为字符串，可以使用`String`构造函数。

```
String(1); // => "1" 
```

`String`构造函数还可以将除了数字之外的各种值转换为字符串。

```
String("str"); // => "str"
String(true); // => "true"
String(null); // => "null"
String(undefined); // => "undefined"
String(Symbol("シンボルの説明文")); // => "Symbol(シンボルの説明文)"
// プリミティブ型ではない値の場合
String([1, 2, 3]); // => "1,2,3"
String({ key: "value" }); // => "[object Object]"
String(function() {}); // "function() {}" 
```

正如上面的结果所示，`String`构造函数的明确转换并不是一种万能方法。 对于布尔值、数字、字符串、未定义、null、符号等原始类型的值，转换结果可能不尽人意。

另一方面，对于对象，返回的字符串通常没有太多意义。 这是因为对于对象，存在比`String`构造函数更适合的方法。 对于数组，有`join`方法，对于对象，有`JSON.stringify`方法等。 因此，`String`构造函数的转换应仅限于原始类型。

### [](#symbol-to-string)*符号 → 字符串*

*当将加法运算符用于字符串时，优先考虑字符串连接。 您可能会想：“如果其中一个是字符串，那么结果就是字符串，而不管另一个操作数是什么？”。

```
"文字列" + x; // 文字列となる？ 
```

然而，在 ES2015 中添加的原始类型 Symbol 无法进行隐式类型转换。 对符号使用字符串连接运算符会抛出异常。 因此，即使一个是字符串，加法运算符的结果也不一定是字符串。

下面的代码示例中，使用字符串连接运算符(`+`)无法将符号转换为字符串，因此会引发`TypeError`异常。

```
"文字列と" + Symbol("シンボルの説明"); // => TypeError: can't convert symbol to string 
```

通过使用`String`构造函数，可以解决这个问题，将符号明确转换为字符串。

```
"文字列と" + String(Symbol("シンボルの説明")); // => "文字列とSymbol(シンボルの説明)" 
```

### [](#string-to-number)*字符串 → 数值*

*将字符串转换为数字的典型案例是接收用户输入的数字。 用户输入始终以字符串形式接收，因此必须先将其转换为数字才能使用。

若要明确从字符串转换为数字，可以使用`Number`构造函数。

```
// ユーザー入力を文字列として受け取る
const input = window.prompt("数字を入力してください", "42");
// 文字列を数値に変換する
const num = Number(input);
console.log(typeof num); // => "number"
console.log(num); // 入力された文字列を数値に変換したもの 
```

此外，还可以使用`Number.parseInt`、`Number.parseFloat`等函数从字符串中提取并转换数字。 `Number.parseInt`从字符串中提取整数，`Number.parseFloat`从字符串中提取浮点数。 `Number.parseInt(字符串, 基数)`的第二个参数指定基数。 例如，如果要解析字符串以获取十进制数值，则将`10`作为第二个参数。

```
// "1"をパースして10 進数として取り出す
console.log(Number.parseInt("1", 10)); // => 1
// 余計な文字は無視してパースした結果を返す
console.log(Number.parseInt("42px", 10)); // => 42
console.log(Number.parseInt("10.5", 10)); // => 10
// 文字列をパースして浮動小数点数として取り出す
console.log(Number.parseFloat("1")); // => 1
console.log(Number.parseFloat("42.5px")); // => 42.5
console.log(Number.parseFloat("10.5")); // => 10.5 
```

然而，用户并不总是输入数字。 `Number`构造函数、

```
// 数字ではないため、数値へは変換できない
Number("文字列"); // => NaN
// 未定義の値はNaNになる
Number(undefined); // => NaN 
```

因此，如果从任意值转换为数字后结果为`NaN`，则需要编写相应的处理。 可以使用`Number.isNaN(x)`方法来检查转换结果是否为`NaN`。

```
const userInput = "任意の文字列";
const num = Number.parseInt(userInput, 10);
if (Number.isNaN(num)) {
    console.log("パースした結果 NaNになった", num);
} 
```

### [](#nan-is-number-type)*`NaN`是非数字但是属于 Number 类型*

*在这里，我们将更详细地了解频繁出现的值`NaN`的情况。 `NaN`是 Not a Number 的缩写，是一种具有特殊性质的 Number 类型的数据。

关于这种名为`NaN`的数据的性质在[IEEE 754](https://ja.wikipedia.org/wiki/IEEE_754)中有规定，并不仅限于 JavaScript。

创建`NaN`值的方法很简单，即将不兼容 Number 类型的数据转换为 Number 类型，结果将为`NaN`。 例如，对象是与数字不兼容的数据。 因此，即使明确转换对象，结果也将为`NaN`。

```
Number({}); // => NaN 
```

また、`NaN`是一种特殊的值，无论如何进行运算，结果都将是`NaN`。 下面的例子展示了当在计算过程中出现`NaN`值时，最终结果也会是`NaN`。

```
const x = 10;
const y = x + NaN;
const z = y + 20;
console.log(x); // => 10
console.log(y); // => NaN
console.log(z); // => NaN 
```

`NaN`看起来像是与其名字“Not a Number”相矛盾的数据。

```
// NaNはnumber 型
console.log(typeof NaN); // => "number" 
```

`NaN`具有一个特殊的属性，即它与自身不匹配。 通过利用这一特性，可以判断一个值是否为`NaN`。

```
function isNaN(x) {
    // NaNは自分自身と一致しない
    return x !== x;
}
console.log(isNaN(1)); // => false
console.log(isNaN("str")); // => false
console.log(isNaN({})); // => false
console.log(isNaN([])); // => false
console.log(isNaN(NaN)); // => true 
```

作为相似处理方式，可以使用`Number.isNaN(x)`方法。 在实际判断值是否为`NaN`时，最好使用`Number.isNaN(x)`方法。¹

```
Number.isNaN(NaN); // => true 
```

`NaN`是在隐式类型转换中最应该避免的值之一。 原因是，正如前面介绍的那样，无论进行何种运算，`NaN`的结果都是`NaN`。 这会使得计算中出现`NaN`的位置不清晰，从而导致调试困难。

例如，下面的`sum`函数接受可变数量的参数，并返回它们的总和。 但是，当调用`sum(x, y, z)`时，结果变为`NaN`。 这是因为参数中包含了`undefined`（未定义值）。

```
// 任意の個数の数値を受け取り、その合計値を返す関数
function sum(...values) {
    return values.reduce((total, value) => {
        return total + value;
    }, 0);
}
const x = 1, z = 10;
let y; // `y`はundefined
console.log(sum(x, y, z)); // => NaN 
```

因此，`sum(x, y, z);`将产生与下面相同的结果。 将数值加到`undefined`上会得到`NaN`。

```
sum(1, undefined, 10); // => NaN
// 計算中にNaNとなるため、最終結果もNaNになる
1 + undefined; // => NaN
NaN + 10; // => NaN 
```

这意味着，即使在`sum`函数中明确将参数转换为 Number 类型，问题也无法避免。 换句话说，可以看出明确的类型转换也无法解决这个问题。

```
function sum(...values) {
    return values.reduce((total, value) => {
        // `value`をNumberで明示的に数値へ変換してから加算する
        return total + Number(value);
    }, 0);
}
const x = 1, z = 10;
let y; // `y`はundefined
console.log(sum(x, y, z)); // => NaN 
```

避免意外的`NaN`转换的方法大致分为两种。

+   在`sum`函数的一侧（被调用方）中，仅接受 Number 类型的值

+   在调用`sum`函数时，仅传递 Number 类型的值

换句话说，这是由调用方或被调用方处理的，两者都会导致更安全的代码。

因此，需要明确指定`sum`函数仅接受数字。

明确的方法包括在`sum`函数的文档（注释）中描述，或者在参数中添加处理以抛出异常，表示除了数值以外的值。

在 JavaScript 中，用于描述参数类型的注释格式[JSDoc](https://jsdoc.app/)很有名。此外，通过在运行时检查值是否为 Number 类型并通过`throw`语句抛出异常，可以明确`sum`函数的使用方式（关于`throw`语句的解释将在“错误处理”章节中说明）。

通过这两个方面详细实现了`sum`函数的前提条件如下：

```
/**
 * 数値を合計した値を返します。
 * 1つ以上の数値と共に呼び出す必要があります。
 * @param {...number} values
 * @returns {number}
 **/
function sum(...values) {
    return values.reduce((total, value) => {
        // 値がNumber 型ではない場合に、例外を投げる
        if (typeof value !== "number") {
            throw new Error(`${value}はNumber 型ではありません`);
        }
        return total + Number(value);
    }, 0);
}
const x = 1, z = 10;
let y; // `y`はundefined
console.log(x, y, z);
// Number 型の値ではない`y`を渡しているため例外が発生する
console.log(sum(x, y, z)); // => Error 
```

通过明确说明`sum`函数的使用方式，当发生错误时，调用方和被调用方可以清楚地知道问题出在哪里。在这种情况下，问题出在将`undefined`值传递给`sum`函数的调用方。

JavaScript 对于类型错误非常宽容，例如会进行隐式类型转换，因此，在编写大型应用程序时，编写能够发现这些难以检测到的错误的代码非常重要。

## [](#unsolved-problem)*即使是显式转换也无法解决的问题*

*正如前面的例子所示，不是所有情况都可以通过显式转换来解决。将不兼容于 Number 类型的值转换为数字，结果将会是`NaN`。一旦出现`NaN`，就只能通过`Number.isNaN(x)`来判断并结束处理。

JavaScript 的类型转换基本上是向信息减少的方向进行的。因此，在进行显式转换之前，首先需要考虑是否需要转换。

### [](#judge-empty-string)*判断是否为空字符串*

*例如，假设我们想要判断字符串是否为空字符串。 `""`（空字符串）是 falsy 值，因此可以通过显式使用`Boolean`构造函数将其转换为布尔值。 但是，虽然 falsy 值不仅限于空字符串，但通过显式转换并不意味着只能判断空字符串。

在下面的代码中，虽然进行了显式类型转换，但`0`也会变成**空字符串**，导致意外行为。

```
// 空文字列かどうかを判定
function isEmptyString(str) {
    // `str`がfalsyな値なら、`isEmptyString`関数は`true`を返す
    return !Boolean(str);
}
// 空文字列列の場合は、trueを返す
console.log(isEmptyString("")); // => true
// falsyな値の場合は、trueを返す
console.log(isEmptyString(0)); // => true
// undefinedの場合は、trueを返す
console.log(isEmptyString()); // => true 
```

在大多数情况下，获取布尔值的方法不是通过类型转换，而是通过其他方式。

在这种情况下，空字符串被定义为“String 类型且长度为 0 的值”，这样一来，我们可以更准确地编写`isEmptyString`函数。通过以下实现，我们现在能够正确判断值是否为空字符串。

```
// 空文字列かどうかを判定
function isEmptyString(str) {
    // String 型でlengthが0の値の場合はtrueを返す
    return typeof str === "string" && str.length === 0;
}
console.log(isEmptyString("")); // => true
// falsyな値でも正しく判定できる
console.log(isEmptyString(0)); // => false
console.log(isEmptyString()); // => false 
```

使用`Boolean`进行类型转换是为了方便，但并不是获取准确布尔值的方法。因此，在进行类型转换之前，首先考虑是否可以通过其他方式解决问题也很重要。

## [](#conclusion)*总结*

*本章我们学习了隐式类型转换和显式类型转换。

+   避免使用隐式类型转换，因为它容易导致意外结果。

+   在比较时应使用严格等于运算符（`===`）而不是等于运算符（`==`）

+   使用函数进行显式类型转换而不是运算符进行隐式类型转换

+   除了显式类型转换外，还有其他方法可以获得布尔值。

> ¹. 虽然有名为`isNaN`的函数类似，但由于它无法正确判断`NaN`，因此建议使用`Number.isNaN`方法。 ↩************
