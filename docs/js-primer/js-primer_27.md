# 例外处理

> 原文：[`jsprimer.net/basic/error-try-catch/`](https://jsprimer.net/basic/error-try-catch/)

本章将学习 JavaScript 中的异常处理。

## *try...catch 语句*

*[try...catch](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/try...catch)语句是用于标记可能发生异常的代码块，并描述异常发生时处理方式的语句。

当`try...catch`语句的`try`块中发生异常时，`try`块中的后续处理不会执行，处理将转移到`catch`块。`catch`块在`try`块内发生异常时，会与发生的错误对象一起调用。`finally`块无论`try`块内是否发生异常，都会在`try`语句的最后执行。

在下面的代码中，`try`块中发生异常，执行了`catch`块的处理，最后执行了`finally`块的处理。

```
try {
    console.log("try 節:この行は実行されます");
    // 未定義の関数を呼び出してReferenceError 例外が発生する
    undefinedFunction();
    // 例外が発生したため、この行は実行されません
} catch (error) {
    // 例外が発生したあとはこのブロックが実行される
    console.log("catch 節:この行は実行されます");
    console.log(error instanceof ReferenceError); // => true
    console.log(error.message); // => "undefinedFunction is not defined"
} finally {
    // このブロックは例外の発生に関係なく必ず実行される
    console.log("finally 節:この行は実行されます");
} 
```

此外，如果`catch`块和`finally`块中有一个存在，则可以省略另一个块。如果只写了`finally`块，则即使没有捕获到异常，`finally`块也会在执行后执行。

```
// catch 節のみ
try {
    undefinedFunction();
} catch (error) {
    console.error(error);
}
// finally 節のみ
try {
    undefinedFunction();
} finally {
    console.log("この行は実行されます");
}
// finally 節のみでは例外がキャッチされないため、この行は実行されません 
```

## *throw 语句*

*[throw](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/throw)语句可以使用户抛出异常。抛出的异常对象可以作为函数的参数在`catch`块中访问。在`catch`块中可以引用对象的标识符称为[异常标识符](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/try...catch#The_exception_identifier)。

在下面的代码中，使用`catch`块的`error`标识符引用了捕获到的错误对象。

```
try {
    // 例外を投げる
    throw new Error("例外が投げられました");
} catch (error) {
    // catch 節のスコープでerrorにアクセスできる
    console.log(error.message); // => "例外が投げられました"
} 
```

## *错误对象*

*`throw`语句可以抛出任何对象作为异常。这里将探讨使用`throw`语句抛出的错误对象。

### *Error*

*`Error`对象的实例可以通过`new Error("错误消息")`创建。构造函数的第一个参数是作为错误消息的字符串。传递的错误消息可以通过 Error 的`message`属性进行引用。

在下面的代码中，使用`assertPositiveNumber`函数创建错误对象，并将其作为异常抛出。抛出的对象可以通过`catch`块的异常标识符（`error`）获取，并可以检查错误消息。

```
// 渡された数値が0 以上ではない場合に例外を投げる関数
function assertPositiveNumber(num) {
    if (num < 0) {
        throw new Error(`${num} is not positive.`);
    }
}

try {
    // 0 未満の値を渡しているので、関数が例外を投げる
    assertPositiveNumber(-1);
} catch (error) {
    console.log(error instanceof Error); // => true
    console.log(error.message); // => "-1 is not positive."
} 
```

`throw`语句可以抛出任何对象作为异常，但通常推荐抛出`Error`对象的实例。原因将在后面讨论**堆栈跟踪**。`Error`对象在实例创建时，会包含有关实例创建的文件名或行数等对调试有用的信息。如果抛出不是`Error`对象的实例，如字符串等，则无法获取堆栈跟踪。

因此，不建议使用`throw`语句抛出非`Error`对象。

```
// 文字列を例外として投げるアンチパターンの例
try {
    throw "例外が投げられました";
} catch (error) {
    // catch 節の例外識別子は、投げられた値を参照する
    console.log(error); // => "例外が投げられました"
} 
```

### *内置错误*

*错误有几种类型，这些类型被定义为内置错误。内置错误是指 ECMAScript 规范或执行环境中预定义的错误对象。可以抛出的内置错误对象是继承自`Error`对象的实例。因此，可以像用户定义的错误一样处理异常。

内置错误有多种类型，但这里将介绍一些代表性的。

#### *ReferenceError*

*[ReferenceError](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/ReferenceError)是在尝试引用不存在的变量或函数等标识符时发生的错误。在下面的代码中，由于正在引用不存在的变量，因此抛出了`ReferenceError`异常。

```
try {
    // 存在しない変数を参照する
    console.log(x);
} catch (error) {
    console.log(error instanceof ReferenceError); // => true
    console.log(error.name); // => "ReferenceError"
    console.log(error.message); // エラーメッセージが表示される
} 
```

#### *SyntaxError*

*[SyntaxError](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/SyntaxError)是在尝试解释非法代码时发生的错误。基本上，`SyntaxError`异常是在 JavaScript 执行前的解析阶段发生的。因此，在执行前发生的`SyntaxError`异常不能在执行时的`try...catch`语句中被捕获。

```
// JavaScriptとして正しくない構文をパースするとSyntaxErrorが発生する
foo! bar! 
```

在下面的代码中，使用`eval`函数强制在运行时发生`SyntaxError`，并确认语法错误是`SyntaxError`。

```
try {
    // eval 関数は渡した文字列をJavaScriptとして実行する関数
    // 正しくない構文をパースさせ、SyntaxErrorを実行時に発生させる
    eval("foo! bar!");
} catch (error) {
    console.log(error instanceof SyntaxError); // => true
    console.log(error.name); // => "SyntaxError"
    console.log(error.message); // エラーメッセージが表示される
} 
```

#### *TypeError*

*[TypeError](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/TypeError)是在值不是期望的类型时发生的错误。在下面的代码中，由于正在调用函数的对象不是函数，因此抛出了`TypeError`异常。

```
try {
    // 関数ではないオブジェクトを関数として呼び出す
    const fn = {};
    fn();
} catch (error) {
    console.log(error instanceof TypeError); // => true
    console.log(error.name); // => "TypeError"
    console.log(error.message); // エラーメッセージが表示される
} 
```

### *抛出内置错误*

*可以创建内置错误的实例，并将该实例作为异常抛出。与普通的`Error`对象一样，可以通过`new`每个内置错误对象来创建实例。

例如，如果要限制函数参数为字符串，可以抛出`TypeError`异常。即使不查看消息，仅凭错误名称就可以立即了解与类型相关的异常。

```
// 文字列を反転する関数
function reverseString(str) {
    if (typeof str !== "string") {
        throw new TypeError(`${str} is not a string`);
    }
    return Array.from(str).reverse().join("");
}

try {
    // 数値を渡す
    reverseString(100);
} catch (error) {
    console.log(error instanceof TypeError); // => true
    console.log(error.name); // => "TypeError"
    console.log(error.message); // => "100 is not a string"
} 
```

## *错误和调试*

*在 JavaScript 开发中，理解调试过程中发生的错误非常重要。通过利用错误提供的信息，可以了解源代码中发生异常的位置和类型。

所有错误都是由扩展自`Error`对象的对象声明的。也就是说，错误具有表示名称的`name`属性和表示内容的`message`属性。通过检查这两个属性，可以在许多情况下提供开发帮助。

在下面的代码中，未包含在`try...catch`块中的部分发生了异常。

```
function fn() {
    // 存在しない変数を参照する
    x++;
}
fn(); 
```

加载此脚本后，控制台将输出有关抛出异常的日志。以下是 Firefox 中的执行示例。

![控制台中的错误显示（Firefox）](img/fa3c3b721816f38fb58865327e9ee6c3.png)

此错误日志包含以下信息。

| 消息 | 含义 |
| --- | --- |
| `ReferenceError: x is not defined` | 错误类型为`ReferenceError`，表示`x`未定义。 |
| `error.js:3:5` | 异常发生在`error.js`的第 3 行第 5 列。也就是`x++;`。 |

此外，消息后面还显示了异常的堆栈跟踪。堆栈跟踪记录了程序执行过程，指出了哪些处理导致错误。

+   堆栈跟踪的第一行是实际发生异常的位置。也就是说，在第 3 行的 `x++;` 发生了异常

+   接下来的行记录了该代码的调用者。也就是说，执行第 3 行代码的是第 5 行的`fn`函数调用

堆栈跟踪记录了调用者的信息。

控制台显示的错误日志包含大量信息。MDN 的[JavaScript 错误参考](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Errors)详细列出了浏览器抛出的内置错误类型和消息。在开发过程中遇到内置错误时，查阅参考以寻找解决方法是一个好习惯。

## *`console.error`和堆栈跟踪*

*`console.error`方法可以将消息与堆栈跟踪一起输出到控制台。

运行以下代码，比较`console.log`和`console.error`的输出结果。

```
function fn() {
    console.log("メッセージ");
    console.error("エラーメッセージ");
}

fn(); 
```

在 Firefox 中运行此代码，控制台输出将如下图所示。

![console.log 和 console.error 的输出结果](img/0acbabe37f577404122b0b87f7220370.png)

`console.log`只有消息，而`console.error`除了消息外还输出堆栈跟踪。因此，在控制台输出错误消息时，使用`console.error`可以更容易进行调试。

此外，大多数浏览器都提供了过滤`console.log`和`console.error`输出的功能。通过使用`console.log`进行普通日志输出，使用`console.error`进行与错误相关的日志输出，可以更容易区分日志的重要性。

## *[ES2022] 错误原因*

*通过捕获错误并重新抛出带有新消息的另一个错误，可以提供有用的调试信息。通过创建新的 Error 对象并抛出来实现。但是，这种方法会导致原始错误的堆栈跟踪丢失。

```
function somethingWork() {
    throw new Error("本来のエラー");
}

try {
    somethingWork();
} catch (error) {
    // `error` が持っていたスタックトレースが失われるため、実際にエラーが発生した場所がわからなくなる 
    throw new Error("somethingWork 関数でエラーが発生しました");
} 
```

要解决堆栈跟踪丢失的问题，可以使用 ES2022 中引入的 Error 的`cause`选项。在创建新错误对象时，通过将原始错误对象传递给第二个参数的`cause`选项，可以保留原始的堆栈跟踪。

```
// 数値の文字列を受け取り数値を返す関数
// 'text' など数値にはならない文字列を渡された場合は例外を投げられる
function safeParseInt(numStr) {
    const num = Number.parseInt(numStr, 10);
    if (Number.isNaN(num)) {
        throw new Error(`${numStr} is not a numeric`);
    }
    return num;
}

// 数字の文字列を二つ受け取り、合計を返す関数
function sumNumStrings(a, b) {
    try {
        const aNumber = safeParseInt(a);
        const bNumber = safeParseInt(b);
        return aNumber + bNumber;
    } catch (e) {
        throw new Error("Failed to sum a and b", { cause: e });
    }
}

try {
    // 数値にならない文字列 'string' を渡しているので例外が投げられる
    sumNumStrings("string", "2");
} catch (err) {
    console.error(err);
} 
```

加载此脚本后，控制台将输出带有从`sumNumsStrings`到`safeParseInt`抛出的堆栈跟踪的错误日志。以下是 Firefox 中的执行示例。

![包含 safeParseInt 堆栈跟踪的 console.error 输出结果](img/82ca3eb6268226652f98a961bfcdbfb8.png)

## *总结*

*本章介绍了异常处理和错误对象。

+   `try...catch`语法可以处理`try`块中发生的异常

+   `catch`块和`finally`块可以同时或单独编写

+   `throw`语句可以抛出异常，将`Error`对象作为异常抛出

+   `Error`对象包含了在 ECMAScript 规范和执行环境中定义的内置错误

+   `Error`对象中记录了堆栈跟踪，对调试非常有用

+   使用 Error Cause，可以创建一个新错误，继承另一个错误的堆栈跟踪*************
