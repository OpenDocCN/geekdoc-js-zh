# 字符串和 Unicode

> 原文：[`jsprimer.net/basic/string-unicode/`](https://jsprimer.net/basic/string-unicode/)

「字符串」的章节介绍了，JavaScript 采用 Unicode 作为字符编码，并采用 UTF-16 作为编码方式。采用 UTF-16 仅仅是为了在 JavaScript 内部处理字符串时使用的字符编码（内部编码）。因此，文件本身的字符编码（外部编码）可以是 UTF-8 等非 UTF-16 的字符编码。

在「字符串」章节中，我们并没有意识到这些字符编码，可以在不意识到内部使用的字符编码是什么的情况下进行字符串处理。但是，JavaScript 的 String 对象也有针对这个字符编码（Unicode）的 API。此外，在处理特定包含表情符号的字符或计算“字符数”时，必须意识到 UTF-16 作为内部编码。

本章将讨论需要意识到字符串中的 Unicode 的情况。此外，Unicode 本身与 ECMAScript 一样具有悠久的历史规范，要介绍 Unicode 的所有内容需要大量的字符串。因此，本章将限定讨论 JavaScript 中的 Unicode 和 UTF-16。

如果想要了解 Unicode 的历史以及字符编码本身的更多信息，请参考“[程序员必备的文字编码技术入门](https://gihyo.jp/book/2019/978-4-297-10291-3)”或“[文字编码「超」研究](https://www.rutles.net/products/detail.php?product_id=298)”等。

## *代码点*

*Unicode 规范为所有字符（包括不可见字符等）定义了 ID。对于这些“字符”的“唯一 ID”，我们称之为**代码点**（代码点）。

处理代码点的方法大多是在 ECMAScript 2015 中添加的。通过使用 ES2015 中添加的 String 的 `codePointAt` 方法和 `String.fromCodePoint` 静态方法，可以相互转换字符串和代码点。

String 的 `codePointAt` 方法^([ES2015])会返回字符串中指定索引处字符的代码点值。

```
// 文字列"あ"のCode Pointを取得
console.log("あ".codePointAt(0)); // => 12354 
```

另一方面，`String.fromCodePoint` 方法^([ES2015])会返回与指定代码点对应的字符。

```
// Code Pointが`12354`の文字を取得する
console.log(String.fromCodePoint(12354)); // => "あ"
// `12354`を16 進数リテラルで表記しても同じ結果
console.log(String.fromCodePoint(0x3042)); // => "あ" 
```

此外，在字符串文字中，可以使用 Unicode 转义序列直接写入代码点。代码点可以以`\u{Code Point 的 16 进制值}`的形式作为转义序列进行编写。在 Unicode 转义序列中，需要提供代码点的 16 进制值。通过将 Number 的 `toString` 方法的基数参数设置为 `16`，可以获取 16 进制字符串。

```
// "あ"のCode Pointは12354
const codePointOfあ = "あ".codePointAt(0);
// 12354の16 進数表現は"3042"
const hexOfあ = codePointOfあ.toString(16);
console.log(hexOfあ);// => "3042"
// Unicodeエスケープで"あ"を表現できる
console.log("\u{3042}"); // => "あ" 
```

## *代码点与代码单元的区别*

*我们介绍了代码点，但 JavaScript 字符串的构成元素是经过 UTF-16 转换的代码单元（详细信息请参考“字符串”章节）。对于某些范围的字符串，代码点和代码单元的值最终会相同。

在下面的代码中，我们展示了名为`アオイ`的字符串的每个元素作为代码点和代码单元的表示。`convertCodeUnits` 函数将字符串转换为代码单元数组，`convertCodePoints` 函数将字符串转换为代码点数组。暂时不需要理解这两个函数的具体实现。

```
// 文字列をCode Unit(16 進数)の配列にして返す
function convertCodeUnits(str) {
    const codeUnits = [];
    for (let i = 0; i < str.length; i++) {
        codeUnits.push(str.charCodeAt(i).toString(16));
    }
    return codeUnits;
}
// 文字列をCode Point(16 進数)の配列にして返す
function convertCodePoints(str) {
    return Array.from(str).map(char => {
        return char.codePointAt(0).toString(16);
    });
}

const str = "アオイ";
const codeUnits = convertCodeUnits(str);
console.log(codeUnits); // => ["30a2", "30aa", "30a4"]
const codePoints = convertCodePoints(str);
console.log(codePoints); // => ["30a2", "30aa", "30a4"] 
```

总结执行结果后，可以看出在这个字符串中，代码点和代码单元的值是相同的。

![字符串中的代码单元和代码点表](img/d509219aa230c9091994e155711f62f1.png)

然而，有些字符串的代码点和代码单元可能会有不同的值。

使用相同的函数，我们将比较`リンゴ🍎`（苹果表情符号）这个字符串的代码单元和代码点。

```
// 文字列をCode Unit(16 進数)の配列にして返す
function convertCodeUnits(str) {
    const codeUnits = [];
    for (let i = 0; i < str.length; i++) {
        codeUnits.push(str.charCodeAt(i).toString(16));
    }
    return codeUnits;
}
// 文字列をCode Point(16 進数)の配列にして返す
function convertCodePoints(str) {
    return Array.from(str).map(char => {
        return char.codePointAt(0).toString(16);
    });
}

const str = "リンゴ🍎";
const codeUnits = convertCodeUnits(str);
console.log(codeUnits); // => ["30ea", "30f3", "30b4", "d83c", "df4e"]
const codePoints = convertCodePoints(str);
console.log(codePoints); // => ["30ea", "30f3", "30b4", "1f34e"] 
```

总结执行结果后，可以看出在包含表情符号的字符串中，代码点和代码单元的值是不同的。

![包含表情符号的字符串中的代码单元和代码点表](img/d20cf51e0c1e366dbe3477309403da4a.png)

具体来说，代码点的元素数为 4，而代码单元的元素数为 5。此外，虽然一个代码点对应于`🍎`，但在代码单元中，`🍎`对应于两个代码单元。在 JavaScript 中，字符串被视为按顺序排列的代码单元，因此该字符串的元素数（长度）为代码单元的个数，即 5。

UTF-16 是一种编码方式，用 16 位（2 字节）的代码单元来表示一个对应于代码点的字符。然而，16 位（2 字节）可以表示的范围只有 65536 种（2 的 16 次方）。目前，Unicode 注册的代码点已经超过 10 万种，因此无法将所有字符与代码单元进行一对一的映射。

在这种情况下，UTF-16 使用两个代码单元的组合（总计 4 个字节）来表示一个字符（一个代码点）。这种机制被称为**代理对**。

## *代理对*

*代理对中，由 2 个代码单元的组合（总计 4 个字节）来表示一个字符（一个代码点）。UTF-16 中，使用以下范围作为代理对的利用区域。

+   `\uD800`～`\uDBFF`：高位代理的范围

+   `\uDC00`～`\uDFFF`：低位代理的范围

当文字列中包含高位代理和低位代理的代码单元时，将两个代码单元组合成一个文字（代码点）进行处理。

以下代码中，展示了由代理对表示的字符「𩸽（ほっけ）」使用两个代码单元进行表达。通过将两个代码单元的转义序列（`\uXXXX`）并列，可以表示字符「𩸽」。另一方面，从 ES2015 开始，也可以使用代码点的转义序列（`\u{XXXX}`）来表示，因此可以使用一个代码点来表示字符「𩸽」。然而，即使使用代码点的转义序列，内部仍然会将其转换为代码单元的值。

```
// 上位サロゲート + 下位サロゲートの組み合わせ
console.log("\uD867\uDE3D"); // => "𩸽"
// Code Pointでの表現
console.log("\u{29e3d}"); // => "𩸽" 
```

在之前的例子中出现的`🍎`（苹果表情符号）也是用替代对表示的文字。

```
// Code Unit（上位サロゲート + 下位サロゲート）
console.log("\uD83C\uDF4E"); // => "🍎"
// Code Point
console.log("\u{1F34E}"); // => "🍎" 
```

这样，在替代对中，两个 Code Unit 表示一个 Code Point。

基本上，字符串被视为按顺序排列的 Code Unit，因此许多`String`方法都是按 Code Unit 操作的。此外，索引访问也是按 Code Unit 进行的。因此，用替代对表示的字符串中，索引访问是针对上位替代符（0 号）和下位替代符（1 号）的。

```
// 内部的にはCode Unitが並んでいるものとして扱われている
console.log("\uD867\uDE3D"); // => "𩸽"
// インデックスアクセスもCode Unitごととなる
console.log("𩸽"[0]); // => "\uD867"
console.log("𩸽"[1]); // => "\uDE3D" 
```

当字符串中包含表情符号或“𩸽（ほっけ）”等用替代对表示的文字时，按 Code Unit 处理的字符串处理会变得复杂。

例如，String 的`length`属性计算的是字符串中的 Code Unit 元素数，因此`"🍎".length`的结果是`2`。

```
console.log("🍎".length); // => 2 
```

在这种情况下，需要考虑按 Code Point 处理字符串。

## *处理 Code Point*

*要按 Code Point 顺序排列字符串，需要使用与 Code Point 对应的函数等方法。

从 ES2015 开始，增加了处理字符串为 Code Point 的方法或语法。接下来要介绍的是，这些方法将处理字符串为 Code Point。

+   包含`CodePoint`名称的方法

+   `u`（Unicode）标志已启用的正则表达式

+   处理字符串的 Iterator（解构、`for...of`、`Array.from`方法等）

我们将查看处理这些 Code Point 的具体方法和用法。

### *正则表达式的`.`和 Unicode*

*ES2015 中，正则表达式添加了`u`（Unicode）标志。添加了`u`标志的正则表达式将按 Code Point 顺序排列的字符串进行处理。

具体来说，我们将查看添加或未添加`u`标志的`.`（除了换行符以外的任意一个字符）的行为差异。

以`/（.）のひらき/`这种模式为例，看看如何提取匹配`.`的部分。

首先，我们可以通过添加`u`标志的正则表达式和 String 的`match`方法来提取匹配的范围内。`match`方法返回的值是`[匹配的整个字符串, 被捕获的字符串]`（详细信息请参考“字符串”章节）。

实际匹配的结果显示，`.`与`𩸽`的下位替代符`\ude3d`相匹配（`\ude3d`单独无法显示，因此会以乱码的形式显示）。

```
const [all, fish] = "𩸽のひらき".match(/(.)のひらき/);
console.log(all); // => "\ude3dのひらき"
console.log(fish); // => "\ude3d" 
```

也就是说，没有添加`u`标志的正则表达式将字符串视为按顺序排列的 Code Unit。

要避免这种意外的结果，可以在正则表达式中添加`u`标志。添加了`u`标志的正则表达式将按 Code Point 处理字符串。因此，任意一个字符匹配的`.`将匹配`𩸽`这个字符（Code Point）。

```
const [all, fish] = "𩸽のひらき".match(/(.)のひらき/u);
console.log(all); // => "𩸽のひらき"
console.log(fish); // => "𩸽" 
```

基本上，添加了`u`标志的正则表达式应该很少出现问题。这是因为很少需要编写只匹配替代对中一个部分的正则表达式。

### *计算 Code Point 的数量*

*String 的`length`属性表示的是构成字符串的 Code Unit 的个数。因此，包含替代对的字符串中，`length`的结果可能会比预期的要大。

```
// Code Unitの個数を返す
console.log("🍎".length); // => 2
console.log("\uD83C\uDF4E"); // => "🍎"
console.log("\uD83C\uDF4E".length); // => 2 
```

JavaScript 没有提供计算字符串中 Code Point 个数的内置方法。为此，需要将字符串转换为按 Code Point 分隔的数组，然后计算数组的长度。

`Array.from`方法^([ES2015])接受一个可迭代的对象作为参数，并返回一个新的数组。可迭代的对象是指实现了`Symbol.iterator`这个特殊名称的方法的对象的总称，`for...of`语句等可以对其进行迭代处理（详细信息请参考“循环与重复处理的 for...of 语句”章节）。

字符串也是可迭代的对象，因此可以使用`Array.from`方法将其转换为按 1 个字符（严格来说是一个 Code Point）分隔的数组。正如之前所介绍的，当将字符串作为可迭代对象处理时，应按 Code Point 进行处理。

```
// Code Pointごとの配列にする
// Array.fromメソッドはIteratorを配列にする
const codePoints = Array.from("リンゴ🍎");
console.log(codePoints); // => ["リ", "ン", "ゴ", "🍎"]
// Code Pointの個数を数える
console.log(codePoints.length); // => 4 
```

然而，在计算 Code Point 的数量时，有时也可能得到不直观的结果。这是因为 Code Point 中定义了诸如控制字符等视觉上不可见的字符。因此，在需要计算视觉上的**字符串长度**时，需要额外的技巧。

### *按 Code Point 进行循环处理*

*使用之前介绍的`Array.from`方法，可以将字符串转换为按 Code Point 分隔的字符数组。然后，可以使用“循环与重复处理”章节中学到的技术，对 Code Point 进行循环处理。

下面的代码计算了字符串中出现`🍎`的次数。`countOfCodePoints`函数将使用`Array.from`将字符串转换为 Code Point 数组，并返回由数组过滤器`codePoint`产生的数组的元素数。

```
// 指定した`codePoint`の個数を数える
function countOfCodePoints(str, codePoint) {
    return Array.from(str).filter(item => {
        return item === codePoint;
    }).length;
}
console.log(countOfCodePoints("🍎🍇🍎🥕🍒", "🍎")); // => 2 
```

`for...of`循环也可以按 Code Point 处理字符串。这是因为`for...of`语句会枚举目标作为迭代器。

我们将使用与之前相同的`countOfCodePoints`函数，但这次使用`for...of`来实现。

```
// 指定した`codePoint`の個数を数える
function countOfCodePoints(str, codePoint) {
    let count = 0;
    for (const item of str) {
        if (item === codePoint) {
            count++;
        }
    }
    return count;
}
console.log(countOfCodePoints("🍎🍇🍎🥕🍒", "🍎")); // => 2 
```

## *结论*

*本章简要介绍了字符串和 Unicode 之间的关系。Unicode 还有一些未在本章介绍的方面。此外，JavaScript 并非总是提供处理 Unicode 的完美 API。

另一方面，正如在“字符串”章节中介绍的那样，即使不考虑 Code Unit 或 Code Point，也可以进行灵活而强大的字符串处理。然而，由于近年来使用表情符号的情况越来越多，因此也越来越需要意识到按 Code Point 编程的情况。

Unicode 是一个独立于 ECMAScript 的规范，因此，处理字符串的问题成为了所有编程语言都会遇到的共同问题。特别是 Java 采用了与 JavaScript 相同的 UTF-16 编码方式，因此存在类似的问题。因此，在遇到 JavaScript 中的字符串处理问题时，了解其他语言的处理方式也是很重要的。********
