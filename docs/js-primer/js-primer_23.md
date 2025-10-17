# 包装对象

> 原文：[`jsprimer.net/basic/wrapper-object/`](https://jsprimer.net/basic/wrapper-object/)

JavaScript 的数据类型可以分为原始类型和对象（详细信息请参考“数据类型与字面量”）。

在以下代码中，使用字符串字面量定义了原始类型的字符串值。原始类型的字符串不是`String`对象的实例。然而，即使是在原始类型的字符串中，也可以调用`String`对象的实例方法`toUpperCase`。

```
// Stringの`toUpperCase`メソッドを呼び出せる
"string".toUpperCase(); // => "STRING" 
```

原始类型的字符串可以调用`String`的实例方法，这看起来有些不可思议。

本章将解释为什么原始类型的值可以调用对象的函数。

## [](#primitive-type-and-wrapper-object)*原始类型与包装对象*

*在原始类型数据中，真值（Boolean）、数值（Number）、BigInt、字符串（String）和符号（Symbol）各自都有对应的对象。例如，对于字符串，有`String`对象。

通过`new`创建这个`String`对象，可以创建`String`对象的实例。

```
// "input value"の値をラップしたStringのインスタンスを生成
const str = new String("input value");
// StringのインスタンスメソッドであるtoUpperCaseを呼び出す
str.toUpperCase(); // => "INPUT VALUE" 
```

这样实例化的对象可以看作是包裹了原始类型值的对象。因此，这种对象被称为针对原始类型值的**包装对象**。

包装对象与原始类型的对应关系如下。

| 包装对象 | 原始类型 | 例如 |
| --- | --- | --- |
| `Boolean` | 真值 | `true`或`false` |
| `Number` | 数值 | `1`或`2` |
| `BigInt` | 大整数 | `1n`或`2n` |
| `String` | 文字串 | `"字符串"` |
| `Symbol` | 符号 | `Symbol("说明")` |

注记：没有为`undefined`和`null`提供对应的包装对象。

注意：包装对象正如其名，是对象。因此，使用`typeof`运算符查看包装对象时，结果会是`"object"`。

```
// プリミティブの文字列は"string"型
const str = "文字列";
console.log(typeof str); // => "string"
// ラッパーオブジェクトは"object"型
const stringWrapper = new String("文字列");
console.log(typeof stringWrapper); // => "object" 
```

## [](#convert-primitive-to-wrapper)*从原始类型值到包装对象的自动转换*

*在 JavaScript 中，当对原始类型的值进行属性访问时，会自动将其转换为对应的包装对象。例如，字符串`"string"`会自动转换为类似`new String("string")`的包装对象。这使得原始类型的字符串可以调用`String`的实例方法。

```
const str = "string";
// プリミティブ型の値に対してメソッド呼び出しを行う
str.toUpperCase();
// `str`へアクセスする際に"string"がラッパーオブジェクトへ変換され、
// ラッパーオブジェクトはStringのインスタンスなのでメソッドを呼び出せる
// つまり、上のコードは下のコードと同じ意味である
(new String(str)).toUpperCase(); 
```

这样，从原始类型值到包装对象的转换是自动进行的。¹

另一方面，也可以从显式创建的包装对象中提取原始类型的值。

通过调用`valueOf`方法可以从包装对象中提取值。例如，可以从以下字符串的包装对象中通过`valueOf`方法提取字符串。

```
const stringWrapper = new String("文字列");
// プリミティブ型の値を取得する
console.log(stringWrapper.valueOf()); // => "文字列" 
```

JavaScript 中，有使用字面量表示的原始类型字符串和用包装对象表示的字符串对象（真值和数值也是如此）。这两者之间没有明确的区分优势，因此推荐始终使用字面量。以下列举三个理由。

+   必要时，原始类型的字符串会自动转换为包装对象

+   `new String("string")`这样的包装对象实例的处理优势不存在

+   使用`typeof`运算符评估包装对象的结果是`"object"`而不是原始类型，这可能会引起混淆

由于这些原因，建议对原始类型的数据始终使用字面量。这样就不需要关注包装对象。

```
// OK: リテラルを使う
const str = "文字列";
// NG: ラッパーオブジェクトを使う
const stringWrapper = new String("文字列"); 
```

## [](#wrapper-object-summary)*总结*

*本章解释了为什么原始类型的值可以调用方法。其背后的机制是存在针对原始类型的包装对象。在访问原始类型的属性时，会自动转换为包装对象，从而使得方法调用等操作成为可能。

有时候会说“JavaScript 中一切都是对象”，虽然原始类型不是对象，但为原始类型提供了对应的包装对象（除了`null`和`undefined`）。因此，“看起来一切都是对象”这种认识是正确的。

> ¹. 这种从原始类型到对象类型的转换称为装箱（boxing），反之，从对象类型到原始类型的转换称为拆箱（unboxing）。 ↩***
