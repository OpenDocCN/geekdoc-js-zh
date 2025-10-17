# 数组

> 原文：[`jsprimer.net/basic/array/`](https://jsprimer.net/basic/array/)

数组是 JavaScript 中常用的对象。

数组是按顺序存储值的对象。数组中存储的每个值称为**元素**，每个元素的位置称为**索引**（`index`）。索引是从第一个元素开始的`0`、`1`、`2`等连续序列。

此外，JavaScript 中的数组是可变长的。因此，可以在创建数组后向数组中添加元素或从数组中删除元素。

本章将学习数组的基本操作以及处理数组时的模式。

## [](#create-and-access)*数组的创建与访问*

*虽然数组的创建和访问方法已在“数据类型与字面量”章节中介绍，但让我们再次回顾一下。

数组的创建使用数组字面量。数组字面量（`[`和`]`）中用逗号（`,`）分隔的元素进行描述。

```
const emptyArray = [];
const numbers = [1, 2, 3];
// 2 次元配列（配列の配列）
const matrix = [
    ["a", "b"],
    ["c", "d"]
]; 
```

通过将创建的数组元素的索引数值用`数组[索引]`这样的方式来描述，可以从数组中读取该索引的元素。数组的第一个元素的索引是`0`。数组的索引是`0`以上的整数，小于`2³² - 1`。

```
const array = ["one", "two", "three"];
console.log(array[0]); // => "one" 
```

也可以使用`数组[索引]`从二维数组（数组中的数组）中读取值。`数组[0][0]`将读取数组中`0`号元素（`["a", "b"]`）的`0`号元素。

```
// 2 次元配列（配列の配列）
const matrix = [
    ["a", "b"],
    ["c", "d"]
];
console.log(matrix[0][0]); // => "a" 
```

数组的`length`属性返回数组元素的个数。因此，要访问数组的最后一个元素，可以使用`array.length - 1`作为索引。

```
const array = ["one", "two", "three"];
console.log(array.length); // => 3
// 配列の要素数 - 1 が 最後の要素のインデックスとなる
console.log(array[array.length - 1]); // => "three" 
```

另一方面，如果访问不存在的索引，会发生什么？在 JavaScript 中，访问不存在的索引时，不会抛出异常，而是返回`undefined`。

```
const array = ["one", "two", "three"];
// `array`にはインデックスが100の要素は定義されていない
console.log(array[100]); // => undefined 
```

这是因为，考虑到数组是对象，所以访问不存在的属性与原理相同。对象在访问不存在的属性时，也会返回`undefined`。

```
const obj = {
    "0": "one",
    "1": "two",
    "2": "three",
    "length": 3
};
// obj["100"]は定義されていないため、undefinedが返る
console.log(obj[100]); // => undefined 
```

此外，数组不一定总是包含`length`数量的元素。如下所示，在数组字面量中省略值可以包含未定义的元素。这种在数组中存在空隙的数组称为**稀疏数组**。另一方面，没有空隙且所有索引都有元素的数组称为**密集数组**。

```
// 未定義の箇所が1つ含まれる疎な配列
// インデックスが1の値を省略しているので、カンマが2つ続いていることに注意
const sparseArray = [1, , 3];
console.log(sparseArray.length); // => 3
// 1 番目の要素は存在しないため undefined が返る
console.log(sparseArray[1]); // => undefined 
```

### [](#array-at)*[ES2022] `Array.prototype.at`*

*在介绍访问数组元素时，我们提到了使用`数组[索引]`这种结构。在这种情况下，要访问数组的最后一个元素，需要使用`array[array.length - 1]`，这需要使用`length`属性。访问最后一个元素稍微有些麻烦，因为需要两次写入`array`。

为了解决这个问题，ES2022 中添加了`Array.prototype.at`方法，可以通过指定相对索引值来访问数组元素。`Array`的`at`方法与`数组[索引]`类似，但可以传递相对索引值作为参数。例如，`.at(0)`或`.at(1)`，传递 0 以上的索引时，与`数组[索引]`一样，可以访问指定位置的元素。另一方面，`.at(-1)`等传递负索引时，可以访问从末尾开始计数的指定位置的元素。

```
const array = ["a", "b", "c"];
//
console.log(array.at(0)); // => "a"
console.log(array.at(1)); // => "b"
// 後ろから1つ目の要素にアクセス
console.log(array.at(-1)); // => "c"
// -1は、次のように書いた場合と同じ結果
console.log(array[array.length - 1]); // => "c" 
```

将`-1`指定给`数组[索引]`的索引时，将访问数组对象的`"-1"`属性。因此，`数组[-1]`通常将返回`undefined`。

```
const array = ["a", "b", "c"];
console.log(array[-1]); // => undefined 
```

## [](#detect-array)*判断对象是否为数组*

*要判断一个对象是否为数组，可以使用`Array.isArray`方法。`Array.isArray`方法如果参数是数组，则返回`true`。

```
const obj = {};
const array = [];
console.log(Array.isArray(obj)); // => false
console.log(Array.isArray(array)); // => true 
```

此外，`typeof`运算符无法判断是否为数组。因为数组是对象的一种，所以`typeof`运算符的结果为`"object"`。

```
const array = [];
console.log(typeof array); // => "object" 
```

### [](#typed-array)*[专栏] [ES2015] TypedArray*

*JavaScript 的数组是可变长的，但存在另一种名为`TypedArray`的对象，它具有固定长度和类型。`TypedArray`是用于表示二进制数据缓冲区的数据类型，在 WebGL 或处理二进制数据时使用。它不能直接使用字符串或数值等原始类型值，因此与普通数组的用途和易用性不同。

另外，由于`TypedArray`的`Array.isArray`方法的结果为`false`，因此可以将其视为另一种类型。

```
// TypedArrayを作成
const typedArray = new Int8Array(8);
console.log(Array.isArray(typedArray)); // => false 
```

因此，在 JavaScript 中提到数组时，应使用`Array`。

## [](#array-destructuring)*[ES2015] 数组与解构赋值*

*在重新定义指定索引的值时，可以使用解构赋值（Destructuring assignment）。

在数组解构赋值中，在左边用类似数组字面量的结构定义要设置的变量名。右边的数组中相应索引的元素将被赋值到左边定义的变量中。

在下面的代码中，将左边的变量与右边的数组中相应索引的元素进行赋值。`first`将赋值为索引为`0`的元素，`second`将赋值为索引为`1`的元素，`third`将赋值为索引为`2`的元素。

```
const array = ["one", "two", "three"];
const [first, second, third] = array;
console.log(first);  // => "one"
console.log(second); // => "two"
console.log(third);  // => "three" 
```

## [](#diff-undefined-and-no-element)*[专栏] undefined 的元素与未定义的元素的区别*

*在稀疏数组中，如果对应的索引没有元素，则返回`undefined`。但是，也存在`undefined`这个值，因此当数组中存在`undefined`值时，无法区分。

在下面的代码中，定义了一个包含`undefined`值的密集数组和一个没有元素的稀疏数组。两者访问元素的结果都是`undefined`，无法区分。

```
// 要素として`undefined`を持つ密な配列
const denseArray = [1, undefined, 3];
// 要素そのものがない疎な配列
const sparseArray = [1, , 3];
console.log(denseArray[1]); // => undefined
console.log(sparseArray[1]); // => undefined 
```

可以使用的区别方法是`Object.hasOwn`静态方法。 使用`Object.hasOwn`静态方法，可以确定数组对象在指定索引处是否存在元素。

```
const denseArray = [1, undefined, 3];
const sparseArray = [1, , 3];
// 要素自体は存在し、その値が`undefined`
console.log(Object.hasOwn(denseArray, 1)); // => true
// 要素自体が存在しない
console.log(Object.hasOwn(sparseArray, 1)); // => false 
```

## [](#search-element)*从数组中搜索元素*

*主要有以下三种目的来搜索数组中的元素。

+   如果需要该元素的索引

+   如果需要元素本身

+   如果需要该元素是否包含在数组中的布尔值

由于数组提供了相应的方法，因此可以按目的查看。

### [](#indexof)*获取索引*

*如果想知道指定元素在数组中的位置，可以使用 Array 的`indexOf`方法或`findIndex`方法^([ES2015])。 位置的概念称为**索引**（`index`），因此方法名称中也包含`index`。

下面的代码使用 Array 的`indexOf`方法，从数组中检索字符串`"JavaScript"`的索引。 `indexOf`方法从头开始搜索与参数和严格等式操作符（`===`）匹配的元素，并返回匹配元素的索引；如果没有匹配的元素，则返回`-1`。 与`indexOf`方法相对应的是`lastIndexOf`方法，它从末尾开始搜索匹配结果。

```
const array = ["Java", "JavaScript", "Ruby", "JavaScript"];
// 先頭から探索して最初に見つかった"JavaScript"のインデックスを取得
const indexOfJS = array.indexOf("JavaScript");
console.log(indexOfJS); // => 1
// 末尾から探索して最初に見つかった"JavaScript"のインデックスを取得
const lastIndexOfJS = array.lastIndexOf("JavaScript");
console.log(lastIndexOfJS); // => 3
console.log(array[indexOfJS]); // => "JavaScript"
console.log(array[lastIndexOfJS]); // => "JavaScript"
// "JS" という要素はないため `-1` が返される
console.log(array.indexOf("JS")); // => -1
console.log(array.lastIndexOf("JS")); // => -1 
```

`indexOf`方法可以发现数组中的原始元素，但如果对象具有相同的属性，则会被视为不同的对象。 看下面的代码，具有相同属性的不同对象无法使用`indexOf`方法找到。 这是因为具有不同引用的对象之间即使使用`===`比较也不会相等。

```
const obj = { key: "value" };
const array = ["A", "B", obj];
console.log(array.indexOf({ key: "value" })); // => -1
// リテラルは新しいオブジェクトを作るため、異なるオブジェクトだと判定される
console.log(obj === { key: "value" }); // => false
// 等価のオブジェクトを検索してインデックスを返す
console.log(array.indexOf(obj)); // => 2 
```

因此，如果想要找到不同对象但值相同的对象，则可以使用 Array 的`findIndex`方法。 将数组的每个元素作为回调函数传递给`findIndex`方法，与`indexOf`方法不同，可以自由编写测试代码。 这样，就可以从数组中找到具有相同属性值的元素，并获取其索引。

```
// colorプロパティを持つオブジェクトの配列
const colors = [
    { "color": "red" },
    { "color": "green" },
    { "color": "blue" }
];
// `color`プロパティが"blue"のオブジェクトのインデックスを取得
const indexOfBlue = colors.findIndex((obj) => {
    return obj.color === "blue";
});
console.log(indexOfBlue); // => 2
console.log(colors[indexOfBlue]); // => { "color": "blue" } 
```

Array 的`findIndex`方法也有对应的`findLastIndex`方法^([ES2023])，`findLastIndex`方法返回从末尾搜索的结果。 如下，`findIndex`返回匹配条件的第一个元素的索引，而`findLastIndex`返回最后一个元素的索引。

```
// dateとcountプロパティを持つオブジェクトの配列
const records = [
    { date: "2020/12/1", count: 5 },
    { date: "2020/12/2", count: 11 },
    { date: "2020/12/3", count: 9 },
    { date: "2020/12/4", count: 12 },
    { date: "2020/12/5", count: 3 }
];
// 10より大きい`count`プロパティを持つ最初のオブジェクトのインデックスを取得
const firstRecordIndex = records.findIndex((record) => {
    return record.count > 10;
});
// 10より大きい`count`プロパティを持つ最後のオブジェクトのインデックスを取得
const lastRecordIndex = records.findLastIndex((record) => {
    return record.count > 10;
});
console.log(firstRecordIndex); // => 1
console.log(records[firstRecordIndex]); // => { date: "2020/12/2", count: 11 }
console.log(lastRecordIndex); // => 3
console.log(records[lastRecordIndex]); // => { date: "2020/12/4", count: 12 } 
```

### [](#find)*获取匹配条件的元素*

*作为从数组中获取元素的方法，也可以使用索引。 只需使用`findIndex`方法获取索引，然后使用该索引访问数组即可。

但是，在使用`findIndex`方法获取元素的情况下，不清楚是要获取索引还是要获取元素本身。

要更明确地表示需要元素本身，可以使用 Array 的`find`方法^([ES2015])。 将测试函数作为回调函数传递给`find`方法，`find`方法的返回值将是该元素本身，如果元素不存在，则返回`undefined`。

```
// colorプロパティを持つオブジェクトの配列
const colors = [
    { "color": "red" },
    { "color": "green" },
    { "color": "blue" }
];
// `color`プロパティが"blue"のオブジェクトを取得
const blueColor = colors.find((obj) => {
    return obj.color === "blue";
});
console.log(blueColor); // => { "color": "blue" }
// 該当する要素がない場合は`undefined`を返す
const whiteColor = colors.find((obj) => {
    return obj.color === "white";
});
console.log(whiteColor); // => undefined 
```

也有与`find`方法对应的`findLast`方法^([ES2023])，`findLast`方法返回从末尾搜索的结果。 如下，`find`返回第一个匹配条件的元素，而`findLast`返回最后一个元素。

```
// dateとcountプロパティを持つオブジェクトの配列
const records = [
    { date: "2020/12/1", count: 5 },
    { date: "2020/12/2", count: 11 },
    { date: "2020/12/3", count: 9 },
    { date: "2020/12/4", count: 12 },
    { date: "2020/12/5", count: 3 }
];
// 10より大きい`count`プロパティを持つ最初のオブジェクトを取得
const firstRecord = records.find((record) => {
    return record.count > 10;
});
// 10より大きい`count`プロパティを持つ最後のオブジェクトを取得
const lastRecord = records.findLast((record) => {
    return record.count > 10;
});
console.log(firstRecord); // => { date: "2020/12/2", count: 11 }
console.log(lastRecord); // => { date: "2020/12/4", count: 12 } 
```

### [](#slice)*获取指定范围的元素*

*作为从数组中获取指定范围元素的方法，可以使用 Array 的`slice`方法。 `slice`方法从第一个参数的起始位置到第二个参数的结束位置（不包括结束位置的元素）提取一个新数组。 第二个参数是可选的，如果省略，则返回包含从数组末尾到末尾的元素的新数组。

```
const array = ["A", "B", "C", "D", "E"];
// インデックス1から4まで(4の要素は含まない)の範囲を取り出す
console.log(array.slice(1, 4)); // => ["B", "C", "D"]
// 第二引数を省略した場合は、第一引数から末尾の要素までを取り出す
console.log(array.slice(1)); // => ["B", "C", "D", "E"]
// マイナスを指定すると後ろから数えた位置となる
console.log(array.slice(-1)); // => ["E"]
// 第一引数と第二引数が同じ場合は、空の配列を返す
console.log(array.slice(1, 1)); // => []
// 第一引数 > 第二引数の場合、常に空配列を返す
console.log(array.slice(4, 1)); // => [] 
```

将`slice`方法与参数关联的关系如下图所示。

```
 +-----+-----+-----+-----+-----+
 | "A" | "B" | "C" | "D" | "E" |
 +-----+-----+-----+-----+-----+
 0     1     2     3     4     5
-5    -4    -3    -2    -1 
```

### [](#get-boolean)*获取布尔值*

*最后，让我们看看如何知道指定的元素是否包含在数组中。 如果可以获取索引或元素，则可以知道该元素是否包含在数组中。

但是，如果只想知道指定元素是否**仅仅**包含在其中，Array 的`findIndex`方法或`find`方法就具有过多的功能。 这使得阅读代码的人无法明确获取的索引或元素的用途。

下面的代码使用 Array 的`indexOf`方法来检查数组中是否包含指定的元素。 将`indexOf`方法的结果分配给`indexOfJS`，但除了检查是否包含外，没有其他用途。 必须完全阅读代码，才能明确意图，这导致代码的可读性不佳。

```
const array = ["Java", "JavaScript", "Ruby"];
// `indexOf`メソッドは含まれていないときのみ`-1`を返すことを利用
const indexOfJS = array.indexOf("JavaScript");
if (indexOfJS !== -1) {
    console.log("配列にJavaScriptが含まれている");
    // ... いろいろな処理 ...
    // `indexOfJS`は、含まれているのかの判定以外には利用してない
} 
```

因此，可以使用 ES2016 引入的 Array 的`includes`方法^([ES2016])。 Array 的`includes`方法判断数组是否包含指定元素。 由于`includes`方法返回布尔值，因此与使用`indexOf`方法相比，意图更明确。 在前述代码中，应该使用`includes`方法，如下所示。

```
const array = ["Java", "JavaScript", "Ruby"];
// `includes`は含まれているなら`true`を返す
if (array.includes("JavaScript")) {
    console.log("配列にJavaScriptが含まれている");
} 
```

`includes`方法与`indexOf`方法类似，不能找到具有相同值但不同对象的对象。 要使用类似于 Array 的`find`方法的测试回调函数来获取布尔值，可以使用 Array 的`some`方法。

Array 的`some`方法会检查回调函数是否有匹配的元素，如果有则返回`true`，否则返回`false`（参见“循环和迭代”章节）。

```
// colorプロパティを持つオブジェクトの配列
const colors = [
    { "color": "red" },
    { "color": "green" },
    { "color": "blue" }
];
// `color`プロパティが"blue"のオブジェクトがあるかどうか
const isIncludedBlueColor = colors.some((obj) => {
    return obj.color === "blue";
});
console.log(isIncludedBlueColor); // => true 
```

## [](#add-and-delete)*添加和删除*

*由于数组是可变长度的，因此可以添加或删除元素到创建后的数组。

若要将元素添加到数组末尾，可以使用 Array 的`push`。 反之，要从末尾删除元素，可以使用 Array 的`pop`。

```
const array = ["A", "B", "C"];
array.push("D"); // "D"を末尾に追加
console.log(array); // => ["A", "B", "C", "D"]
const poppedItem = array.pop(); // 最末尾の要素を削除し、その要素を返す
console.log(poppedItem); // => "D"
console.log(array); // => ["A", "B", "C"] 
```

要素を配列の先頭へ追加するにはArrayの`unshift`が利用できます。 一方、配列の先頭から要素を削除するにはArrayの`shift`が利用できます。

```
const array = ["A", "B", "C"];
array.unshift("S"); // "S"を先頭に追加
console.log(array); // => ["S", "A", "B", "C"]
const shiftedItem = array.shift(); // 先頭の要素を削除
console.log(shiftedItem); // => "S"
console.log(array); // => ["A", "B", "C"] 
```

## [](#concat)*配列同士を結合*

*Arrayの`concat`メソッドを使うことで配列と配列を結合した新しい配列を作成できます。

```
const array = ["A", "B", "C"];
const newArray = array.concat(["D", "E"]);
console.log(newArray); // => ["A", "B", "C", "D", "E"] 
```

また、`concat`メソッドは配列だけではなく任意の値を要素として結合できます。

```
const array = ["A", "B", "C"];
const newArray = array.concat("新しい要素");
console.log(newArray); // => ["A", "B", "C", "新しい要素"] 
```

## [](#spread)*[ES2015] 配列の展開*

*`...`（Spread 構文）を使うことで、配列リテラル中に既存の配列を展開できます。

次のコードでは、配列リテラルの末尾に配列を展開しています。 これは、Arrayの`concat`メソッドで配列同士を結合するのと同じ結果になります。

```
const array = ["A", "B", "C"];
// Spread 構文を使った場合
const newArray = ["X", "Y", "Z", ...array];
// concatメソッドの場合
const newArrayConcat = ["X", "Y", "Z"].concat(array);
console.log(newArray); // => ["X", "Y", "Z", "A", "B", "C"]
console.log(newArrayConcat); // => ["X", "Y", "Z", "A", "B", "C"] 
```

Spread 構文は、`concat`メソッドとは異なり、配列リテラル中の任意の位置に配列を展開できます。 そのため、次のように要素の途中に配列を展開できます。

```
const array = ["A", "B", "C"];
const newArray = ["X", ...array, "Z"];
console.log(newArray); // => ["X", "A", "B", "C", "Z"] 
```

## [](#flat)*[ES2019] 配列をフラット化*

*Arrayの`flat`メソッド^([ES2019])を使うことで、多次元配列をフラットな配列に変換できます。 引数を指定しなかった場合は1 段階のみのフラット化ですが、引数に渡す数値でフラット化する深さを指定できます。 配列をすべてフラット化する場合には、無限を意味する`Infinity`を値として渡すことで実現できます。

```
const array = [[["A"], "B"], "C"];
// 引数なしは1を指定した場合と同じ
console.log(array.flat()); // => [["A"], "B", "C"]
console.log(array.flat(1)); // => [["A"], "B", "C"]
console.log(array.flat(2)); // => ["A", "B", "C"]
// すべてをフラット化するにはInfinityを渡す
console.log(array.flat(Infinity)); // => ["A", "B", "C"] 
```

また、Arrayの`flat`メソッドは必ず新しい配列を作成して返すメソッドです。 そのため、これ以上フラット化できない配列をフラット化しても、同じ要素を持つ新しい配列を返します。

```
const array = ["A", "B", "C"];
console.log(array.flat()); // => ["A", "B", "C"] 
```

## [](#delete-element)*配列から要素を削除*

*### [](#splice)*`Array.prototype.splice`*

*要删除数组的开头或结尾元素，可以使用 Array 的`shift`方法或`pop`方法。但是，无法删除任意索引处的元素。要删除任意索引处的元素，请使用 Array 的`splice`方法。*

使用`splice`方法可以自动填充删除的元素。`splice`方法可以从指定的索引处删除指定数量的元素，并在必要时添加元素。

```
const array = [];
array.splice(インデックス, 削除する要素数);
// 削除と同時に要素の追加もできる
array.splice(インデックス, 削除する要素数, ...追加する要素); 
```

例如，要删除索引为`1`的元素，需要指定从索引`1`开始删除一个元素。这时，删除的元素会自动填充，因此不会变成稀疏数组。

```
const array = ["a", "b", "c"];
// 1 番目から1つの要素("b")を削除
array.splice(1, 1);
console.log(array); // => ["a", "c"]
console.log(array.length); // => 2
console.log(array[1]); // => "c"
// すべて削除
array.splice(0, array.length);
console.log(array.length); // => 0 
```

### [](#assign-to-length)*`length`属性赋值*

*可以通过 Array 的`splice`方法删除所有元素，也可以通过将`length`属性赋值为元素数的方法来实现。*

```
const array = [1, 2, 3];
array.length = 0; // 配列を空にする
console.log(array); // => [] 
```

将`length`属性赋值为元素数时，数组会被截断到该元素数。也就是说，将`length`属性赋值为`0`时，索引为`0`及以上的所有元素都会被删除。

### [](#assign-empty-array)*空数组赋值*

*最后，不是删除数组的元素，而是将新的空数组赋值给变量。下面的代码中，通过将空数组赋值给`array`变量，使`array`引用空数组。*

```
let array = [1, 2, 3];
console.log(array.length); // => 3
// 新しい配列で変数を上書き
array = [];
console.log(array.length); // => 0 
```

原本`array`变量引用的`[1, 2, 3]`不再被任何地方引用，将通过垃圾回收从内存中释放。

如果使用`const`声明了数组，则无法对变量进行重新赋值，因此这种方法不可用。因此，如果需要重新赋值，则需要使用`let`或`var`声明变量。

```
const array = [1, 2, 3];
console.log(array.length); // => 3
// `const`で宣言された変数には再代入できない
array = []; // TypeError: invalid assignment to const `array' が発生 
```

## [](#mutable-immutable)*破坏性方法与非破坏性方法*

*介绍过的修改数组的方法有破坏性方法和非破坏性方法。了解这两种方法的区别对于避免意外结果非常重要。*

破坏性方法（可变方法）是指修改数组对象本身并返回修改后的数组或修改部分的函数。非破坏性方法（不可变方法）是指在创建数组对象的副本后进行修改，并返回该副本的函数。

作为破坏性方法的例子，有向数组中添加元素的 Array 的`push`方法。`push`方法向`myArray`的数组本身添加元素。因此，`myArray`变量引用的数组会改变，所以这是一个破坏性方法。

```
const myArray = ["A", "B", "C"];
const result = myArray.push("D");
// `push`の返り値は配列ではなく、追加後の配列のlength
console.log(result); // => 4
// `myArray`が参照する配列そのものが変更されている
console.log(myArray); // => ["A", "B", "C", "D"] 
```

作为非破坏性方法的例子，有 Array 的`concat`方法。`concat`方法会复制`myArray`的数组并添加元素，然后返回这个新数组。由于`myArray`变量引用的数组没有改变，所以这是一个非破坏性方法。

```
const myArray = ["A", "B", "C"];
// `concat`の返り値は結合済みの新しい配列
const newArray = myArray.concat("D");
console.log(newArray); // => ["A", "B", "C", "D"]
// `myArray`は変更されていない
console.log(myArray); // => ["A", "B", "C"]
// `newArray`と`myArray`は異なる配列オブジェクト
console.log(myArray === newArray); // => false 
```

在 JavaScript 中，从名称上区分破坏性方法和非破坏性方法比较困难。此外，有些返回数组的破坏性方法，从返回值也无法区分。例如，Array 的`sort`方法返回排序后的数组，但它是破坏性方法。

下表中介绍的这些方法是破坏性方法。

| 方法名 | 返回值 |
| --- | --- |
| [`Array.prototype.pop`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/pop) | 数组的最后一个值 |
| [`Array.prototype.push`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/push) | 变更后的数组的 length |
| [`Array.prototype.splice`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/splice) | 包含被删除元素的数组 |
| [`Array.prototype.reverse`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/reverse) | 反转后的数组 |
| [`Array.prototype.shift`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/shift) | 数组的第一个值 |
| [`Array.prototype.sort`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) | 排序后的数组 |
| [`Array.prototype.unshift`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift) | 修改后的数组的长度 |
| [`Array.prototype.copyWithin`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/copyWithin)^([ES2015]) | 修改后的数组 |
| [`Array.prototype.fill`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/fill)^([ES2015]) | 修改后的数组 |

由于破坏性方法可能会产生意外的副作用，因此使用时需要注意。例如，如果想要提供一个从数组中删除特定索引处元素的函数`removeAtIndex`，就需要注意这一点。

```
// `array`の`index`番目の要素を削除した配列を返す関数
function removeAtIndex(array, index) { /* 実装 */ } 
```

在使用破坏性方法删除数组中的元素时，例如使用`splice`方法，将会影响作为参数传递的数组。 在这种情况下，因`removeAtIndex`函数具有副作用，因此对其具有破坏性的特性进行注释是有益的。

```
// `array`の`index`番目の要素を削除した配列を返す関数
// 引数の`array`は破壊的に変更される
function removeAtIndex(array, index) {
    array.splice(index, 1);
    return array;
}
const array = ["A", "B", "C"];
// `array`から1 番目の要素を削除した配列を取得
const newArray = removeAtIndex(array, 1);
console.log(newArray); // => ["A", "C"]
// `array`自体にも影響を与える
console.log(array); // => ["A", "C"] 
```

另一方面，非破坏性方法会创建数组的副本，因此对原始数组没有影响。 要将此`removeAtIndex`函数变为非破坏性函数，需要将接收到的数组复制一份，然后进行更改。

JavaScript 中没有`copy`方法本身，但常用的数组复制方法是使用 Array 的`slice`方法和`concat`方法。 当不带参数调用`slice`方法和`concat`方法时，它们会返回该数组的副本。

```
const myArray = ["A", "B", "C"];
// `slice`は`myArray`のコピーを返す - `myArray.concat()`でも同じ
const copiedArray = myArray.slice();
myArray.push("D");
console.log(myArray); // => ["A", "B", "C", "D"]
// `array`のコピーである`copiedArray`には影響がない
console.log(copiedArray); // => ["A", "B", "C"]
// コピーであるため参照は異なる
console.log(copiedArray === myArray); // => false 
```

通过对复制的数组进行更改，可以将`removeAtIndex`函数实现为非破坏性函数。 由于非破坏性函数不会对参数数组产生副作用，因此不需要提醒注释。

```
// `array`の`index`番目の要素を削除した配列を返す関数
function removeAtIndex(array, index) {
    // コピーを作成してから変更する
    const copiedArray = array.slice();
    copiedArray.splice(index, 1);
    return copiedArray;
}
const array = ["A", "B", "C"];
// `array`から1 番目の要素を削除した配列を取得
const newArray = removeAtIndex(array, 1);
console.log(newArray); // => ["A", "C"]
// 元の`array`には影響がない
console.log(array); // => ["A", "B", "C"] 
```

这样，JavaScript 数组中存在破坏性和非破坏性方法。 由于名称上的区分困难，并且为了避免副作用，人们通常会创建副本然后使用破坏性方法。

然而，ES2023 添加了一些变化以改进这种情况。 目前，仅有破坏性方法的`splice`、`reverse`和`sort`，现在也有了非破坏性版本，分别为`toSpliced`、`toReversed`和`toSorted`。

以`to`开头的这些非破坏性方法接收与破坏性方法相同的参数，但它们返回的是经过非破坏性更改的数组。 下面的`toSpliced`方法示例表明，由于它复制了数组然后进行了更改，所以原始数组`array`没有受到影响。

```
const array = ["A", "B", "C"];
// `toSpliced`は`array`を複製してから変更する
const newArray = array.toSpliced(1, 1);
console.log(newArray); // => ["A", "C"]
// コピー元の`array`には影響がない
console.log(array); // => ["A", "B", "C"] 
```

在前面的`removeAtIndex`函数实现中，我们使用`slice`方法复制数组，然后调用`splice`方法。 下面的代码使用`toSpliced`方法更简洁地实现了非破坏性的`removeAtIndex`函数。

```
// `array`の`index`番目の要素を削除した配列を返す関数
function removeAtIndex(array, index) {
    // コピーを作成してから変更する
    return array.toSpliced(index, 1);
}
const array = ["A", "B", "C"];
// `array`から1 番目の要素を削除した配列を取得
const newArray = removeAtIndex(array, 1);
console.log(newArray); // => ["A", "C"]
// 元の`array`には影響がない
console.log(array); // => ["A", "B", "C"] 
```

此外，ES2023 还引入了用于非破坏性地更改数组中指定索引元素的`with`方法。 `array[index] = value`的赋值操作本身是一种破坏性操作，而`with`方法是一种非破坏性方法，它复制数组然后更改指定索引的元素并返回该数组。

```
const array = ["A", "B", "C"];
// `array`の1 番目の要素を変更した配列を返す
const newArray = array.with(1, "B2");
console.log(newArray); // => ["A", "B2", "C"] 
```

下表总结了破坏性方法对应的非破坏性方法。

| 破坏性方法 | 非破坏性方法 |
| --- | --- |
| `array[index] = item` | [`Array.prototype.with`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/with)^([ES2023]) |
| [`Array.prototype.pop`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/pop) | [`array.slice(0, -1)`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)和[`array.at(-1)`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/at)^([ES2022]) |
| [`Array.prototype.push`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/push) | `[...array, item]`^([ES2015]) |
| [`Array.prototype.splice`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/splice) | [`Array.prototype.toSpliced`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toSpliced)^([ES2023]) |
| [`Array.prototype.reverse`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/reverse) | [`Array.prototype.toReversed`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toReversed)^([ES2023]) |
| [`Array.prototype.sort`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/sort) | [`Array.prototype.toSorted`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toSorted)^([ES2023]) |
| [`Array.prototype.shift`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/shift) | [`array.slice(1)`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/slice)和[`array.at(0)`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/at)^([ES2022]) |
| [`Array.prototype.unshift`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift) | `[item, ...array]`^([ES2015]) |
| [`Array.prototype.copyWithin`](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/copyWithin)^([ES2015]) | 无 |
| [Array.prototype.fill](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array/fill)^([ES2015]) | 无 |

破坏性方法虽然简单，但会改变原始数组，可能会导致意外的副作用和错误。 非破坏性方法需要谨慎使用，但不会改变原始数组，因此不会产生副作用。

因此，首先考虑使用非破坏性方法，如果不行的话，可以考虑使用破坏性方法。

## [](#array-iterate)*数组迭代方法*

*在“循环与迭代”章节中，我们部分介绍了对数组进行迭代处理的方法，现在让我们重新看一下相关的 Array 方法。 `forEach`、`map`、`filter`、`reduce`方法是迭代处理中经常使用的。 这些方法都需要一个回调函数作为参数，因此被称为高阶函数。*

### [](#array-foreach)*`Array.prototype.forEach`*

*Array 的`forEach`方法会按顺序将数组元素传递给回调函数，进行迭代处理的方法。*

回调函数的参数是`element, index, array`，它会从数组的第一个元素开始依次进行迭代处理。

```
const array = [1, 2, 3];
array.forEach((currentValue, index, array) => {
    console.log(currentValue, index, array);
});
// コンソールの出力
// 1, 0, [1, 2, 3]
// 2, 1, [1, 2, 3]
// 3, 2, [1, 2, 3] 
```

### [](#array-map)*`Array.prototype.map`*

*Array 的`map`方法会按顺序将数组元素传递给回调函数，并返回一个新数组，其中包含回调函数返回的值。 这是一个非破坏性方法，用于处理数组的每个元素。*

回调函数的参数是`element, index, array`，它会从数组的第一个元素开始依次进行迭代处理。 `map`方法的返回值是一个包含每个回调函数返回值的新数组。

```
const array = [1, 2, 3];
// 各要素に10を乗算した新しい配列を作成する
const newArray = array.map((currentValue, index, array) => {
    return currentValue * 10;
});
console.log(newArray); // => [10, 20, 30]
// 元の配列とは異なるインスタンス
console.log(array === newArray); // => false 
```

### [](#array-filter)*`Array.prototype.filter`*

*Array 的`filter`方法会按顺序将数组元素传递给回调函数，返回回调函数返回`true`的元素组成的新数组。 这是一个非破坏性方法，用于创建一个新数组，其中包含从数组中删除不需要的元素。*

回调函数的参数是`element, index, array`，它会从数组的第一个元素开始依次进行迭代处理。 `filter`方法的返回值是回调函数返回`true`的元素组成的新数组。

```
const array = [1, 2, 3];
// 奇数の値を持つ要素だけを集めた配列を返す
const newArray = array.filter((currentValue, index, array) => {
    return currentValue % 2 === 1;
});
console.log(newArray); // => [1, 3]
// 元の配列とは異なるインスタンス
console.log(array === newArray); // => false 
```

### [](#array-reduce)*`Array.prototype.reduce`*

*Array 的`reduce`方法会累加器和数组元素依次传递给回调函数，并返回一个累加器。 当你想要从数组中创建一个不包含数组的任意值时，可以使用它。*

与之前介绍的迭代处理方法不同，回调函数的参数是`accumulator, element, index, array`。 你可以在`reduce`方法的第二个参数中传递一个初始值给`accumulator`。

在下面的代码中，`reduce`方法将数组的每个元素相加并返回一个数字作为累加器的初始值。也就是说，它返回了数组元素的总和作为一个 Number 类型的值。

```
const array = [1, 2, 3];
// すべての要素を加算した値を返す
// accumulatorの初期値は`0`
const totalValue = array.reduce((accumulator, currentValue, index, array) => {
    return accumulator + currentValue;
}, 0);
// 0 + 1 + 2 + 3という式の結果が返り値になる
console.log(totalValue); // => 6 
```

传递给`reduce`方法的回调函数会被调用 3 次，每次都会传递以下参数，并产生以下结果。

|  | accumulator | currentValue | 返回的值 |
| --- | --- | --- | --- |
| 第一次调用 | 0 | 1 | 0 + 1 |
| 第二次调用 | 1 | 2 | 1 + 2 |
| 第三次调用 | 3 | 3 | 3 + 3 |

Array 的`reduce`方法稍微复杂一些，但它具有从数组创建非数组数据类型值的功能。 此外，由于`reduce`方法可以直接从数组返回 Number 类型的值，因此在声明了无法重新赋值的`const`变量`totalValue`。

使用`forEach`方法等迭代处理计算数组数字总和时，需要使用`let`声明的变量`totalValue`以便重新赋值。

```
const array = [1, 2, 3];
// 初期値は`0`
let totalValue = 0;
array.forEach((currentValue) => {
    totalValue += currentValue;
});
console.log(totalValue); // => 6 
```

因为使用`let`声明的变量可以重新赋值，所以变量的值可能会在意外的地方更改，从而导致错误。 因此，在可能的情况下，最好使用`const`声明变量。 在这种情况下，`reduce`方法非常有用。 另一方面，`reduce`方法的可读性不太好，这可能会导致代码意图不明确。

虽然`reduce`方法具有优点和可读性的折衷，但在使用时需要注意确保代码清晰易懂。 尤其是在处理`reduce`方法时，应该通过函数封装来明确处理意图。

```
const array = [1, 2, 3];
function sum(array) {
    return array.reduce((accumulator, currentValue) => {
        return accumulator + currentValue;
    }, 0);
}
console.log(sum(array)); // => 6 
```

## [](#array-like)*[专栏] 类似数组对象*

*类似数组的对象称为**Array-like 对象**。Array-like 对象可以像数组一样通过索引进行访问，并且像数组一样具有`length`属性。但是，它不是数组的实例，因此指的是不具有数组原型方法的对象。*

| 功能 | 类似数组对象 | 数组 |
| --- | --- | --- |
| 索引访问（`array[0]`） | 可以 | 可以 |
| 长度（`array.length`） | 拥有 | 拥有 |
| Array 原型方法（如`forEach`方法等） | 有时没有 | 有 |

类似数组对象的例子包括`arguments`。 `arguments`对象是可以从`function`声明的函数中引用的变量。 `arguments`对象按顺序存储传递给函数的参数值，并且可以像数组一样访问参数。

```
function myFunc() {
    console.log(arguments[0]); // => "a"
    console.log(arguments[1]); // => "b"
    console.log(arguments[2]); // => "c"
    // 配列ではないため、配列のメソッドは持っていない
    console.log(typeof arguments.forEach); // => "undefined"
}
myFunc("a", "b", "c"); 
```

Array-like 对象还是数组可以通过使用`Array.isArray`方法来判断。 `Array-like`对象不是数组，所以结果始终为`false`。

```
function myFunc() {
    console.log(Array.isArray([1, 2, 3])); // => true
    console.log(Array.isArray(arguments)); // => false
}
myFunc("a", "b", "c"); 
```

Array-like 对象是一种看起来像数组但不是数组的对象。 使用`Array.from`方法^([ES2015])可以将 Array-like 对象转换为数组以进行处理。 一旦转换为数组，就可以使用 Array 方法。

```
function myFunc() {
    // Array-likeオブジェクトを配列へ変換
    const argumentsArray = Array.from(arguments);
    console.log(Array.isArray(argumentsArray)); // => true
    // 配列のメソッドを利用できる
    argumentsArray.forEach(arg => {
        console.log(arg);
    });
}
myFunc("a", "b", "c"); 
```

## [](#method-chain-and-high-order-function)*方法链与高阶函数*

*数组经常使用方法链的模式。 方法链是指对调用方法的返回值进行进一步的方法调用的模式。*

在下面的代码中，对 Array 的`concat`方法的返回值，也就是对数组进一步调用`concat`方法的方法链进行了处理。

```
const array = ["a"].concat("b").concat("c");
console.log(array); // => ["a", "b", "c"] 
```

将这段代码中的`concat`方法调用拆分开来可以更容易理解正在发生的事情。 `concat`方法的返回值是合并的新数组。在之前的方法链中，可以看到正在对该新数组进行进一步的值合并，使用`concat`方法。

```
// メソッドチェーンを分解した例
// 一時的な`abArray`という変数が増えている
const abArray = ["a"].concat("b");
console.log(abArray); // => ["a", "b"]
const abcArray = abArray.concat("c");
console.log(abcArray); // => ["a", "b", "c"] 
```

使用方法链可以使处理过程更简洁。 尽管使用方法链的最终处理结果与其他方式相同，但可以省略中间的临时变量。 在上面的示例中，方法链中省略了名为`abArray`的临时变量。

方法链不仅限于数组，但在数组中常见。 这是因为在显示数组数据时，通常需要将其处理为其他数据类型，如字符串或数字。 由于数组实现了许多返回数组的高阶函数，因此可以灵活处理数组。

以下代码定义了一个名为`ECMAScriptVersions`的数组，其中包含了 ECMAScript 的版本名称和发布年份。 假设我们要从该数组中提取`2000`年以前发行的 ECMAScript 版本名称列表。 为了实现这个目标，需要将“筛选出 2000 年以前的数据”和“从数据中提取`name`”这两个处理结合起来。

这两个处理可以通过 Array 的`filter`方法和`map`方法来实现。 使用`filter`方法从数组中筛选出在`2000`年之前的数据，然后使用`map`方法从每个元素中提取`name`属性。 由于这两种方法都返回数组，因此可以在方法链中连接它们。

```
// ECMAScriptのバージョン名と発行年
const ECMAScriptVersions = [
    { name: "ECMAScript 1", year: 1997 },
    { name: "ECMAScript 2", year: 1998 },
    { name: "ECMAScript 3", year: 1999 },
    { name: "ECMAScript 5", year: 2009 },
    { name: "ECMAScript 5.1", year: 2011 },
    { name: "ECMAScript 2015", year: 2015 },
    { name: "ECMAScript 2016", year: 2016 },
    { name: "ECMAScript 2017", year: 2017 },
];
// メソッドチェーンで必要な加工処理を並べている
const versionNames = ECMAScriptVersions
    // 2000 年以下のデータに絞り込み
    .filter(ECMAScript => ECMAScript.year <= 2000)
    // それぞれの要素から`name`プロパティを取り出す
    .map(ECMAScript => ECMAScript.name);
console.log(versionNames); // => ["ECMAScript 1", "ECMAScript 2", "ECMAScript 3"] 
```

使用方法链可以将由多个处理组成的处理看作一个整体处理。 过长的方法链会导致函数过长，难以阅读，但适度的方法链可以使处理过程更清晰。

## [](#conclusion)*总结*

*本章介绍了数组相关内容。

+   数组是一种可以存储有序元素的对象。

+   数组有破坏性方法和非破坏性方法。

+   数组具有用于执行迭代处理的高阶函数方法。

+   方法链利用数组方法返回数组的特性。

数组是 JavaScript 中经常使用的对象之一，有许多不同类型的方法。 由于本书未介绍所有方法，请参阅[有关数组的文档](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array)以获取更多详细信息。
