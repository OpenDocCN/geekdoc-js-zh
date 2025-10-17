# 变量与声明

> 原文：[`jsprimer.net/basic/variables/`](https://jsprimer.net/basic/variables/)

编程语言中，通过给数据（如字符串、数值等）命名，使其可以重复使用，这种功能称为**变量**。

JavaScript 有`const`、`let`、`var`三个关键字用于声明“这是变量”。

`var`是最早的变量声明关键字之一，但存在容易引起意外行为的问题。因此，在 ECMAScript 2015 中，为了改善`var`的问题，引入了新的关键字`const`和`let`。

本章将按顺序介绍`const`、`let`、`var`，分别探讨使用这些方法声明的变量的不同之处。

## *[ES2015] `const`*

*使用`const`关键字可以声明不能重赋的变量，并定义该变量所引用的值（初始值）。

可以像以下这样，在`const`关键字后面写上`变量名`，在赋值运算符（`=`）的右边写上变量的`初始值`来定义变量。

```
const 変数名 = 初期値; 
```

在以下代码中，声明了变量`bookTitle`，并定义了其初始值为字符串`"JavaScript Primer"`。

```
const bookTitle = "JavaScript Primer"; 
```

`const`、`let`、`var`这三个关键字具有共同的机制，但可以通过`,`（逗号）来分隔变量，从而同时定义多个变量。

在以下代码中，依次定义了变量`bookTitle`和`bookCategory`。

```
const bookTitle = "JavaScript Primer",
      bookCategory = "プログラミング"; 
```

这与以下写法具有相同的意义。

```
const bookTitle = "JavaScript Primer";
const bookCategory = "プログラミング"; 
```

此外，`const`是用于声明不能重赋的变量的关键字。因此，使用`const`关键字声明的变量，之后不能对其进行值赋。

在以下代码中，由于使用`const`声明了变量`bookTitle`并对其进行了值的重赋，因此发生了以下错误（`TypeError`）。错误发生会导致后续的处理无法执行。

```
const bookTitle = "JavaScript Primer";
bookTitle = "新しいタイトル"; // => TypeError: invalid assignment to const 'bookTitle' 
```

通常，变量到值的重赋会破坏所谓的“变量引用透明性”规则，这是导致 bug 容易发生的原因之一。因此，如果不需要对变量进行值的重赋，建议使用`const`关键字进行变量声明。

在需要值重赋的变量场景中，例如在循环等重复处理过程中，可能需要改变特定变量所引用的值。在这种情况下，可以使用可以值重赋的`let`关键字。

## *[ES2015] `let`*

*使用`let`关键字可以声明可以值重赋的变量。`let`的使用方法与`const`几乎相同。

在以下代码中，声明了变量`bookTitle`，并定义了其初始值为字符串`"JavaScript Primer"`。

```
let bookTitle = "JavaScript Primer"; 
```

`let`与`const`不同，也可以定义没有初始值的变量。没有指定初始值的变量将使用默认值`undefined`进行初始化（`undefined`表示值未定义）。

在以下代码中，声明了变量`bookTitle`。此时，由于没有指定初始值，因此默认使用`undefined`进行初始化。

```
let bookTitle;
// `bookTitle`は自動的に`undefined`という値になる 
```

使用`let`声明的变量`bookTitle`可以通过使用赋值运算符（`=`）来赋值。赋值运算符（`=`）的右边写上要赋给变量的值，这里写的是字符串`"JavaScript Primer"`。

```
let bookTitle;
bookTitle = "JavaScript Primer"; 
```

使用`let`声明的变量可以进行多次值赋。

```
let count = 0;
count = 1;
count = 2;
count = 3; 
```

## *`var`*

*使用`var`关键字可以声明可以值重赋的变量。`var`的使用方法与`let`几乎相同。

```
var bookTitle = "JavaScript Primer"; 
```

在`var`中，可以像`let`一样声明没有初始值的变量，并且可以对变量进行值的重赋。

```
var bookTitle;
bookTitle = "JavaScript Primer";
bookTitle = "新しいタイトル"; 
```

### *`var`的问题*

*`var`与`let`很相似，但`var`关键字可以重新定义同名变量。

`let`或`const`中，如果尝试重新定义同名变量，则会发生以下语法错误（`SyntaxError`）。这可以防止错误地重复定义变量。

```
// "x"という変数名で変数を定義する
let x;
// 同じ名前の変数"x"を定義するとSyntaxErrorとなる
let x; // => SyntaxError: redeclaration of let x 
```

另一方面，`var`可以重新定义同名变量。这可能导致在不经意间使用相同的变量名进行定义，而不会产生错误，并覆盖原有的值。

```
// "x"という変数を定義する
var x = 1;
// 同じ名前の変数"x"を定義できる
var x = 2;
// 変数 xは2となる 
```

此外，`var`存在所谓的“变量提升”的意外行为，而`let`或`const`则解决了这个问题。关于`var`引起的变量提升问题，将在“函数与作用域”章节中解释。因此，目前只需记住“`let`是改进版的`var`”即可。

这样，`var`存在许多问题。而且，在几乎所有情况下，都可以用`const`或`let`替换`var`。因此，对于将要编写的代码，最好避免使用`var`。

## *[专栏] 为什么会添加`let`和`const`？*

*在 ECMAScript 2015 中，不是通过改善`var`本身，而是通过添加新的关键字`const`和`let`来避免`var`的问题。没有改变`var`本身的动作是为了保持向后兼容性。

因为如果改变`var`的行为，那么已经用`var`编写的代码的行为也会改变，可能导致应用程序无法运行。因此，新添加的`const`或`let`等关键字虽然被添加到 ECMAScript 规范中，但使用这些关键字的源代码在添加时并不存在。¹ 因此，即使添加了`const`或`let`，也不会影响向后兼容性。

这样，ECMAScript 在添加新功能时也重视向后兼容性，因此没有改变`var`本身的动作。

## *变量名可以使用规则*

*到目前为止，我们已经了解了`const`、`let`、`var`关键字以及它们各自的变量声明特性。 在任何关键字中，可用于声明变量的名称规则都是相同的。 此外，此规则适用于 JavaScript 标识符，如变量名和函数名。

变量名（标识符）有以下规则：

1.  将变量名设为由半角字母、`_`（下划线）、`$`（美元符号）和数字`0`至`9`组成的名称

1.  变量名不能以数字开头

1.  不能使用保留字作为名称

变量名应为由半角字母`A`至`Z`（大写）和`a`至`z`（小写）、`_`（下划线）、`$`（美元符号）、数字`0`至`9`组成的名称。 在 JavaScript 中，区分大小写。

除此之外，虽然可以在变量名中使用平假名和部分汉字等，但是如果混合全角字符，则可能会因环境问题而难以处理，因此不建议使用。

```
let $; // OK: $が利用できる
let _title; // OK: _が利用できる
let jquery; // OK: 小文字のアルファベットが利用できる
let TITLE; // OK: 大文字のアルファベットが利用できる
let es2015; // OK: 数字は先頭以外なら利用できる
let 日本語の変数名; // OK: 一部の漢字や日本語も利用できる 
```

可以在变量名中包含数字，但是变量名不能以数字开头。 这是因为会导致无法区分变量名和数字。

```
let 1st; // NG: 数字から始まっている
let 123; // NG: 数字のみで構成されている 
```

此外，无法将保留字用作变量名。 保留字是指具有语法意义的关键字，例如`let`。 可以在[保留字 - JavaScript | MDN](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar)上找到保留字列表，但基本上是保留用于语法的名称。

```
let let; // NG: `let`は変数宣言のために予約されているので利用できない
let if; // NG: `if`はif 文のために予約されているので利用できない 
```

## *[专栏] `const`并非常量*

*`const`是定义“不能重新赋值的变量”的变量声明，并不一定定义常量。常量是指一旦定义的名称（变量名）始终表示相同值的东西。

在 JavaScript 中，也可以使用`const`声明接近常量的变量。如果使用`const`声明的变量初始化为无法更改的原始值，则实际上就是常量。原始值是指除对象之外的数据，例如数字和字符串（有关详细信息，请参阅“数据类型和字面量”章节）。

```
// TEN_NUMBERという変数は常に10という値を示す
const TEN_NUMBER = 10; 
```

但是，在 JavaScript 中，也可以使用`const`声明对象等。 例如，对象值本身在初始化后也可以更改。

```
// `const`でオブジェクトを定義している
const object = {
    key: "値"
};
// オブジェクトそのものは変更できてしまう
object.key = "新しい値"; 
```

因此，由于使用`const`声明的变量不一定始终表示相同的值，因此不能称之为常量（有关详细信息，请参见“对象”章节）。

此外，`const`没有变量命名规则，也没有赋值限制。因此，最好理解`const`声明的特性是定义“不能重新赋值的变量”。

## *总结*

*本章介绍了 JavaScript 中用于声明变量的关键字`const`、`let`、`var`。

+   `const`可以声明不能重新赋值的变量

+   `let`可以声明可以重新赋值的变量

+   `var`可以声明可以重新赋值的变量，但已知存在一些问题

+   变量名（标识符）有一些可用的名称规则

`var`在大多数情况下都可以用`let`或`const`来替代。`const`定义了不能重新赋值的变量。通过禁止重新赋值，可以减少因错误而引起的 bug。因此，在声明变量时，首先考虑是否可以用`const`定义，如果不行，则建议使用`let`。

> ¹. 由于`let`和`const`在 ECMAScript 2015 之前被定义为保留字，所以不会与现有代码冲突。 ↩********
