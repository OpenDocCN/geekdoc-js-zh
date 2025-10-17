# 関数とthis

> 原文：[`jsprimer.net/basic/function-this/`](https://jsprimer.net/basic/function-this/)

この章では`this`という特殊な動作をするキーワードについて見ていきます。基本的にはメソッドの中で利用しますが、`this`は読み取り専用のグローバル変数のようなものでどこにでも書けます。加えて、`this`の参照先（評価結果）は条件によって異なります。

`this`の参照先は主に次の条件によって変化します。

+   実行コンテキストにおける`this`

+   コンストラクタにおける`this`

+   関数とメソッドにおける`this`

+   Arrow Functionにおける`this`

コンストラクタにおける`this`は、次の章である「クラス」で扱います。この章ではさまざまな条件での`this`について扱いますが、`this`が実際に使われるのはメソッドにおいてです。そのため、あらゆる条件下での`this`の動きをすべて覚える必要はありません。

この章では、さまざまな条件下で変わる`this`の参照先と関数やArrow Functionとの関係を見ていきます。また、実際にどのような状況で問題が発生するかを知り、`this`の動きを予測可能にするにはどのようにするかを見ていきます。

## [](#execution-context-this)*実行コンテキストと`this`*

*最初に「JavaScriptとは」の章において、JavaScriptには実行コンテキストとして"Script"と"Module"があるという話をしました。どの実行コンテキストでJavaScriptのコードを評価するかは、実行環境によってやり方が異なります。この章では、ブラウザの`script`要素と`type`属性を使い、それぞれの実行コンテキストを明示しながら`this`の動きを見ていきます。

トップレベル（もっとも外側のスコープ）にある`this`は、実行コンテキストによって値が異なります。実行コンテキストの違いは意識しにくい部分であり、トップレベルで`this`を使うと混乱を生むことになります。そのため、コードのトップレベルにおいては`this`を使うべきではありませんが、それぞれの実行コンテキストにおける動作を紹介します。

### [](#script-this)*スクリプトにおける`this`*

*実行コンテキストが"Script"である場合、トップレベルのスコープに書かれた`this`はグローバルオブジェクトを参照します。グローバルオブジェクトは、実行環境ごとに異なるものが定義されています。ブラウザのグローバルオブジェクトは`window`オブジェクト、Node.jsのグローバルオブジェクトは`global`オブジェクトとなります。

ブラウザでは、`script`要素の`type`属性を指定していない場合は、実行コンテキストが"Script"として実行されます。この`script`要素の直下に書いた`this`はグローバルオブジェクトである`window`オブジェクトとなります。

```
<script> // 実行コンテキストは"Script"
console.log(this); // => window </script> 
```

### [](#module-this)*モジュールにおける`this`*

*実行コンテキストが"Module"である場合、そのトップレベルのスコープに書かれた`this`は常に`undefined`となります。

ブラウザで、`script`要素に`type="module"`属性がついた場合は、実行コンテキストが"Module"として実行されます。この`script`要素の直下に書いた`this`は`undefined`となります。

```
<script type="module"> // 実行コンテキストは"Module"
console.log(this); // => undefined </script> 
```

このように、トップレベルのスコープの`this`は実行コンテキストによって`undefined`となる場合があります。

単純にグローバルオブジェクトを参照したい場合は、`this`ではなく`globalThis`を使います。`globalThis`は実行環境のグローバルオブジェクトを参照するためにES2020で導入されました。

実行環境のグローバルオブジェクトは、ブラウザでは`window`、Node.jsでは`global`のように名前が異なります。そのため同じコードで、異なるグローバルオブジェクトを参照するには、コード上で分岐する必要がありました。ES2020ではこの問題を解決するために、実行環境のグローバルオブジェクトを参照する`globalThis`が導入されました。

```
// ブラウザでは`window`オブジェクト、Node.jsでは`global`オブジェクトを参照する
console.log(globalThis); 
```

## [](#function-and-method-this)*関数とメソッドにおける`this`*

***関数**を定義する方法として、`function`キーワードによる関数宣言と関数式、Arrow Functionなどがあります。`this`が参照先を決めるルールは、Arrow Functionとそれ以外の関数定義の方法で異なります。

そのため、まずは関数定義の種類について振り返ってから、それぞれの`this`について見ていきます。

### [](#type-of-function)*関数の種類*

*「関数と宣言」の章で詳しく紹介していますが、関数の定義方法と呼び出し方について改めて振り返ってみましょう。**関数**を定義する場合には、次の3つの方法を利用します。

```
// `function`キーワードからはじめる関数宣言
function fn1() {}
// `function`を式として扱う関数式
const fn2 = function() {};
// Arrow Functionを使った関数式
const fn3 = () => {}; 
```

それぞれ定義した関数は`関数名()`と書くことで呼び出せます。

```
// 関数宣言
function fn() {}
// 関数呼び出し
fn(); 
```

### [](#type-of-method)*メソッドの種類*

*JavaScriptではオブジェクトのプロパティが関数である場合にそれを**メソッド**と呼びます。一般的にはメソッドも含めたものを**関数**と言い、関数宣言などとプロパティである関数を区別する場合に**メソッド**と呼びます。

メソッドを定義する場合には、オブジェクトのプロパティに関数式を定義するだけです。

```
const obj = {
    // `function`キーワードを使ったメソッド
    method1: function() {
    },
    // Arrow Functionを使ったメソッド
    method2: () => {
    }
}; 
```

これに加えてメソッドには短縮記法があります。オブジェクトリテラルの中で `メソッド名(){ /*メソッドの処理*/ }`と書くことで、メソッドを定義できます。

```
const obj = {
    // メソッドの短縮記法で定義したメソッド
    method() {
    }
}; 
```

これらのメソッドは、`オブジェクト名.メソッド名()`と書くことで呼び出せます。

```
const obj = {
    // メソッドの定義
    method() {
    }
};
// メソッド呼び出し
obj.method(); 
```

関数定義とメソッドの定義についてまとめると、次のようになります。

| 名前 | 関数 | メソッド |
| --- | --- | --- |
| 関数宣言(`function fn(){}`) | ✔ | x |
| 関数式(`const fn = function(){}`) | ✔ | ✔ |
| Arrow Function(`const fn = () => {}`) | ✔ | ✔ |
| メソッドの短縮記法(`const obj = { method(){} }`) | x | ✔ |

最初に書いたように`this`の挙動は、Arrow Functionの関数定義とそれ以外（`function`キーワードやメソッドの短縮記法）の関数定義で異なります。そのため、まずは**Arrow Function 以外**の関数やメソッドにおける`this`を見ていきます。

## [](#function-without-arrow-function-this)*非箭头函数中的`this`*

*非箭头函数（包括方法）中的`this`在执行时确定其值。换句话说，`this`就像是一个传递给函数的隐含参数，其传递的值在函数执行时确定。

下面的代码是假设性的。函数中写出的`this`将引用函数调用源暗含传递的值。这个规则适用于非箭头函数或方法，箭头函数定义的函数或方法则遵循不同的规则。

```
// 疑似的な`this`の値の仕組み
// 関数は引数として暗黙的に`this`の値を受け取るイメージ
function fn(暗黙的に渡されるthisの値, 仮引数) {
    console.log(this); // => 暗黙的に渡されるthisの値
}
// 暗黙的に`this`の値を引数として渡しているイメージ
fn(暗黙的に渡すthisの値, 引数); 
```

函数中的`this`的基本引用点是**基础对象**。基础对象是指“调用方法时，方法点操作符或方括号操作符左边的对象”。如果没有基础对象，`this`将是`undefined`。

例如，当像`fn()`这样调用函数时，由于这个`fn`函数调用没有基础对象，所以`this`是`undefined`。另一方面，当像`obj.method()`这样调用方法时，这个`obj.method`方法调用没有基础对象是`obj`对象，`this`是`obj`。

```
// `fn`関数はメソッドではないのでベースオブジェクトはない
fn();
// `obj.method`メソッドのベースオブジェクトは`obj`
obj.method();
// `obj1.obj2.method`メソッドのベースオブジェクトは`obj2`
// ドット演算子、ブラケット演算子どちらも結果は同じ
obj1.obj2.method();
obj1["obj2"]["method"](); 
```

`this`的值不是在函数定义时确定的，而是在调用时确定的。这是在后面将要介绍的“`this`引起的问题模式”中详细介绍的内容。在非箭头函数中，仅从函数定义来看无法确定`this`的值。

### [](#function-declaration-expression-this)*函数声明或函数表达式中的`this`*

*首先，让我们看看函数声明或函数表达式的情况。

在下面的例子中，通过函数声明定义了函数`fn1`，通过函数表达式定义了函数`fn2`，并在每个函数内返回`this`。定义的每个函数都像`fn1()`和`fn2()`一样作为普通函数调用。这时，由于没有基础对象，`this`是`undefined`。

```
"use strict";
function fn1() {
    return this;
}
const fn2 = function() {
    return this;
};
// 関数の中の`this`が参照する値は呼び出し方によって決まる
// `fn1`と`fn2`どちらもただの関数として呼び出している
// メソッドとして呼び出していないためベースオブジェクトはない
// ベースオブジェクトがない場合、`this`は`undefined`となる
console.log(fn1()); // => undefined
console.log(fn2()); // => undefined 
```

这同样适用于在函数中定义并调用函数的情况。

```
"use strict";
function outer() {
    console.log(this); // => undefined
    function inner() {
        console.log(this); // => undefined
    }
    // `inner`関数呼び出しのベースオブジェクトはない
    inner();
}
// `outer`関数呼び出しのベースオブジェクトはない
outer(); 
```

本书中的代码如果没有注释，则视为 strict mode。但代码示例中明确地写入了`"use strict";`来表示 strict mode（详细信息请参考「JavaScript 是什么」中的 strict mode）。这是因为，在非 strict mode 的情况下，如果`this`是`undefined`，则`this`会被转换为引用全局对象的值。

strict mode 是为了防止这种难以预料的行为而引入的。然而，strict mode 之外的其他函数中的`this`将变为`undefined`，因此没有用途。因此，不需要在方法之外使用`this`。

### [](#method-this)*方法调用中的`this`*

*接下来，让我们看看方法的情况。在方法的情况下，该方法属于某个对象。这是因为，在 JavaScript 中，将作为对象属性指定的函数称为方法。

在下面的例子中，`method1`和`method2`分别作为方法被调用。这时，每个基础对象是`obj`，`this`是`obj`。

```
const obj = {
    // 関数式をプロパティの値にしたメソッド
    method1: function() {
        return this;
    },
    // 短縮記法で定義したメソッド
    method2() {
        return this;
    }
};
// メソッド呼び出しの場合、それぞれの`this`はベースオブジェクト(`obj`)を参照する
// メソッド呼び出しの`.`の左にあるオブジェクトがベースオブジェクト
console.log(obj.method1()); // => obj
console.log(obj.method2()); // => obj 
```

这样就可以利用`this`来引用同一个对象中的其他属性。

```
const person = {
    fullName: "Brendan Eich",
    sayName: function() {
        // `person.fullName`と書いているのと同じ
        return this.fullName;
    }
};
// `person.fullName`を出力する
console.log(person.sayName()); // => "Brendan Eich" 
```

这样，可以通过`this.属性名`代替`对象名.属性名`来引用方法所属对象属性的属性。

对象可以多层嵌套，但`this`引用基础对象的规则是相同的。

看看下面的代码，可以发现嵌套的对象中，方法内的`this`引用的是基础对象`obj3`。这时的基础对象不是通过点连接的最左边的`obj1`，而是从方法的角度看左边的`obj3`。

```
const obj1 = {
    obj2: {
        obj3: {
            method() {
                return this;
            }
        }
    }
};
// `obj1.obj2.obj3.method`メソッドの`this`は`obj3`を参照
console.log(obj1.obj2.obj3.method() === obj1.obj2.obj3); // => true 
```

## [](#this-problem)*`this`引起的问题模式*

*我们已经了解了`this`可以引用函数（包括方法）的调用基础对象。`this`可以作为直接写入所属对象的替代品来使用，但`this`也有各种问题。

这个问题的原因在于`this`引用的值在函数调用时确定。下面将详细介绍导致问题的两种主要模式的示例和各自的解决方案。

### [](#assign-this-function)*问题：将包含`this`的函数代入变量时*

*在 JavaScript 中，作为方法定义的函数有时会被当作普通函数调用。这是因为方法是将函数作为值持有的属性，属性可以被重新赋值给变量。

因此，作为方法定义的函数也可以被代入到另一个变量中，并以普通函数的方式调用。在这种情况下，即使作为方法定义的函数，由于在执行时被视为普通函数，基础对象也会改变。这是`this`在定义时而不是在执行时确定的性质。

具体来说，看看`this`在执行时变化的例子。在下面的例子中，将`person.sayName`方法代入变量`say`后执行。这时，`say`函数（`sayName`方法的引用）没有基础对象。因此，`this`是`undefined`，`undefined.fullName`无法引用并抛出异常。

```
"use strict";
const person = {
    fullName: "Brendan Eich",
    sayName: function() {
        // `this`は呼び出し元によって異なる
        return this.fullName;
    }
};
// `sayName`メソッドは`person`オブジェクトに所属する
// `this`は`person`オブジェクトとなる
console.log(person.sayName()); // => "Brendan Eich"
// `person.sayName`を`say`変数に代入する
const say = person.sayName;
// 代入したメソッドを関数として呼ぶ
// この`say`関数はどのオブジェクトにも所属していない
// `this`はundefinedとなるため例外を投げる
say(); // => TypeError: Cannot read property 'fullName' of undefined 
```

最终，执行的代码与下面的代码相同。在下面的代码中，尝试引用`undefined.fullName`导致了异常。

```
"use strict";
// const say = person.sayName; は次のようなイメージ
const say = function() {
    return this.fullName;
};
// `this`は`undefined`となるため例外を投げる
say(); // => TypeError: Cannot read property 'fullName' of undefined 
```

这样，除了箭头函数之外的函数中，`this`不是在定义时确定的，而是在执行时确定的。因此，如果函数包含`this`，并且没有按预期调用，就会出现错误的结果。

解决这个问题有两种主要方法。

其中一种方法是，将作为方法定义的函数作为方法调用。如果不将方法作为普通函数调用，就不会出现这个问题。

另一种方法是，通过指定`this`值来调用函数的方法来执行函数。

#### [](#call-apply-bind)*解决方法：call、apply、bind 方法*

*还有一种方法可以显式指定函数或方法的`this`并执行函数。 `Function`（函数对象）提供了`call`、`apply`、`bind`等方法，用于显式指定`this`并执行函数。*

`call`方法将第一个参数指定为要作为`this`的值，并将剩余的参数指定为要传递给调用函数的参数。可以说`call`方法是一个可以显式传递`this`值的方法。

```
関数.call(thisの値, ...関数の引数); 
```

在下面的示例中，我们在指定`person`对象为`this`的情况下调用了`say`函数。 `call`方法的第二个参数将传递给`say`函数的形参`message`。

```
"use strict";
function say(message) {
    return `${message} ${this.fullName}！`;
}
const person = {
    fullName: "Brendan Eich"
};
// `this`を`person`にして`say`関数を呼びだす
console.log(say.call(person, "こんにちは")); // => "こんにちは Brendan Eich！"
// `say`関数をそのまま呼び出すと`this`は`undefined`となるため例外が発生
say("こんにちは"); // => TypeError: Cannot read property 'fullName' of undefined 
```

`apply`方法将第一个参数指定为要作为`this`的值，并将第二个参数作为函数的参数传递为数组。

```
関数.apply(thisの値, [関数の引数 1, 関数の引数 2]); 
```

在下面的示例中，我们在指定`person`对象为`this`的情况下调用了`say`函数。 `apply`方法的第二个参数指定的数组会自动展开并传递给`say`函数的形参`message`。

```
"use strict";
function say(message) {
    return `${message} ${this.fullName}！`;
}
const person = {
    fullName: "Brendan Eich"
};
// `this`を`person`にして`say`関数を呼びだす
// callとは異なり引数を配列として渡す
console.log(say.apply(person, ["こんにちは"])); // => "こんにちは Brendan Eich！"
// `say`関数をそのまま呼び出すと`this`は`undefined`となるため例外が発生
say("こんにちは"); // => TypeError: Cannot read property 'fullName' of undefined 
```

`call`方法和`apply`方法的区别仅在于传递参数给函数的方式不同。 此外，如果不需要`this`的值，通常会传递`null`给这两种方法。

```
function add(x, y) {
    return x + y;
}
// `this`が不要な場合は、nullを渡す
console.log(add.call(null, 1, 2)); // => 3
console.log(add.apply(null, [1, 2])); // => 3 
```

最后让我们谈谈`bind`方法。 正如其名称所示，`bind`会创建一个绑定了`this`值的新函数。

```
関数.bind(thisの値, ...関数の引数); // => thisや引数がbindされた関数 
```

在下面的示例中，我们创建了一个包装了将`this`绑定到`person`对象的`say`函数的函数。 通过在`bind`方法的第二个参数之后传递值，可以绑定函数的参数。

```
function say(message) {
    return `${message} ${this.fullName}！`;
}
const person = {
    fullName: "Brendan Eich"
};
// `this`を`person`に束縛した`say`関数をラップした関数を作る
const sayPerson = say.bind(person, "こんにちは");
console.log(sayPerson()); // => "こんにちは Brendan Eich！" 
```

将这个`bind`方法表示为普通函数如下所示。 可以看出，`bind`是一个创建绑定了`this`和参数的函数的方法。

```
function say(message) {
    return `${message} ${this.fullName}！`;
}
const person = {
    fullName: "Brendan Eich"
};
// `this`を`person`に束縛した`say`関数をラップした関数を作る
//  say.bind(person, "こんにちは"); は次のようなラップ関数を作る
const sayPerson = () => {
    return say.call(person, "こんにちは");
};
console.log(sayPerson()); // => "こんにちは Brendan Eich！" 
```

通过使用`call`、`apply`、`bind`方法，可以在明确指定`this`的情况下调用函数。 但是，每次调用函数时都使用这些方法会导致需要为调用函数而调用函数，这样会变得繁琐。 因此，基本上最好的方法是“将作为方法定义的函数作为方法调用”。 在其中，如果确实需要固定`this`，则可以使用`call`、`apply`、`bind`方法。

### [](#callback-and-this)*问题：回调函数和`this`*

*在回调函数中引用`this`可能会导致问题。 这个问题在处理 Array 的`map`方法等回调函数时很容易发生。*

让我们看一个具体的例子，说明回调函数中的`this`是一个问题。 在下面的代码中，我们使用`map`方法在`prefixArray`方法中。 在这种情况下，`map`方法的回调函数中，试图引用`this`以引用`Prefixer`对象。

然而，在这个回调函数中，`this`是`undefined`，因此无法访问`undefined.prefix`，导致`TypeError`异常。

```
"use strict";
// strict modeを明示しているのは、thisがグローバルオブジェクトに暗黙的に変換されるのを防止するため
const Prefixer = {
    prefix: "pre",
    /**
     * `strings`配列の各要素にprefixをつける
     */
    prefixArray(strings) {
        return strings.map(function(str) {
            // コールバック関数における`this`は`undefined`となる(strict mode)
            // そのため`this.prefix`は`undefined.prefix`となり例外が発生する
            return this.prefix + "-" + str;
        });
    }
};
// `prefixArray`メソッドにおける`this`は`Prefixer`
Prefixer.prefixArray(["a", "b", "c"]); // => TypeError: Cannot read property 'prefix' of undefined 
```

让我们看看为什么回调函数中的`this`会变为`undefined`。 请注意，Array 的`map`方法将一个在场景中定义的匿名函数作为回调函数传递。

```
// ...
    prefixArray(strings) {
        // 無名関数をコールバック関数として渡している
        return strings.map(function(str) {
            return this.prefix + "-" + str;
        });
    }
// ... 
```

在这种情况下，传递给 Array 的`map`方法的回调函数将像`callback()`一样被调用为普通函数。 换句话说，在调用回调函数时，该函数没有基础对象。 因此，`callback`函数的`this`将是`undefined`。

在前面的示例中，我们直接将匿名函数作为回调函数传递给方法，但是将其存储在`callback`变量中然后传递也会得到相同的结果。

```
"use strict";
// strict modeを明示しているのは、thisがグローバルオブジェクトに暗黙的に変換されるのを防止するため
const Prefixer = {
    prefix: "pre",
    prefixArray(strings) {
        // コールバック関数は`callback()`のように呼び出される
        // そのためコールバック関数における`this`は`undefined`となる(strict mode)
        const callback = function(str) {
            return this.prefix + "-" + str;
        };
        return strings.map(callback);
    }
};
// `prefixArray`メソッドにおける`this`は`Prefixer`
Prefixer.prefixArray(["a", "b", "c"]); // => TypeError: Cannot read property 'prefix' of undefined 
```

#### [](#substitute-this)*解决方法：将`this`赋值给临时变量*

*回调函数内的`this`引用会发生变化的问题的解决方法是将`this`赋值给另一个变量，以保持其引用。*

`this`在函数调用者处会发生变化，其引用的是调用者的基础对象。 在调用`prefixArray`方法时，`this`是`Prefixer`对象。 但是，由于回调函数被重新作为函数调用，`this`变为`undefined`就成了问题。

因此，通过将第一个`prefixArray`方法调用中的`this`引用保存为临时变量，可以避免这个问题。 在下面的代码中，我们将`prefixArray`方法的`this`保存在`that`变量中。 通过在回调函数中引用`that`变量而不是`this`，可以使回调函数引用与`prefixArray`方法调用相同的`this`。

```
"use strict";
const Prefixer = {
    prefix: "pre",
    prefixArray(strings) {
        // `that`は`prefixArray`メソッド呼び出しにおける`this`となる
        // つまり`that`は`Prefixer`オブジェクトを参照する
        const that = this;
        return strings.map(function(str) {
            // `this`ではなく`that`を参照する
            return that.prefix + "-" + str;
        });
    }
};
// `prefixArray`メソッドにおける`this`は`Prefixer`
const prefixedStrings = Prefixer.prefixArray(["a", "b", "c"]);
console.log(prefixedStrings); // => ["pre-a", "pre-b", "pre-c"] 
```

当然，您也可以使用`Function`的`call`方法等明确传递`this`并调用函数。 此外，像 Array 的`map`方法等具有将`this`作为参数传递的机制。 因此，通过在第二个参数中传递`this`值，也可以解决这个问题。

```
"use strict";
const Prefixer = {
    prefix: "pre",
    prefixArray(strings) {
        // Arrayの`map`メソッドは第二引数に`this`となる値を渡せる
        return strings.map(function(str) {
            // `this`が第二引数の値と同じになる
            // つまり`prefixArray`メソッドと同じ`this`となる
            return this.prefix + "-" + str;
        }, this);
    }
};
// `prefixArray`メソッドにおける`this`は`Prefixer`
const prefixedStrings = Prefixer.prefixArray(["a", "b", "c"]);
console.log(prefixedStrings); // => ["pre-a", "pre-b", "pre-c"] 
```

但是，这些解决方法需要意识到回调函数中`this`的变化。 问题的根源在于方法调用及其中的回调函数中的`this`发生了变化。 在 ES2015 中，引入了箭头函数作为一种不改变`this`的回调函数定义方法。

#### [](#arrow-function-callback)*対処法: Arrow Functionでコールバック関数を扱う*

*通常の関数やメソッドは呼び出し時に暗黙的に`this`の値を受け取り、関数内の`this`はその値を参照します。 一方、Arrow Functionはこの暗黙的な`this`の値を受け取りません。 そのためArrow Function 内の`this`は、スコープチェーンの仕組みと同様に外側の関数（この場合は`prefixArray`メソッド）を探索します。 これにより、Arrow Functionで定義したコールバック関数は呼び出し方には関係なく、常に外側の関数の`this`をそのまま利用します。

Arrow Functionを使うことで、先ほどのコードは次のように書けます。

```
"use strict";
const Prefixer = {
    prefix: "pre",
    prefixArray(strings) {
        return strings.map((str) => {
            // Arrow Function 自体は`this`を持たない
            // `this`は外側の`prefixArray`関数が持つ`this`を参照する
            // そのため`this.prefix`は"pre"となる
            return this.prefix + "-" + str;
        });
    }
};
// このとき、`prefixArray`のベースオブジェクトは`Prefixer`となる
// つまり、`prefixArray`メソッド内の`this`は`Prefixer`を参照する
const prefixedStrings = Prefixer.prefixArray(["a", "b", "c"]);
console.log(prefixedStrings); // => ["pre-a", "pre-b", "pre-c"] 
```

このように、Arrow Functionでのコールバック関数における`this`は簡潔です。 コールバック関数内での`this`の対処法として`this`を代入する方法を紹介しましたが、 ES2015からはArrow Functionを使うのがもっとも簡潔です。

このArrow Functionと`this`の関係についてより詳しく見ていきます。

## [](#arrow-function-this)*Arrow Functionと`this`*

*Arrow Functionで定義された関数やメソッドにおける`this`がどの値を参照するかは関数の定義時（静的）に決まります。 一方、Arrow Functionではない関数においては、`this`は呼び出し元に依存するため関数の実行時（動的）に決まります。

Arrow Functionとそれ以外の関数で大きく違うことは、Arrow Functionは`this`を暗黙的な引数として受けつけないということです。 そのため、Arrow Function 内には`this`が定義されていません。このときの`this`は外側のスコープ（関数）の`this`を参照します。

这与变量中的作用域链机制相同，如果该作用域中没有定义`this`，则会向外层作用域探索。因此，在 Arrow Function 中的`this`引用始终会向外层作用域（函数）探索`this`的定义（详细信息请参考作用域链）。另外，`this`是 ECMAScript 的关键字，因此用户不能定义名为`this`的变量。

```
// thisはキーワードであるため、ユーザーは`this`という名前の変数を定義できない
const this = "thisは読み取り専用"; // => SyntaxError: Unexpected token this 
```

因此，`this`的引用与普通变量一样，是静态（定义时）确定的（详细信息请参考静态作用域）。也就是说，Arrow Function 中的`this`是“Arrow Function 自身外部的最接近的函数的`this`值”。

让我们来具体看看 Arrow Function 中`this`的行为。

首先来看一下函数表达式中的 Arrow Function。

在下面的例子中，我们在函数表达式中定义了 Arrow Function，并在其中输出了`this`。这时，由于`fn`的外部没有函数，所以没有符合“自身外部的最接近的函数”的条件。在这种情况下，`this`的值与顶层定义的`this`相同。

```
// Arrow Functionで定義した関数
const fn = () => {
    // この関数の外側には関数は存在しない
    // トップレベルの`this`と同じ値
    return this;
};
console.log(fn() === this); // => true 
```

已经介绍了顶层定义的`this`的值会因执行上下文而异。当执行上下文为"Script"时，`this`的值将变为全局对象；当执行上下文为"Module"时，`this`的值变为`undefined`。

按照下一个例子，如果定义了一个通常的函数来包围 Arrow Function，那么箭头函数中的`this`将变成“自身外部的更近的函数的`this`值”，这与之前的情况相同。

```
"use strict";
function outer() {
    // Arrow Functionで定義した関数を返す
    return () => {
        // この関数の外側には`outer`関数が存在する
        // `outer`関数に`this`を書いた場合と同じ
        return this;
    };
}
// `outer`関数の返り値はArrow Functionにて定義された関数
const innerArrowFunction = outer();
console.log(innerArrowFunction()); // => undefined 
```

也就是说，这个 Arrow Function 中的`this`与在`outer`函数中引用`this`时的值相同。

```
"use strict";
function outer() {
    // `outer`関数直下の`this`
    const that = this;
    // Arrow Functionで定義した関数を返す
    return () => {
        // Arrow Function 自身は`this`を持たない
        // `outer`関数に`this`を書いた場合と同じ
        return that;
    };
}
// `outer()`と呼び出したときの`this`は`undefined`(strict mode)
const innerArrowFunction = outer();
console.log(innerArrowFunction()); // => undefined 
```

### [](#method-callback-arrow-function)*方法和回调函数与 Arrow Function*

*在方法内使用回调函数是更有效地使用 Arrow Function 的模式。使用`function`关键字定义回调函数时，需要考虑回调函数的调用方式，因为使用`function`关键字定义的函数中的`this`值会根据调用方式而变化。

从回调函数的角度来看，由于调用方式的不同，不能使用`this`。因此，我们采取了在外部作用域中将`this`临时存储到变量中的回避方法。

```
// `callback`関数を受け取り呼び出す関数
const callCallback = (callback) => {
    // `callback`を呼び出す実装
};

const obj = {
    method() {
        callCallback(function() {
            // ここでの `this` は`callCallback`の実装に依存する
            // `callback()`のように単純に呼び出されるなら`this`は`undefined`になる
            // Functionの`call`メソッドなどを使って特定のオブジェクトを指定するかもしれない
            // この問題を回避するために`const that = this`のような一時変数を使う
        });
    }
}; 
```

另一方面，如果使用 Arrow Function 定义回调函数，则会引用外部函数的一个`this`。在这种情况下，由 Arrow Function 定义的回调函数中的`this`不会因调用方式而变化。因此，不需要使用临时变量等方法来回避。

```
// `callback`関数を受け取り呼び出す関数
const callCallback = (callback) => {
    // `callback`を呼び出す実装
};

const obj = {
    method() {
        callCallback(() => {
            // ここでの`this`は1つ外側の関数における`this`と同じ
        });
    }
}; 
```

这个 Arrow Function 中的`this`不受调用方式的影响。也就是说，可以不考虑回调函数的调用方式来处理`this`。

```
const Prefixer = {
    prefix: "pre",
    prefixArray(strings) {
        return strings.map((str) => {
            // `Prefixer.prefixArray()` と呼び出されたとき
            // `this`は常に`Prefixer`を参照する
            return this.prefix + "-" + str;
        });
    }
};
const prefixedStrings = Prefixer.prefixArray(["a", "b", "c"]);
console.log(prefixedStrings); // => ["pre-a", "pre-b", "pre-c"] 
```

### [](#not-bind-arrow-function)*Arrow Function 不能绑定`this`*

*在 Arrow Function 定义的函数中，使用`call`、`apply`、`bind`指定`this`的行为会被忽略。这是因为 Arrow Function 没有`this`。

按照下面的方式，即使使用`call`指定了 Arrow Function 定义的函数中的`this`，也可以发现`this`的引用并没有改变。同样，使用`apply`或`bind`方法时，`this`的引用也不会改变。

```
const fn = () => {
    return this;
};
// Scriptコンテキストの場合、スクリプト直下のArrow Functionの`this`はグローバルオブジェクト
console.log(fn()); // グローバルオブジェクト
// callで`this`を`{}`にしようとしても、`this`は変わらない
console.log(fn.call({})); // グローバルオブジェクト 
```

正如之前所述，使用`function`关键字定义的函数在调用时，会将基础对象作为`this`的值以隐式参数的形式传递。另一方面，Arrow Function 的函数在调用时不会接收`this`，`this`的引用在定义时是静态确定的。

此外，`this`只在定义在箭头函数中的函数中保持不变，箭头函数中的`this`引用的是“最接近其外部作用域的函数的`this`值”，可以通过`call`方法进行更改。

```
const obj = {
    method() {
        const arrowFunction = () => {
            return this;
        };
        return arrowFunction();
    }
};
// 通常の`this`は`obj.method`の`this`と同じ
console.log(obj.method()); // => obj
// `obj.method`の`this`を変更すれば、Arrow Functionの`this`も変更される
console.log(obj.method.call("THAT")); // => "THAT" 
```

## [](#conclusion)*总结*

*介绍了`this`是一个根据情况引用不同值的关键字。以下是对`this`评估结果的总结。

| 执行上下文 | 严格模式 | 代码 | `this`的评估结果 |
| --- | --- | --- | --- |
| Script | ＊ | `this` | `globalThis` |
| Script | ＊ | `const fn = () => this` | `globalThis` |
| Script | NO | `const fn = function(){ return this; }` | `globalThis` |
| Script | YES | `const fn = function(){ return this; }` | `undefined` |
| Script | ＊ | `const obj = { method: () => { return this; } }` | `globalThis` |
| Module | YES | `this` | `undefined` |
| Module | YES | `const fn = () => this` | `undefined` |
| Module | YES | `const fn = function(){ return this; }` | `undefined` |
| Module | YES | `const obj = { method: () => { return this; } }` | `undefined` |
| ＊ | ＊ | `const obj = { method(){ return this; } }` | `obj` |
| ＊ | ＊ | `const obj = { method: function(){ return this; } }` | `obj` |

> ＊表示在任何情况下都不会影响`this`的评估结果。

实际在浏览器中运行的结果可以在[JavaScript 中的`this`值是什么](https://azu.github.io/what-is-this/)网站上查看。

`this` 是在 JavaScript 中引入的面向对象编程的概念。除了方法之外，`this` 也可以在其他情况下进行评估，但由于执行上下文和严格模式等因素，结果会有所不同，导致混淆。因此，在非方法函数中不应使用 `this`。¹

此外，在方法中，`this` 根据调用方式而有不同的值，介绍了由此产生的问题和解决方法。通过使用箭头函数可以更清晰地解决回调函数中的 `this` 问题。这背后的原因是通过箭头函数定义的函数不具有 `this` 属性。

> ¹. ES2015 的规范编辑者艾伦·沃夫斯-布洛克先生也指出，在普通函数中不应该使用 `this`。[twitter.com/awbjs/status/938272440085446657](https://twitter.com/awbjs/status/938272440085446657); ↩*******************
