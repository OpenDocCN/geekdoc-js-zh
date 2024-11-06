# JSON

> 原文：[`jsprimer.net/basic/json/`](https://jsprimer.net/basic/json/)

この章では、JavaScriptと密接な関係にあるJSONというデータフォーマットについて見ていきます。

## [](#what-is-json)*JSONとは*

*JSONはJavaScript Object Notationの略で、JavaScriptのオブジェクトリテラルをベースに作られた軽量なデータフォーマットです。 JSONの仕様は[ECMA-404](https://www.ecma-international.org/publications-and-standards/standards/ecma-404/)として標準化されています。 JSONは、人間にとって読み書きが容易で、マシンにとっても簡単にパースや生成を行える形式になっています。 そのため、多くのプログラミング言語がJSONを扱う機能を備えています。

JSONはJavaScriptのオブジェクトリテラル、配列リテラル、各種プリミティブ型の値を組み合わせたものです。 ただしJSONとJavaScriptは一部の構文に違いがあります。 たとえばJSONでは、オブジェクトリテラルのキーを必ずダブルクォートで囲まなければいけません。 また、小数点から書きはじめる数値リテラルや、先頭がゼロからはじまる数値リテラルも使えません。 これらは機械がパースしやすくするために仕様で定められた制約です。

```
{
    "object": {
        "number": 1,
        "string": "js-primer",
        "boolean": true,
        "null": null,
        "array": [1, 2, 3]
    }
} 
```

JSONの細かい仕様に関しては[json.orgの日本語ドキュメント](https://www.json.org/json-ja.html)にわかりやすくまとまっているので、参考にするとよいでしょう。

## [](#json-object)*`JSON`オブジェクト*

*JavaScriptでJSONを扱うには、ビルトインオブジェクトである[JSONオブジェクト](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/JSON)を利用します。 `JSON`オブジェクトはJSON 形式の文字列とJavaScriptのオブジェクトを相互に変換するための`parse`メソッドと`stringify`メソッドを提供します。

### [](#json-parse)*JSON 文字列をオブジェクトに変換する*

*[JSON.parseメソッド](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse)は引数に与えられた文字列をJSONとしてパースし、その結果をJavaScriptのオブジェクトとして返す関数です。 次のコードは簡単なJSON 形式の文字列をJavaScriptのオブジェクトに変換する例です。

```
// JSONはダブルクォートのみを許容するため、シングルクォートでJSON 文字列を記述
const json = '{ "id": 1, "name": "js-primer" }';
const obj = JSON.parse(json);
console.log(obj.id); // => 1
console.log(obj.name); // => "js-primer" 
```

文字列がJSONの配列を表す場合は、`JSON.parse`メソッドの返り値も配列になります。

```
const json = "[1, 2, 3]";
console.log(JSON.parse(json)); // => [1, 2, 3] 
```

与えられた文字列がJSON 形式でパースできない場合は例外が投げられます。 また、実際のアプリケーションでJSONを扱うのは、外部のプログラムとデータを交換する用途がほとんどです。 外部のプログラムが送ってくるデータが常にJSONとして正しい保証はありません。 そのため、`JSON.parse`メソッドは基本的に`try...catch`構文で例外処理をするべきです。

```
const userInput = "not json value";
try {
    const json = JSON.parse(userInput);
} catch (error) {
    console.log("パースできませんでした");
} 
```

### [](#json-format)*オブジェクトをJSON 文字列に変換する*

*[JSON.stringifyメソッド](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify)は第一引数に与えられたオブジェクトをJSON 形式の文字列に変換して返す関数です。 HTTP 通信でサーバーにデータを送信するときや、 アプリケーションが保持している状態を外部に保存するときなどに必要になります。 次のコードはJavaScriptのオブジェクトをJSON 形式の文字列に変換する例です。

```
const obj = { id: 1, name: "js-primer", bio: null };
console.log(JSON.stringify(obj)); // => '{"id":1,"name":"js-primer","bio":null}' 
```

`JSON.stringify`メソッドにはオプショナルな引数が2つあります。 第二引数はreplacer 引数とも呼ばれ、関数あるいは配列を渡せます。 関数を渡した場合は引数にプロパティのキーと値が渡され、その返り値によって文字列に変換される際の挙動をコントロールできます。 次の例は値がnullであるプロパティを除外してJSONに変換するreplacer 引数の例です。 replacer 引数の関数で`undefined`が返されたプロパティは、変換後のJSONに含まれなくなります。

```
const obj = { id: 1, name: "js-primer", bio: null };
const replacer = (key, value) => {
    if (value === null) {
        return undefined;
    }
    return value;
};
console.log(JSON.stringify(obj, replacer)); // => '{"id":1,"name":"js-primer"}' 
```

replacer 引数に配列を渡した場合はプロパティの許可リストとして使われ、 その配列に含まれる名前のプロパティだけが変換されます。

```
const obj = { id: 1, name: "js-primer", bio: null };
const replacer = ["id", "name"];
console.log(JSON.stringify(obj, replacer)); // => '{"id":1,"name":"js-primer"}' 
```

第三引数はspace 引数とも呼ばれ、変換後のJSON 形式の文字列を読みやすくフォーマットする際のインデントを設定できます。 数値を渡すとその数値分の長さのスペースで、文字列を渡すとその文字列でインデントされます。 次のコードはスペース2 個でインデントされたJSONを得る例です。

```
const obj = { id: 1, name: "js-primer" };
// replacer 引数を使わない場合はnullを渡して省略するのが一般的です
console.log(JSON.stringify(obj, null, 2));
/*
{
   "id": 1,
   "name": "js-primer"
}
*/ 
```

また、次のコードはタブ文字でインデントされたJSONを得る例です。

```
const obj = { id: 1, name: "js-primer" };
console.log(JSON.stringify(obj, null, "\t"));
/*
{
   "id": 1,
   "name": "js-primer"
}
*/ 
```

## [](#not-serialization-object)*JSONにシリアライズできないオブジェクト*

*`JSON.stringify`メソッドはJSONで表現可能な値だけをシリアライズします。 そのため、値が関数や`Symbol`、あるいは`undefined`であるプロパティなどは変換されません。 ただし、配列の値としてそれらが見つかったときには例外的に`null`に置き換えられます。 またキーが`Symbol`である場合にもシリアライズの対象外になります。 代表的な変換の例を次の表とサンプルコードに示します。

| シリアライズ前の値 | シリアライズ後の値 |
| --- | --- |
| 文字列・数値・真偽値 | 対応する値 |
| null | null |
| 配列 | 配列 |
| オブジェクト | オブジェクト |
| 関数 | 変換されない（配列のときはnull） |
| undefined | 変換されない（配列のときはnull） |
| Symbol | 変換されない（配列のときはnull） |
| RegExp | {} |
| Map, Set | {} |
| BigInt | 例外が発生する |

```
// 値が関数のプロパティ
console.log(JSON.stringify({ x: function() {} })); // => '{}'
// 値がSymbolのプロパティ
console.log(JSON.stringify({ x: Symbol("") })); // => '{}'
// 値がundefinedのプロパティ
console.log(JSON.stringify({ x: undefined })); // => '{}'
// 配列の場合
console.log(JSON.stringify({ x: [10, function() {}] })); // => '{"x":[10,null]}'
// キーがSymbolのプロパティ
JSON.stringify({ [Symbol("foo")]: "foo" }); // => '{}'
// 値がRegExpのプロパティ
console.log(JSON.stringify({ x: /foo/ })); // => '{"x":{}}'
// 値がMapのプロパティ
const map = new Map();
map.set("foo", "foo");
console.log(JSON.stringify({ x: map })); // => '{"x":{}}' 
```

オブジェクトがシリアライズされる際は、そのオブジェクトの列挙可能なプロパティだけが再帰的にシリアライズされます。 `RegExp`や`Map`、`Set`などのインスタンスは列挙可能なプロパティを持たないため、空のオブジェクトに変換されます。

此外，`JSON.stringify`方法有时会序列化失败。常见的情况是当尝试序列化具有循环引用的对象时，会引发异常。例如，如果递归遍历某个对象的属性，直到发现自身时，序列化将变得不可能。不仅仅是`JSON.parse`方法，`JSON.stringify`方法也要进行异常处理，以确保安全使用。

```
const obj = { foo: "foo" };
obj.self = obj;
try {
    JSON.stringify(obj);
} catch (error) {
    console.error(error); // => "TypeError: Converting circular structure to JSON"
} 
```

## [](#serialization-by-toJSON)*使用`toJSON`方法进行序列化*

*如果对象具有`toJSON`方法，则`JSON.stringify`方法将使用`toJSON`方法的返回值，而不是默认的字符串转换。就像下面的例子一样，它将递归地处理作为参数直接传递以及作为参数属性出现的情况。*

```
const obj = {
    foo: "foo",
    toJSON() {
        return "bar";
    }
};
console.log(JSON.stringify(obj)); // => '"bar"'
console.log(JSON.stringify({ x: obj })); // => '{"x":"bar"}' 
```

`toJSON`方法被用于将自定义类以特殊格式进行序列化。

## [](#conclusion)*总结*

*在本章中，我们学习了关于 JSON 的知识。

+   JSON 是基于 JavaScript 对象文字的轻量级数据格式。

+   使用`JSON`对象进行序列化和反序列化。

+   也有一些无法被序列化为 JSON 格式的对象。

+   `JSON.stringify`利用了要序列化的对象的`toJSON`方法**********
