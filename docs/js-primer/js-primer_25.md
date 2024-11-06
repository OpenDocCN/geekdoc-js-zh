# 関数とthis

> 原文：[`jsprimer.net/basic/function-this/`](https://jsprimer.net/basic/function-this/)

この章では`this`という特殊な動作をするキーワードについて見ていきます。 基本的にはメソッドの中で利用しますが、`this`は読み取り専用のグローバル変数のようなものでどこにでも書けます。 加えて、`this`の参照先（評価結果）は条件によって異なります。

`this`の参照先は主に次の条件によって変化します。

+   実行コンテキストにおける`this`

+   コンストラクタにおける`this`

+   関数とメソッドにおける`this`

+   Arrow Functionにおける`this`

コンストラクタにおける`this`は、次の章である「クラス」で扱います。 この章ではさまざまな条件での`this`について扱いますが、`this`が実際に使われるのはメソッドにおいてです。 そのため、あらゆる条件下での`this`の動きをすべて覚える必要はありません。

この章では、さまざまな条件下で変わる`this`の参照先と関数やArrow Functionとの関係を見ていきます。 また、実際にどのような状況で問題が発生するかを知り、`this`の動きを予測可能にするにはどのようにするかを見ていきます。

## [](#execution-context-this)*実行コンテキストと`this`*

*最初に「JavaScriptとは」の章において、JavaScriptには実行コンテキストとして"Script"と"Module"があるという話をしました。 どの実行コンテキストでJavaScriptのコードを評価するかは、��行環境によってやり方が異なります。 この章では、ブラウザの`script`要素と`type`属性を使い、それぞれの実行コンテキストを明示しながら`this`の動きを見ていきます。

トップレベル（もっとも外側のスコープ）にある`this`は、実行コンテキストによって値が異なります。 実行コンテキストの違いは意識しにくい部分であり、トップレベルで`this`を使うと混乱を生むことになります。 そのため、コードのトップレベルにおいては`this`を使うべきではありませんが、それぞれの実行コンテキストにおける動作を紹介します。

### [](#script-this)*スクリプトにおける`this`*

*実行コンテキストが"Script"である場合、トップレベルのスコープに書かれた`this`はグローバルオブジェクトを参照します。 グローバルオブジェクトは、実行環境ごとに異なるものが定義されています。 ブラウザのグローバルオブジェクトは`window`オブジェクト、Node.jsのグローバルオブジェクトは`global`オブジェクトとなります。

ブラウザでは、`script`要素の`type`属性を指定していない場合は、実行コンテキストが"Script"として実行されます。 この`script`要素の直下に書いた`this`はグローバルオブジェクトである`window`オブジェクトとなります。

```
<script> // 実行コンテキストは"Script"
console.log(this); // => window </script> 
```

### [](#module-this)*モジュールにおける`this`*

*実行コンテキストが"Module"である場合、そのトップレベルのスコープに書かれた`this`は常に`undefined`となります。

ブラウザで、`script`要素に`type="module"`属性がついた場合は、実行コンテキストが"Module"として実行されます。 この`script`要素の直下に書いた`this`は`undefined`となります。

```
<script type="module"> // 実行コンテキストは"Module"
console.log(this); // => undefined </script> 
```

このように、トップレベルのスコープの`this`は実行コンテキストによって`undefined`となる場合があります。

単純にグローバルオブジェクトを参照したい場合は、`this`ではなく`globalThis`を使います。 `globalThis`は実行環境のグローバルオブジェクトを参照するためにES2020で導入されました。

実行環境のグローバルオブジェクトは、ブラウザでは`window`、Node.jsでは`global`のように名前が異なります。 そのため同じコードで、異なるグローバルオブジェクトを参照するには、コード上で分岐する必要がありました。 ES2020ではこの問題を解決するために、実行環境のグローバルオブジェクトを参照する`globalThis`が導入されました。

```
// ブラウザでは`window`オブジェクト、Node.jsでは`global`オブジェクトを参照する
console.log(globalThis); 
```

## [](#function-and-method-this)*関数とメソッドにおける`this`*

***関数**を定義する方法として、`function`キーワードによる関数宣言と関数式、Arrow Functionなどがあります。 `this`が参照先を決めるルールは、Arrow Functionとそれ以外の関数定義の方法で異なります。

そのため、まずは関数定義の種類について振り返ってから、それぞれの`this`について見ていきます。

### [](#type-of-function)*関数の種類*

*「関数と宣言」の章で詳しく紹介していますが、関数の定義方法と呼び出し方について改めて振り返ってみましょう。 **関数**を定義する場合には、次の3つの方法を利用します。

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

*JavaScriptではオブジェクトのプロパティが関数である場合にそれを**メソッド**と呼びます。 一般的にはメソッドも含めたものを**関数**と言い、関数宣言などとプロパティである関数を区別する場合に**メソッド**と呼びます。

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

これに加えてメソッドには短縮記法があります。 オブジェクトリテラルの中で `メソッド名(){ /*メソッドの処理*/ }`と書くことで、メソッドを定義できます。

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

最初に書いたように`this`の挙動は、Arrow Functionの関数定義とそれ以外（`function`キーワードやメソッドの短縮記法）の関数定義で異なります。 そのため、まずは**Arrow Function 以外**の関数やメソッドにおける`this`を見ていきます。

## [](#function-without-arrow-function-this)*Arrow Function 以外の関数における`this`*

*Arrow Function 以外の関数（メソッドも含む）における`this`は、実行時に決まる値となります。 言い方を変えると`this`は関数に渡される暗黙的な引数のようなもので、その渡される値は関数を実行するときに決まります。

次のコードは疑似的なものです。 関数の中に書かれた`this`は、関数の呼び出し元から暗黙的に渡される値を参照することになります。 このルールはArrow Function 以外の関数やメソッドで共通した仕組みとなります。Arrow Functionで定義した関数やメソッドはこのルールとは別の仕組みとなります。

```
// 疑似的な`this`の値の仕組み
// 関数は引数として暗黙的に`this`の値を受け取るイメージ
function fn(暗黙的に渡されるthisの値, 仮引数) {
    console.log(this); // => 暗黙的に渡されるthisの値
}
// 暗黙的に`this`の値を引数として渡しているイメージ
fn(暗黙的に渡すthisの値, 引数); 
```

関数における`this`の基本的な参照先（暗黙的に関数に渡す`this`の値）は**ベースオブジェクト**となります。 ベースオブジェクトとは「メソッドを呼ぶ際に、そのメソッドのドット演算子またはブラケット演算子のひとつ左にあるオブジェクト」のことを言います。 ベースオブジェクトがない場合の`this`は`undefined`となります。

たとえば、`fn()`のように関数を呼び出したとき、この`fn`関数呼び出しのベースオブジェクトはないため、`this`は`undefined`となります。 一方、`obj.method()`のようにメソッドを呼び出したとき、この`obj.method`メソッド呼び出しのベースオブジェクトは`obj`オブジェクトとなり、`this`は`obj`となります。

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

`this`は関数の定義ではなく呼び出し方で参照する値が異なります。これは、後述する「`this`が問題となるパターン」で詳しく紹介します。 Arrow Function 以外の関数では、関数の定義だけを見て`this`の値が何かということは決定できない点に注意が必要です。

### [](#function-declaration-expression-this)*関数宣言や関数式における`this`*

*まずは、関数宣言や関数式の場合を見ていきます。

次の例では、関数宣言で関数`fn1`、関数式で関数`fn2`を定義し、それぞれの関数内で`this`を返します。 定義したそれぞれの関数を`fn1()`と`fn2()`のようにただの関数として呼び出しています。 このとき、ベースオブジェクトはないため、`this`は`undefined`となります。

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

これは、関数の中に関数を定義して呼び出す場合も同じです。

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

この書籍では注釈がないコードはstrict modeとして扱いますが、コード例に`"use strict";`と改めてstrict modeを明示しています（詳細は「JavaScriptとは」のstrict modeを参照）。 なぜなら、strict modeではない状況で`this`が`undefined`の場合は、`this`がグローバルオブジェクトを参照するように変換される問題があるためです。

strict modeは、このような意図しにくい動作を防止するために導入されています。 しかしながら、strict modeのメソッド以外の関数における`this`は`undefined`となるため使い道がありません。 そのため、メソッド以外で`this`を使う必要はありません。

### [](#method-this)*メソッド呼び出しにおける`this`*

*次に、メソッドの場合を見ていきます。 メソッドの場合は、そのメソッドが何かしらのオブジェクトに所属しています。 なぜなら、JavaScriptではオブジェクトのプロパティとして指定される関数のことをメソッドと呼ぶためです。

次の例では`method1`と`method2`はそれぞれメソッドとして呼び出されています。 このとき、それぞれのベースオブジェクトは`obj`となり、`this`は`obj`となります。

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

これを利用すれば、メソッドの中から同じオブジェクトに所属する別のプロパティを`this`で参照できます。

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

このようにメソッドが所属するオブジェクトのプロパティを、`オブジェクト名.プロパティ名`の代わりに`this.プロパティ名`で参照できます。

オブジェクトは何重にもネストできますが、`this`はベースオブジェクトを参照するというルールは同じです。

次のコードを見てみると、ネストしたオブジェクトにおいてメソッド内の`this`がベースオブジェクトである`obj3`を参照していることがわかります。 このときのベースオブジェクトはドットでつないだ一番左の`obj1`ではなく、メソッドから見てひとつ左の`obj3`となります。

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

## [](#this-problem)*`this`が問題となるパターン*

*`this`はその関数（メソッドも含む）呼び出しのベースオブジェクトを参照することがわかりました。 `this`は所属するオブジェクトを直接書く代わりとして利用できますが、一方`this`にはいろいろな問題があります。

この問題の原因は`this`がどの値を参照するかは関数の呼び出し時に決まるという性質に由来します。 この`this`の性質が問題となるパターンの代表的な2つの例とそれぞれの対策について見ていきます。

### [](#assign-this-function)*問題: `this`を含むメソッドを変数に代入した場合*

*JavaScriptではメソッドとして定義したものが、後からただの関数として呼び出されることがあります。 なぜなら、メソッドは関数を値に持つプロパティのことで、プロパティは変数に代入し直すことができるためです。

そのため、メソッドとして定義した関数も、別の変数に代入してただの関数として呼び出されることがあります。 この場合には、メソッドとして定義した関数であっても、実行時にはただの関数であるためベースオブジェクトが変わっています。 これは`this`が定義した時点ではなく実行したときに決まるという性質そのものです。

具体的に、`this`が実行時に変わる例を見ていきます。 次の例では、`person.sayName`メソッドを変数`say`に代入してから実行しています。 このときの`say`関数（`sayName`メソッドを参照）のベースオブジェクトはありません。 そのため、`this`は`undefined`となり、`undefined.fullName`は参照できずに例外を投げます。

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

最终，执行的代码与下面的代码相同。 在下面的代码中，尝试引用`undefined.fullName`导致了异常。

```
"use strict";
// const say = person.sayName; は次のようなイメージ
const say = function() {
    return this.fullName;
};
// `this`は`undefined`となるため例外を投げる
say(); // => TypeError: Cannot read property 'fullName' of undefined 
```

这样，除了箭头函数之外的函数中，`this`不是在定义时确定的，而是在执行时确定的。 因此，如果函数包含`this`，并且没有按预期调用，就会出现错误的结果。

解决这个问题有两种主要方法。

其中一种方法是，将作为方法定义的函数作为方法调用。 如果不将方法作为普通函数调用，就不会出现这个问题。

另一种方法是，使用指定`this`值调用函数的方法来执行函数。

#### [](#call-apply-bind)*解决方法：call、apply、bind 方法*

*还有一种方法可以显式指定函数或方法的`this`并执行函数。 `Function`（函数对象）提供了`call`、`apply`、`bind`等方法，用于显式指定`this`并执行函数。*

`call`方法将第一个参数指定为要作为`this`的值，并将剩余的参数指定为要传递给调用函数的参数。 可以说`call`方法是一个可以显式传递`this`值的方法。

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

これは、変数におけるスコープチェーンの仕組みと同様で、そのスコープに`this`が定義されていない場合には外側のスコープを探索します。 そのため、Arrow Function 内の`this`の参照で、常に外側のスコープ（関数）へと`this`の定義を探索しに行きます（詳細はスコープチェーンを参照）。 また、`this`はECMAScriptのキーワードであるため、ユーザーは`this`という変数を定義できません。

```
// thisはキーワードであるため、ユーザーは`this`という名前の変数を定義できない
const this = "thisは読み取り専用"; // => SyntaxError: Unexpected token this 
```

これにより、通常の変数のように`this`がどの値を参照するかは静的（定義時）に決定されます（詳細は静的スコープを参照）。 つまり、Arrow Functionにおける`this`は「Arrow Function 自身の外側のスコープに定義されたもっと近い関数の`this`の値」となります。

具体的なArrow Functionにおける`this`の動きを見ていきましょう。

まずは、関数式のArrow Functionを見ていきます。

次の例では、関数式で定義したArrow Functionの中の`this`をコンソールに出力しています。 このとき、`fn`の外側には関数がないため、「自身より外側のスコープに定義されたもっとも近い関数」の条件にあてはまるものはありません。 このときの`this`はトップレベルに書かれた`this`と同じ値になります。

```
// Arrow Functionで定義した関数
const fn = () => {
    // この関数の外側には関数は存在しない
    // トップレベルの`this`と同じ値
    return this;
};
console.log(fn() === this); // => true 
```

トップレベルに書かれた`this`の値は実行コンテキストによって異なることを紹介しました。 `this`の値は、実行コンテキストが"Script"ならばグローバルオブジェクトとなり、"Module"ならば`undefined`となります。

次の例のように、Arrow Functionを包むように通常の関数が定義されている場合はどうでしょうか。 Arrow Functionにおける`this`は「自身の外側のスコープにあるもっと近い関数の`this`の値」となるのは同じです。

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

つまり、このArrow Functionにおける`this`は`outer`関数で`this`を参照した場合と同じ値になります。

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

### [](#method-callback-arrow-function)*メソッドとコールバック関数とArrow Function*

*メソッド内におけるコールバック関数はArrow Functionをより活用できるパターンです。 `function`キーワードでコールバック関数を定義すると、`this`の値はコールバック関数の呼ばれ方を意識する必要があります。 なぜなら、`function`キーワードで定義した関数における`this`は呼び出し方によって変わるためです。

コールバック関数側から見ると、どのように呼ばれるかによって変わってしまう`this`を使うことはできません。 そのため、コールバック関数の外側のスコープで`this`を一時変数に代入し、それを使うという回避方法を取っていました。

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

一方、Arrow Functionでコールバック関数を定義した場合は、1つ外側の関数の`this`を参照します。 このときのArrow Functionで定義したコールバック関数における`this`は呼び出し方によって変化しません。 そのため、`this`を一時変数に代入するなどの回避方法は必要ありません。

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

このArrow Functionにおける`this`は呼び出し方の影響を受けません。 つまり、コールバック関数がどのように呼ばれるかという実装についてを考えることなく`this`を扱えます。

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

### [](#not-bind-arrow-function)*Arrow Functionは`this`をbindできない*

*Arrow Functionで定義した関数では`call`、`apply`、`bind`を使った`this`の指定は単に無視されます。 これは、Arrow Functionは`this`を持てないためです。

次のようにArrow Functionで定義した関数に対して`call`で`this`を指定しても、`this`の参照先が代わっていないことがわかります。 同様に`apply`や`bind`メソッドを使った場合��`this`の参照先は変わりません。

```
const fn = () => {
    return this;
};
// Scriptコンテキストの場合、スクリプト直下のArrow Functionの`this`はグローバルオブジェクト
console.log(fn()); // グローバルオブジェクト
// callで`this`を`{}`にしようとしても、`this`は変わらない
console.log(fn.call({})); // グローバルオブジェクト 
```

最初に述べたように`function`キーワードで定義した関数では呼び出し時に、ベースオブジェクトが`this`の値として暗黙的な引数のように渡されます。 一方、Arrow Functionの関数は呼び出し時に`this`を受け取らず、`this`の参照先は定義時に静的に決定されます。

此外，`this`只在箭头函数中定义的函数中保持不变，箭头函数中`this`引用的是“最接近其外部作用域的函数的`this`值”，可以通过`call`方法进行更改。

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

*介绍了`this`是一个根据情况引用不同值的关键字。 总结`this`的评估结果如下表所示。

| 执行上下文 | 严格模式 | 代码 | `this`的评估结果 |
| --- | --- | --- | --- |
| Script | ＊ | `this` | globalThis |
| Script | ＊ | `const fn = () => this` | globalThis |
| Script | NO | `const fn = function(){ return this; }` | globalThis |
| Script | YES | `const fn = function(){ return this; }` | undefined |
| Script | ＊ | `const obj = { method: () => { return this; } }` | globalThis |
| Module | YES | `this` | undefined |
| Module | YES | `const fn = () => this` | undefined |
| Module | YES | `const fn = function(){ return this; }` | undefined |
| Module | YES | `const obj = { method: () => { return this; } }` | undefined |
| ＊ | ＊ | `const obj = { method(){ return this; } }` | `obj` |
| ＊ | ＊ | `const obj = { method: function(){ return this; } }` | `obj` |

> ＊表示在任何情况下都不会影响`this`的评估结果。

实际在浏览器中运行的结果可以在[JavaScript 中的`this`值是什么](https://azu.github.io/what-is-this/)网站上查看。

`this`是在 JavaScript 中引入的面向对象编程的概念。 除了方法之外，`this`也可以在其他情况下进行评估，但由于执行上下文和严格模式等因素，结果会有所不同，导致混淆。 因此，在非方法函数中不应使用`this`。^(1)

此外，在方法中，`this`根据调用方式而有不同的值，介绍了由此产生的问题和解决方法。 通过使用箭头函数可以更清晰地解决回调函数中的`this`问题。 这背后的原因是通过箭头函数定义的函数不具有`this`属性。

> ¹. ES2015 的规范编辑者 Allen Wirfs-Brock 先生也指出，在普通函数中不应该使用`this`。[`twitter.com/awbjs/status/938272440085446657`](https://twitter.com/awbjs/status/938272440085446657); ↩*******************
