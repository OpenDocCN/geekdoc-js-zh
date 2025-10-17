# 对象

> 原文：[`jsprimer.net/basic/object/`](https://jsprimer.net/basic/object/)

对象是属性的集合。属性是指名称（键）与值（值）相对应的项。属性的键可以使用字符串或`Symbol`，值可以指定任意的数据。此外，一个对象可以拥有多个属性，因此一个对象可以表示多种多样的值。

之前出现的数组或函数等也是对象的一种。JavaScript 有一个名为`Object`的内置对象，它是所有对象的基类。内置对象是在执行环境中预先定义的对象。`Object`这个内置对象是在 ECMAScript 规范中定义的，因此可以在所有 JavaScript 执行环境中使用。

本章将介绍对象的创建和操作方法，以及内置对象`Object`。

## [](#create-object)*创建对象*

*创建对象时，可以使用对象字面量（`{}`）。

```
// プロパティを持たない空のオブジェクトを作成
const obj = {}; 
```

在对象字面量中，可以创建具有初始属性的空对象。属性是在对象字面量（`{}`）中用冒号（`:`）分隔键和值来描述的。

```
// プロパティを持つオブジェクトを定義する
const obj = {
    // キー: 値
    "key": "value"
}; 
```

对象字面量的属性名（键）可以省略引号（`"`或`'`）。因此，可以这样写。

```
// プロパティ名（キー）はクォートを省略することが可能
const obj = {
    // キー: 値
    key: "value"
}; 
```

但是，不能用作变量名的属性名需要用引号（`"`或`'`）括起来。下面的`my-prop`属性名因为包含不能用作变量名的`-`，所以无法定义（请参考“变量与声明”章节中的“变量名可以使用名称的规则”）。

```
const object = {
    // キー: 値
    my-prop: "value" // NG
}; 
```

定义`my-prop`这样的属性名时，需要用引号（`"`或`'`）括起来。

```
const obj = {
    // キー: 値
    "my-prop": "value" // OK
}; 
```

对象字面量可以创建具有多个属性（键和值的组合）的对象。要定义多个属性，需要用逗号（`,`）分隔每个属性。

```
const color = {
    // それぞれのプロパティは`,`で区切る
    red: "red",
    green: "green",
    blue: "blue"
}; 
```

如果将变量名指定为属性值，则该键将引用指定的变量。

```
const name = "名前";
// `name`というプロパティ名で`name`の変数を値に設定したオブジェクト
const obj = {
    name: name
};
console.log(obj); // => { name: "名前" } 
```

从 ES2015 开始，如果指定的属性名和值中的变量名相同，则可以省略`{ name }`，如下面的代码所示，将变量`name`作为值设置到属性名`name`中。

```
const name = "名前";
// `name`というプロパティ名で`name`の変数を値に設定したオブジェクト
const obj = {
    name
};
console.log(obj); // => { name: "名前" } 
```

这种省略语法在模块或分割赋值中也是通用的。因此，当在`{}`中单独书写属性名时，请注意这是在利用这种省略语法。

### [](#object-instance-object)*`{}`是`Object`的实例对象*

*`Object`是 JavaScript 的内置对象。对象字面量（`{}`）是创建基于此内置对象`Object`的新对象的语法。

除了对象字面量之外，还可以使用`new`运算符从`Object`创建新的对象。下面的代码中，使用`new Object()`创建对象，但这与空对象字面量具有相同的意义。

```
// プロパティを持たない空のオブジェクトを作成
// = `Object`からインスタンスオブジェクトを作成
const obj = new Object();
console.log(obj); // => {} 
```

对象字面量更简洁，可以指定属性的初始值，因此使用`new Object()`没有优势。

使用`new Object()`创建对象被称为“创建`Object`的实例对象”。然而，需要区分`Object`、实例对象等术语。因此，本书中无论是使用对象字面量还是`new Object`，都统称为“创建对象”。

应注意对象字面量是从`Object`创建新的实例对象。

## [](#property-access)*属性访问*

*访问对象属性的方法有两种：**点记法**（`.`）和**方括号记法**（`[]`）。使用这两种记法指定属性名时，可以引用具有该名称的属性的值。

```
const obj = {
    key: "value"
};
// ドット記法で参照
console.log(obj.key); // => "value"
// ブラケット記法で参照
console.log(obj["key"]); // => "value" 
```

在点记法（`.`）中，属性名需要满足与变量名相同的标识符命名规则（请参考“变量与声明”章节中的“变量名可以使用名称的规则”）。

```
obj.key; // OK
// プロパティ名が数字から始まる識別子は利用できない
obj.123; // NG
// プロパティ名にハイフンを含む識別子は利用できない
obj.my-prop; // NG 
```

另一方面，**方括号记法**可以在`[`和`]`之间写任意表达式。因此，与标识符的命名规则无关，可以指定任意字符串作为属性名。但请注意，属性名会隐式地转换为字符串。

```
const obj = {
    key: "value",
    123: 456,
    "my-key": "my-value"
};

console.log(obj["key"]); // => "value"
// プロパティ名が数字からはじまる識別子も利用できる
console.log(obj[123]); // => 456
// プロパティ名は暗黙的に文字列に変換されているため、次も同じプロパティを参照している
console.log(obj["123"]); // => 456
// プロパティ名にハイフンを含む識別子も利用できる
console.log(obj["my-key"]); // => "my-value" 
```

此外，在方括号记法中，可以使用变量作为属性名。下面的代码中，使用方括号记法指定了变量`myLang`作为属性名。

```
const languages = {
    ja: "日本語",
    en: "英語"
};
const myLang = "ja";
console.log(languages[myLang]); // => "日本語" 
```

在点记法中，不能使用变量作为属性名，因此如果需要指定变量作为属性名，则应使用方括号记法。基本上，可以使用简洁的点记法（`.`），如果点记法无法使用，则可以使用方括号记法（`[]`）。

## [](#object-destructuring)*[ES2015] 对象与分割赋值*

*在需要多次访问同一对象的属性时，多次书写`对象.属性名`会显得冗长。因此，有时会重新定义一个短名称的变量来引用该属性。

在下面的代码中，定义了变量`ja`和`en`，并将其初始值设置为`languages`对象的属性。

```
const languages = {
    ja: "日本語",
    en: "英語"
};
const ja = languages.ja;
const en = languages.en;
console.log(ja); // => "日本語"
console.log(en); // => "英語" 
```

在重新定义对象属性为变量时，可以使用分割赋值（Destructuring assignment）。

在对象的分割赋值中，将变量名定义为类似于对象字面量的结构。右边的对象中的相应属性名将被赋值给左侧定义的变量。

在下面的代码中，与之前的代码相同，从`languages`对象中提取`ja`和`en`属性，并将其定义为变量。

```
const languages = {
    ja: "日本語",
    en: "英語"
};
const { ja, en } = languages;
console.log(ja); // => "日本語"
console.log(en); // => "英語" 
```

## [](#add-property)*添加属性*

*对象具有可变（可修改）的特性，即使创建后也可以更改其值。因此，可以在创建后向对象添加属性。

添加属性的方法很简单，只需将值分配给要创建的属性名即可。 在这种情况下，如果对象中不存在指定的属性，则会自动创建该属性。

可以使用点表示法或方括号表示法来添加属性。

```
// 空のオブジェクト
const obj = {};
// `key`プロパティを追加して値を代入
obj.key = "value";
console.log(obj.key); // => "value" 
```

正如前面介绍的，点表示法只能使用作为变量标识符的属性名。

另一方面，方括号表示法使用`object[expression]`中`expression`的评估结果作为属性名。 因此，当要将下面的内容作为属性名处理时，应使用方括号表示法。

+   变量

+   无法作为变量标识符的字符串

+   Symbol

```
const key = "key-string";
const obj = {};
// `key`の評価結果 "key-string" をプロパティ名に利用
obj[key] = "value of key";
// 取り出すときも同じく`key`変数を利用
console.log(obj[key]); // => "value of key" 
```

使用方括号表示法定义属性也可以在对象字面量中使用。 在对象字面量中使用方括号表示法定义的属性名称为**Computed property names**。 Computed property names 是从 ES2015 引入的语法，但其将`expression`的评估结果用作属性名与方括号表示法相同。

下面的代码使用 Computed property names，将`key`变量的评估结果`"key-string"`作为属性名。

```
const key = "key-string";
// Computed Propertyで`key`の評価結果 "key-string" をプロパティ名に利用
const obj = {
    [key]: "value"
};
console.log(obj[key]); // => "value" 
```

我们已经介绍了 JavaScript 对象在创建后是可变的特性。 因此，函数可以向其传递的对象自行添加属性。

下面的代码是一个不好的例子，它在`changeProperty`函数中向传入的对象添加属性。

```
function changeProperty(obj) {
    obj.key = "value";
    // いろいろな処理...
}
const obj = {};
changeProperty(obj); // objのプロパティを変更している
console.log(obj.key); // => "value" 
```

这样做会使对象的属性难以理解，因此最好尽量避免在创建后添加新属性。 我们建议在对象创建时使用对象字面量中定义属性。

### 删除属性

*要删除对象的属性，可以使用`delete`运算符。 可以通过将要删除的属性指定为`delete`运算符的右侧来删除属性。

```
const obj = {
    key1: "value1",
    key2: "value2"
};
// key1プロパティを削除
delete obj.key1;
// key1プロパティが削除されている
console.log(obj); // => { "key2": "value2" } 
```

### 使用 const 定义的对象是可变的*[专栏]*

*在前面的代码示例中，可以看到使用`const`声明的对象的属性可以无错误地更改。 执行下面的代码后，可以看到对象的属性值已更改。

```
const obj = { key: "value" };
obj.key = "Hi!"; // constで定義したオブジェクト(`obj`)が変更できる
console.log(obj.key); // => "Hi!" 
```

JavaScript 的`const`并不是固定值，而是用于防止对变量的重新赋值。 因此，虽然可以防止对`obj`变量的重新赋值，但无法阻止对变量所指向的对象的更改（参考"变量和声明"中的 const）。

```
const obj = { key: "value" };
obj = {}; // => TypeError 
```

要防止更改创建的对象的属性，需要使用`Object.freeze`方法。 `Object.freeze`会冻结对象。 对冻结的对象进行属性添加或更改将导致抛出异常。

但是，如果使用`Object.freeze`方法，则必须与 strict mode 一起使用（有关详细信息，请参阅"JavaScript 简介"中的 strict mode）。 在非 strict mode 下，即使对象被冻结，也不会抛出异常，而是简单地忽略对冻结对象的属性更改。

```
"use strict";
const object = Object.freeze({ key: "value" });
// freezeしたオブジェクトにプロパティを追加や変更できない
object.key = "value"; // => TypeError: "key" is read-only 
```

## 确认属性是否存在

*在 JavaScript 中，访问不存在的属性不会引发异常，而是返回`undefined`。 下面的代码尝试访问`obj`中不存在的`notFound`属性，因此返回`undefined`值。

```
const obj = {};
console.log(obj.notFound); // => undefined 
```

在 JavaScript 中，访问不存在的属性不会引发异常。 由于属性名拼写错误，只会返回`undefined`值，这可能导致错误不易被发现。

即使属性名拼写错误，也不会引发异常。 只有在尝试访问嵌套属性名时才会引发异常。

```
const widget = {
    window: {
        title: "ウィジェットのタイトル"
    }
};
// `window`を`windw`と間違えているが、例外は発生しない
console.log(widget.windw); // => undefined
// さらにネストした場合に、例外が発生する
// `undefined.title`と書いたのと同じ意味となるため
console.log(widget.windw.title); // => TypeError: widget.windw is undefined
// 例外が発生した文以降は実行されません 
```

`undefined`和`null`不是对象，因此访问不存在的属性会导致异常。 有四种方法可以检查对象是否具有某个属性。

+   `undefined`与之比较

+   `in`演算子

+   `Object.hasOwn`静态方法^([ES2022])

+   `Object.prototype.hasOwnProperty`方法

### 检查属性是否存在：与 undefined 比较

*访问不存在的属性会返回`undefined`，因此通过实际访问属性也可以进行判断。 下面的代码通过检查`key`属性的值是否不为`undefined`来判断属性是否存在。

```
const obj = {
    key: "value"
};
// `key`プロパティが`undefined`ではないなら、プロパティが存在する?
if (obj.key !== undefined) {
    // `key`プロパティが存在する?ときの処理
    console.log("`key`プロパティの値は`undefined`ではない");
} 
```

然而，这种方法存在一个问题，即当属性的值为`undefined`时，无法区分属性本身是否存在。 在下面的代码中，由于`key`属性的值为`undefined`，因此尽管属性存在，但 if 语句块不会执行。

```
const obj = {
    key: undefined
};
// `key`プロパティの値が`undefined`である場合
if (obj.key !== undefined) {
    // この行は実行されません
} 
```

由于存在这样的问题，要判断属性是否存在，可以使用`in`运算符或`Object.hasOwn`静态方法。

### 检查属性是否存在：使用 in 运算符

*`in`运算符用于判断指定对象上是否存在指定属性，并返回布尔值。

```
"プロパティ名" in オブジェクト; // true or false 
```

下面的代码检查`obj`是否具有`key`属性。 `in`运算符只关注属性的存在与否，而不关心属性的值。

```
const obj = { key: undefined };
// `key`プロパティを持っているならtrue
if ("key" in obj) {
    console.log("`key`プロパティは存在する");
} 
```

### 检查属性是否存在：`Object.hasOwn`静态方法*[ES2022]*

*`Object.hasOwn`静态方法可以确定目标对象是否具有指定的属性。 将要检查的属性名称作为`Object.hasOwn`静态方法的参数传递给它。

```
const obj = {};
// objが"プロパティ名"を持っているかを確認する
Object.hasOwn(obj, "プロパティ名"); // true or false 
```

下面的代码判断了`obj`是否具有`key`属性。 `Object.hasOwn`静态方法也是如此，无论属性的值如何，只要对象具有指定的属性，它就会返回`true`。

```
const obj = { key: undefined };
// `obj`が`key`プロパティを持っているならtrueとなる
if (Object.hasOwn(obj, "key")) {
    console.log("`obj`は`key`プロパティを持っている");
} 
```

`in`操作符和`Object.hasOwn`静态方法返回相同的结果，但在严格意义上它们的行为有时会有所不同。 要了解这种行为上的差异，首先需要理解特殊对象——原型对象。 因此，关于`in`操作符和`Object.hasOwn`静态方法的区别，将在下一章的“原型对象”中详细解释。

### 检查属性是否存在：`Object.prototype.hasOwnProperty`方法

*`Object.hasOwn`静态方法是在 ES2022 中引入的。 在 ES2022 之前，通常使用类似的方法`Object.prototype.hasOwnProperty`。 `hasOwnProperty`方法与`Object.hasOwn`静态方法非常相似，但它们在调用它们的对象实例方面有所不同。

```
const obj = { key: undefined };
// `obj`が`key`プロパティを持っているならtrueとなる
if (obj.hasOwnProperty("key")) {
    console.log("`obj`は`key`プロパティを持っている");
} 
```

但是，由于`hasOwnProperty`方法存在缺陷，因此在可以使用`Object.hasOwn`静态方法的情况下，没有理由使用它。 由于这个缺陷也与原型对象有关，所以将在下一章的“原型对象”中详细解释。

## 可选链操作符（`?.`）*[ES2020]*

*介绍了 4 种检查属性是否存在的方法。 如果属性的存在很重要，通常会使用`in`操作符或`Object.hasOwn`静态方法。*

但是，如果最终想要获取的是属性的值，则使用 if 语句与`undefined`进行比较也没有问题。 因为当需要获取值时，区分属性是否存在以及属性的值是否为`undefined`并没有意义。

下面的代码如果定义了`widget.window.title`属性的值（不是`undefined`），则将该属性的值显示在控制台上。

```
function printWidgetTitle(widget) {
    // 例外を避けるために`widget`のプロパティの存在を順番に確認してから、値を表示している
    if (widget.window !== undefined && widget.window.title !== undefined) {
        console.log(`ウィジェットのタイトルは${widget.window.title}です`);
    } else {
        console.log("ウィジェットのタイトルは未定義です");
    }
}
// タイトルが定義されているwidget
printWidgetTitle({
    window: {
        title: "Book Viewer"
    }
});
// タイトルが未定義のwidget
printWidgetTitle({
    // タイトルが定義されてない空のオブジェクト
}); 
```

在访问嵌套属性（如`widget.window.title`）时，必须逐个检查属性的存在。 这是因为如果`widget`对象不具有`window`属性，则会返回`undefined`。 在这种情况下，如果进一步访问嵌套的`widget.window.title`属性，则会导致引用`undefined.title`，从而导致异常。

然而，每次访问属性时都用 AND 操作符（`&&`）与`undefined`进行比较会显得冗长。

为了解决这个问题，在 ES2020 中引入了一个新的语法，即可选链操作符（`?.`）。 可选链操作符（`?.`）与点表示法（`.`）类似，用于访问属性。

可选链操作符（`?.`）在左操作数为 nullish（`null`或`undefined`）时不会继续评估并返回`undefined`。 反之，如果属性存在，则返回该属性的值。

换句话说，可选链操作符（`?.`）在访问不存在的属性时不会引发异常，而是返回`undefined`。

```
const obj = {
    a: {
        b: "objのaプロパティのbプロパティ"
    }
};
// obj.a.b は存在するので、その評価結果を返す
console.log(obj?.a?.b); // => "objのaプロパティのbプロパティ"
// 存在しないプロパティのネストも`undefined`を返す
// ドット記法の場合は例外が発生してしまう
console.log(obj?.notFound?.notFound); // => undefined
// undefinedやnullはnullishなので、`undefined`を返す
console.log(undefined?.notFound?.notFound); // => undefined
console.log(null?.notFound?.notFound); // => undefined 
```

当使用 Optional chaining 操作符（`?.`）时，甚至不需要使用 if 语句就可以编写一个函数来显示上述小部件的标题。 在下面的`printWidgetTitle`函数中，如果可以访问`widget?.window?.title`，那么它的评估结果将被赋给变量`title`。 如果无法访问属性，则返回`undefined`，因此通过 Nullish coalescing 操作符(`??`)，右侧的`"未定义"`将成为变量`title`的默认值。

```
function printWidgetTitle(widget) {
    const title = widget?.window?.title ?? "未定義";
    console.log(`ウィジェットのタイトルは${title}です`);
}
printWidgetTitle({
    window: {
        title: "Book Viewer"
    }
}); // "ウィジェットのタイトルはBook Viewerです" と出力される
printWidgetTitle({
    // タイトルが定義されてない空のオブジェクト
}); // "ウィジェットのタイトルは未定義です" と出力される 
```

此外，可选链操作符（`?.`）也可以与方括号表示法（`[]`）结合使用。 在方括号表示法中，如果左操作数为 nullish（`null`或`undefined`），则不会继续评估并返回`undefined`。 反之，如果属性存在，则返回该属性的值。

```
const languages = {
    ja: {
        hello: "こんにちは！"
    },
    en: {
        hello: "Hello!"
    }
};
const langJapanese = "ja";
const langKorean = "ko";
const messageKey = "hello";
// Optional chaining 演算子（`?.`）とブラケット記法を組みわせた書き方
console.log(languages?.[langJapanese]?.[messageKey]); // => "こんにちは！"
// `languages`に`ko`プロパティが定義されていないため、`undefined`を返す
console.log(languages?.[langKorean]?.[messageKey]); // => undefined 
```

## [](#toString-method)*`toString`方法*

*对象的`toString`方法是一种将对象本身转换为字符串的方法。 也可以使用`String`构造函数来执行此操作。 那么这两者有什么区别呢？（关于`String`构造函数，请参阅“隐式强制类型转换”）*

实际上`String`构造函数会调用传递给它的对象的`toString`方法。因此，`String`构造函数和`toString`方法的结果都是相同的。

```
const obj = { key: "value" };
console.log(obj.toString()); // => "[object Object]"
// `String`コンストラクタ関数は`toString`メソッドを呼んでいる
console.log(String(obj)); // => "[object Object]" 
```

这一点可以通过重新定义对象的`toString`方法来理解。 试着将自定义的`toString`方法定义为对象，然后尝试使用`String`构造函数将其转换为字符串。 结果会发现，重新定义的`toString`方法的返回值将成为`String`构造函数的返回值。

```
// 独自のtoStringメソッドを定義
const customObject = {
    toString() {
        return "custom value";
    }
};
console.log(String(customObject)); // => "custom value" 
```

## [](#object-property-is-to-string)*[专栏] 对象属性名会被转换为字符串*

*在访问对象的属性时，指定的属性名会被隐式转换为字符串。 使用方括号表示法时，可以将对象作为属性名指定，但这并不会按预期工作。 这是因为将对象转换为字符串会得到`"[object Object]"`这样的字符串。

下面的代码中，使用方括号表示法指定了`keyObject1`和`keyObject2`作为属性名。 但是，`keyObject1`和`keyObject2`都会被转换为相同的`"[object Object]"`字符串，导致属性被意外覆盖。

```
const obj = {};
const keyObject1 = { a: 1 };
const keyObject2 = { b: 2 };
// どちらも同じプロパティ名（"[object Object]"）に代入している
obj[keyObject1] = "1";
obj[keyObject2] = "2";
console.log(obj); //  { "[object Object]": "2" } 
```

作为唯一的例外，Symbol 不会被转换为字符串，可以作为对象的属性名。

```
const obj = {};
// Symbolは例外的に文字列化されず扱える
const symbolKey1 = Symbol("シンボル1");
const symbolKey2 = Symbol("シンボル2");
obj[symbolKey1] = "1";
obj[symbolKey2] = "2";
console.log(obj[symbolKey1]); // => "1"
console.log(obj[symbolKey2]); // => "2" 
```

通常，应该记住对象的属性名被视为字符串。 此外，内置对象`Map`可以将对象用作键（详细信息请参阅“Map/Set”章节）。 因此，如果要将对象用作键，则应使用`Map`。

## [](#static-method)*对象的静态方法*

*最后，让我们看一下作为内置对象的`Object`的静态方法。 **静态方法**是可以从实例对象的基础对象调用的方法。

`Object`的`toString`方法等是从`Object`的实例对象调用的方法。 相反，像`Object.hasOwn`这样的静态方法是`Object`本身实现的方法。

这里介绍了一些在对象处理中常用的**静态方法**。

### [](#enumeration)*对象的枚举*

*正如之前提到的，对象是属性的集合。 有三种静态方法可用于枚举对象的属性。

+   `Object.keys`方法：返回一个由对象的属性名组成的数组

+   `Object.values`方法^([ES2017])：返回对象值的数组

+   `Object.entries`方法^([ES2017])：返回对象属性名和值的数组的数组

分别返回对象的键、值和键值对的数组。

```
const obj = {
    "one": 1,
    "two": 2,
    "three": 3
};
// `Object.keys`はキーを列挙した配列を返す
console.log(Object.keys(obj)); // => ["one", "two", "three"]
// `Object.values`は値を列挙した配列を返す
console.log(Object.values(obj)); // => [1, 2, 3]
// `Object.entries`は[キー, 値]の配列を返す
console.log(Object.entries(obj)); // => [["one", 1], ["two", 2], ["three", 3]] 
```

结合这些列举属性的静态方法和数组的`forEach`方法，可以对属性进行迭代处理。 下面的代码使用`Object.keys`方法获取属性名列表并将其输出到控制台。

```
const obj = {
    "one": 1,
    "two": 2,
    "three": 3
};
const keys = Object.keys(obj);
keys.forEach(key => {
    console.log(key);
});
// 次の値が順番に出力される
// "one"
// "two"
// "three" 
```

### [](#copy-and-merge)*对象的合并和复制*

*`Object.assign`方法^([ES2015])可以将一个对象分配给另一个对象。 使用此方法可以复制对象或合并对象。

`Object.assign`方法用于将一个或多个`sources`对象的可枚举属性复制到`target`对象中。 返回值是`target`对象。

```
const obj = Object.assign(target, ...sources); 
```

#### [](#merge)*对象的合并*

*让我们看一些具体的对象合并示例。

下面的代码将创建一个新的空对象作为`target`。 这个空对象（`target`）将合并`objectA`和`objectB`，成为`Object.assign`方法的返回值。

```
const objectA = { a: "a" };
const objectB = { b: "b" };
const merged = Object.assign({}, objectA, objectB);
console.log(merged); // => { a: "a", b: "b" } 
```

第一���参数可以是现有对象，而不仅仅是空对象。 如果指定现有对象作为第一个参数，则该对象的属性将被修改。

下面的代码向指定的`objectA`添加属性。

```
const objectA = { a: "a" };
const objectB = { b: "b" };
const merged = Object.assign(objectA, objectB);
console.log(merged); // => { a: "a", b: "b" }
// `objectA`が変更されている
console.log(objectA); // => { a: "a", b: "b" }
console.log(merged === objectA); // => true 
```

通过将空对象作为`target`，可以创建一个合并了对象而不影响现有对象的新对象。 因此，通常将空对象字面量作为`Object.assign`方法的第一个参数是典型用法。

当属性名重复时，后一个对象的属性将覆盖前一个对象的属性。 在 JavaScript 中，通常是按顺序从前到后处理。 因此，可以将`objectA`分配给空对象，然后将`objectB`分配给结果。

```
// `version`のプロパティ名が被っている
const objectA = { version: "a" };
const objectB = { version: "b" };
const merged = Object.assign({}, objectA, objectB);
// 後ろにある`objectB`のプロパティで上書きされる
console.log(merged); // => { version: "b" } 
```

#### [](#object-spread-syntax)*[ES2018] 对象的 spread 構文合并*

*在 ES2018 中，引入了对象的合并操作符`...`（spread 構文）。 在 ES2015 中，数组元素的展开操作符`...`（spread 構文）已经被支持，而在 ES2018 中也支持了对象。 对象的 spread 構文可以展开指定对象的属性到对象字面量中。

对象的 spread 構文会创建一个新对象，与`Object.assign`不同。 这是因为 spread 構文只能在对象字面量中使用，而对象字面量会创建新对象。

下面的代码将返回合并了`objectA`和`objectB`的新对象。

```
const objectA = { a: "a" };
const objectB = { b: "b" };
const merged = {
    ...objectA,
    ...objectB
};
console.log(merged); // => { a: "a", b: "b" } 
```

当属性名冲突时，后一个对象将优先。 因此，当合并具有相同属性名的对象时，后一个对象将覆盖属性。

```
// `version`のプロパティ名が被っている
const objectA = { version: "a" };
const objectB = { version: "b" };
const merged = {
    ...objectA,
    ...objectB,
    other: "other"
};
// 後ろにある`objectB`のプロパティで上書きされる
console.log(merged); // => { version: "b", other: "other" } 
```

#### [](#copy)*对象的复制*

*JavaScript 中没有提供复制对象的函数。 但是，通过创建一个新的空对象，并将现有对象的属性复制到其中，可以实现对象的复制。 使用`Object.assign`方法可以复制对象。

```
// 引数の`obj`を浅く複製したオブジェクトを返す
const shallowClone = (obj) => {
    return Object.assign({}, obj);
};
const obj = { a: "a" };
const cloneObj = shallowClone(obj);
console.log(cloneObj); // => { a: "a" }
// オブジェクトを複製しているので、異なるオブジェクトとなる
console.log(obj === cloneObj); // => false 
```

需要注意的是，`Object.assign`方法只会进行浅拷贝（shallow copy）操作，即只会拷贝`sources`对象直接的属性。浅拷贝意味着不会递归地复制嵌套对象的属性。如果属性的值是一个对象，它不会复制嵌套的对象。

```
const shallowClone = (obj) => {
    return Object.assign({}, obj);
};
const obj = {
    level: 1,
    nest: {
        level: 2
    },
};
const cloneObj = shallowClone(obj);
// `nest`プロパティのオブジェクトは同じオブジェクトのままになる 
console.log(cloneObj.nest === obj.nest); // => true 
```

相反，如果要递归地复制属性值，即进行深拷贝（deep copy），可以通过递归调用浅拷贝来实现。下面的代码示例展示了如何使用`shallowClone`来实现`deepClone`。

```
// 引数の`obj`を浅く複製したオブジェクトを返す
const shallowClone = (obj) => {
    return Object.assign({}, obj);
};
// 引数の`obj`を深く複製したオブジェクトを返す
function deepClone(obj) {
    const newObj = shallowClone(obj);
    // プロパティがオブジェクト型であるなら、再帰的に複製する
    Object.keys(newObj)
        .filter(k => typeof newObj[k] === "object")
        .forEach(k => newObj[k] = deepClone(newObj[k]));
    return newObj;
}
const obj = {
    level: 1,
    nest: {
        level: 2
    }
};
const cloneObj = deepClone(obj);
// `nest`オブジェクトも再帰的に複製されている
console.log(cloneObj.nest === obj.nest); // => false 
```

因此，JavaScript 的内置方法通常只提供浅实现，并且很少提供深实现。这是因为 JavaScript 作为一种语言只提供了最基本的功能，更复杂的功能需要用户自己实现。

JavaScript 作为语言规范最低限度地定义了功能，因此有很多由用户创建的小型功能库来补充它。这些库通过名为 npm 的 JavaScript 包管理工具进行发布，构建了 JavaScript 生态系统。有关使用库的信息，请参阅“用例：在 Node.js 中创建 CLI 应用程序”章节。

## [](#conclusion)*总结*

*本章介绍了对象。*

+   存在名为`Object`的内置对象。

+   使用`{}`（对象字面量）来创建和更新对象的方法。

+   要确认属性是否存在，可以使用`in`操作符或`Object.hasOwn`静态方法。

+   可选链操作符（`?.`）允许同时检查嵌套属性的存在并访问它们。

+   对象的实例方法和静态方法。

JavaScript 的`Object`是其他对象的基础对象。接下来的"原型对象"章节将探讨`Object`作为基础对象的运作方式。
