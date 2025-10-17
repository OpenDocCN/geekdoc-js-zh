# [ES2015] Map/Set

> 原文：[`jsprimer.net/basic/map-and-set/`](https://jsprimer.net/basic/map-and-set/)

JavaScriptでデータの集まり（コレクション）を扱うビルトインオブジェクトは、ObjectやArrayだけではありません。 この章では、ES2015で導入されたマップ型のコレクションである`Map`と、セット型のコレクションである`Set`について学びます。

## Map

*[Map](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Map)はマップ型のコレクションを扱うためのビルトインオブジェクトです。 マップとは、キーと値の組み合わせからなる抽象データ型です。 他のプログラミング言語の文脈では辞書やハッシュマップ、連想配列などと呼ばれることもあります。

### マップの作成と初期化

*`Map`オブジェクトを`new`することで、新しいマップを作成できます。 作成されたばかりのマップは何も持っていません。 そのため、マップが持つ要素の数を返す`size`プロパティは`0`を返します。

```
const map = new Map();
console.log(map.size); // => 0 
```

`Map`オブジェクトを`new`で初期化するときに、コンストラクタに初期値を渡せます。 コンストラクタ引数として渡せるのは**エントリー**の配列です。 エントリーとは、1つのキーと値の組み合わせを`[キー, 値]`という形式の配列で表現したものです。

次のコードでは、Mapに初期値となるエントリー（配列）の配列を渡しています。

```
const map = new Map([["key1", "value1"], ["key2", "value2"]]);
// 2つのエントリーで初期化されている
console.log(map.size); // => 2 
```

### 要素の追加と取り出し

*`Map`には新しい要素を`set`メソッドで追加でき、追加した要素を`get`メソッドで取り出せます。

`set`メソッドは特定のキーと値を持つ要素をマップに追加します。 ただし、同じキーで複数回`set`メソッドを呼び出した際は、後から追加された値で上書きされます。

`get`メソッドは特定のキーにひもづいた値を取り出します。 また、特定のキーにひもづいた値を持っているかを確認する`has`メソッドがあります。

```
const map = new Map();
// 新しい要素の追加
map.set("key", "value1");
console.log(map.size); // => 1
console.log(map.get("key")); // => "value1"
// 要素の上書き
map.set("key", "value2");
console.log(map.get("key")); // => "value2"
// キーの存在確認
console.log(map.has("key")); // => true
console.log(map.has("foo")); // => false 
```

`delete`メソッドはマップから要素を削除します。 `delete`メソッドに渡されたキーと、そのキーにひもづいた値がマップから削除されます。 また、マップが持つすべての要素を削除するための`clear`メソッドがあります。

```
const map = new Map();
map.set("key1", "value1");
map.set("key2", "value2");
console.log(map.size); // => 2
map.delete("key1");
console.log(map.size); // => 1
map.clear();
console.log(map.size); // => 0 
```

### マップの反復処理

*マップが持つ要素を列挙するメソッドとして、`forEach`、`keys`、`values`、`entries`があります。

`forEach`メソッドはマップが持つすべての要素を、マップへの挿入順に反復処理します。 コールバック関数には引数として値、キー、マップの3つが渡されます。 配列の`forEach`メソッドと似ていますが、インデックスの代わりにキーが渡されます。 配列はインデックスにより要素を特定しますが、マップはキーにより要素を特定するためです。

```
const map = new Map([["key1", "value1"], ["key2", "value2"]]);
const results = [];
map.forEach((value, key) => {
    results.push(`${key}:${value}`);
});
console.log(results); // => ["key1:value1","key2:value2"] 
```

`keys`メソッドはマップが持つすべての要素のキーを挿入順に並べた**Iterator**オブジェクトを返します。 同様に、`values`メソッドはマップが持つすべての要素の値を挿入順に並べたIteratorオブジェクトを返します。 これらの返り値はIteratorオブジェクトであって配列ではありません。 そのため、次の例のように`for...of`文で反復処理を行ったり、`Array.from`メソッドに渡して配列に変換して使ったりします。

```
const map = new Map([["key1", "value1"], ["key2", "value2"]]);
const keys = [];
// keysメソッドの返り値(Iterator)を反復処理する
for (const key of map.keys()) {
    keys.push(key);
}
console.log(keys); // => ["key1","key2"]
// keysメソッドの返り値(Iterator)から配列を作成する
const keysArray = Array.from(map.keys());
console.log(keysArray); // => ["key1","key2"] 
```

`entries`メソッドはマップが持つすべての要素をエントリーとして挿入順に並べたIteratorオブジェクトを返します。 先述のとおりエントリーは`[キー, 値]`の配列です。 そのため、配列の分割代入を使うとエントリーからキーと値を簡単に取り出せます。

```
const map = new Map([["key1", "value1"], ["key2", "value2"]]);
const entries = [];
for (const [key, value] of map.entries()) {
    entries.push(`${key}:${value}`);
}
console.log(entries); // => ["key1:value1","key2:value2"] 
```

また、マップ自身もiterableなオブジェクトなので、`for...of`文で反復処理できます。 マップを`for...of`文で反復したときは、すべての要素をエントリーとして挿入順に反復処理します。 つまり、`entries`メソッドの返り値を反復処理するときと同じ結果が得られます。

```
const map = new Map([["key1", "value1"], ["key2", "value2"]]);
const results = [];
for (const [key, value] of map) {
    results.push(`${key}:${value}`);
}
console.log(results); // => ["key1:value1","key2:value2"] 
```

### マップとしてのObjectとMap

*ES2015で`Map`が導入されるまで、JavaScriptにおいてマップ型を実現するために`Object`が利用されてきました。 何かをキーにして値にアクセスするという点で、`Map`と`Object`はよく似ています。 ただし、マップとしての`Object`にはいくつかの問題があります。

+   `Object`の`prototype`オブジェクトから継承されたプロパティによって、意図しないマッピングを生じる危険性がある

+   また、プロパティとしてデータを持つため、キーとして使えるのは文字列か`Symbol`に限られる

`Object`には`prototype`オブジェクトがあるため、いくつかのプロパティは初期化されたときから存在します。 `Object`をマップとして使うと、そのプロパティと同じ名前のキーを使おうとしたときに問題となります （詳細は「オブジェクト」の章の「プロパティの存在を確認する」を参照）。

たとえば`constructor`という文字列は`Object.prototype.constructor`プロパティと衝突してしまいます。 そのため`constructor`のような文字列をオブジェクトのキーに使うことで意図しないマッピングを生じる危険性があります。

```
const map = {};
// マップがキーを持つことを確認する
function has(key) {
    return typeof map[key] !== "undefined";
}
console.log(has("foo")); // => false
// Objectのプロパティが存在する
console.log(has("constructor")); // => true 
```

このマップとして使うオブジェクトの問題は、`Object`のインスタンスを`Object.create(null)`のように初期化して作ることで回避されてきました （詳細は「プロトタイプオブジェクト」の章の「`Object.prototype`を継承しないオブジェクト」を参照）。

ES2015では、これらの問題を根本的に解決する`Map`が導入されました。 `Map`はプロパティとは異なる仕組みでデータを格納します。 そのため、`Map`のプロトタイプが持つメソッドやプロパティとキーが衝突することはありません。 また、`Map`ではマップのキーとしてあらゆるオブジェクトを使えます。

ほかにも`Map`には次のような利点があります。

+   マップのサイズを簡単に知ることができる

+   マップが持つ要素を簡単に列挙できる

+   オブジェクトをキーにすると参照ごとに違うマッピングができる

たとえばショッピングカートのような仕組みを作るとき、次のように`Map`を使って商品のオブジェクトと注文数をマッピングできます。

```
// ショッピングカートを表現するクラス
class ShoppingCart {
    constructor() {
        // 商品とその数を持つマップ
        this.items = new Map();
    }
    // カートに商品を追加する
    addItem(item) {
        // `item`がない場合は`undefined`を返すため、Nullish coalescing 演算子(`??`)を使いデフォルト値として`0`を設定する
        const count = this.items.get(item) ?? 0;
        this.items.set(item, count + 1);
    }
    // カート内の合計金額を返す
    getTotalPrice() {
        return Array.from(this.items).reduce((total, [item, count]) => {
            return total + item.price * count;
        }, 0);
    }
    // カートの中身を文字列にして返す
    toString() {
        return Array.from(this.items).map(([item, count]) => {
            return `${item.name}:${count}`;
        }).join(",");
    }
}
const shoppingCart = new ShoppingCart();
// 商品一覧
const shopItems = [
    { name: "みかん", price: 100 },
    { name: "リンゴ", price: 200 },
];

// カートに商品を追加する
shoppingCart.addItem(shopItems[0]);
shoppingCart.addItem(shopItems[0]);
shoppingCart.addItem(shopItems[1]);

// 合計金額を表示する
console.log(shoppingCart.getTotalPrice()); // => 400
// カートの中身を表示する
console.log(shoppingCart.toString()); // => "みかん:2,リンゴ:1" 
```

当使用`Object`作为映射时，许多问题可以通过使用`Map`对象来解决，但`Map`并不总是`Object`的替代品。作为映射的`Object`具有以下优点。

+   由于存在字面量表达式，因此易于创建

+   由于具有规定的 JSON 表示，因此可以使用`JSON.stringify`函数轻松地将`Object`转换为 JSON。

+   不论是原生 API 还是外部库，许多函数都设计为以`Object`作为映射传递。

在以下示例中，在接收到登录表单的 submit 事件后，向服务器发送 POST 请求。为了向服务器发送 JSON 字符串，使用`JSON.stringify`函数。因此，创建了一个`Object`映射，其中包含表单的输入内容。在这种情况下，使用`Object`作为这种简单映射是合适的。

```
// URLとObjectのマップを受け取ってPOSTリクエストを送る関数
function sendPOSTRequest(url, data) {
    // fetchを使ってPOSTリクエストを送る
    fetch(url, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(data),
    }).catch((error) => {
        console.error(error);
    });
}

// formのsubmitイベントを受け取る関数
function onLoginFormSubmit(event) {
    const form = event.target;
    const data = {
        userName: form.elements.userName,
        password: form.elements.password,
    };
    sendPOSTRequest("/api/login", data);
} 
```

### *WeakMap*

*[WeakMap](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)与`Map`一样，是用于处理映射的内置对象。与`Map`的不同之处在于，它使用**弱引用**（Weak Reference）来持有键。

[弱引用](https://ja.wikipedia.org/wiki/%E5%BC%B1%E3%81%84%E5%8F%82%E7%85%A7)是一种特殊的引用，它不会阻止垃圾收集（GC）释放对象。通常，只有当对象不再被任何地方引用时，GC 才能从内存中释放对象。弱引用例外地，即使存在对该对象的弱引用，GC 也可以释放该对象。

因此，弱引用用于防止因持续引用不再需要的对象而导致的内存泄漏。由于`WeakMap`中不再需要的键及其关联的值会自动删除，因此不会引起内存泄漏。

在以下代码中，首先将`obj`设置为`{}`，然后在`WeakMap`中将该`obj`作为键并设置值（`"value"`）。接下来，将`obj`赋值为另一个值（此处为`null`），则`obj`最初引用的`{}`值将不再被任何地方引用。此时，`WeakMap`持有对`{}`的弱引用，但弱引用不会阻止 GC，因此`{}`作为不再需要的值将被 GC 从内存中释放。

同时，`WeakMap`可以丢弃与已释放对象（`{}`）关联的值（`"value"`）。但是，实际何时从内存中释放取决于 JavaScript 引擎的实现。

```
const map = new WeakMap();
// キーとなるオブジェクト
let obj = {};
// {} への参照をキーに値をセットする
map.set(obj, "value");
// {} への参照を破棄する
obj = null;
// GCが発生するタイミングでWeakMapから値が破棄される 
```

`WeakMap`与`Map`相似，但不是可迭代的。因此，不存在用于列举键的`keys`方法或返回数据数量的`size`属性。此外，由于它使用弱引用持有键，因此可以作为键使用的只有引用类型的对象和`Symbol`（es2023）。

`WeakMap`的主要用途之一是将私有值存储在类中。将`this`（类实例）作为`WeakMap`的键，可以保持无法从实例外部访问的值。此外，当类实例不再被引用时，这些值将自动释放。

在以下代码中，使用`WeakMap`管理触发事件的监听器函数（事件监听器）。事件监听器是指当事件发生时被调用的函数。如果使用`Map`实现此映射，则事件监听器将一直保留在内存中，直到显式删除。在这里使用`WeakMap`，则传递给`addListener`方法的`listener`在`EventEmitter`实例不再被引用时将自动释放。

```
// イベントリスナーを管理するマップ
const listenersMap = new WeakMap();

class EventEmitter {
    addListener(listener) {
        // this(インスタンス)にひもづいたリスナーの配列を取得する
        const listeners = listenersMap.get(this) ?? [];
        const newListeners = listeners.concat(listener);
        // this をキーにして、新しいリスナーの配列をセットする
        listenersMap.set(this, newListeners);
    }
    // ...EventEmitterには他にもメソッドがあるが省略...
}

// `event`は`EventEmitter`のインスタンスへの参照をもつ
let event = new EventEmitter();
// `EventEmitter`のインスタンスへイベントリスナーを追加する
event.addListener(() => {
    // `EventEmitter`のインスタンスに紐づくイベントリスナーの処理
});
// `event`へ`null`を代入することで、`EventEmitter`のインスタンスへの参照がなくなる
// インスタンスがどこからも参照されなくなったため、紐づいていたイベントリスナーが自動的に解放される
event = null; 
```

此外，它还常用于临时保存从某个对象计算出的结果。在以下示例中，保存了 HTML 元素的高度计算结果，以避免在第二次及以后进行相同的计算。

```
const cache = new WeakMap();

function getHeight(element) {
    if (cache.has(element)) {
        return cache.get(element);
    }
    const height = element.getBoundingClientRect().height;
    // elementオブジェクトに対して高さをひもづけて保存している
    cache.set(element, height);
    return height;
} 
```

### *[专栏] 键的等价性与 NaN*

*`Map`中设置值时可以使用任何对象作为键。在这种情况下，该映射是否已经包含特定的键，即插入和覆盖的判断基本上与`===`运算符相同。

然而，键为`NaN`的处理方式例外。在`Map`中，键的比较中，`NaN`总是被视为等价的。这种行为被称为[Same-value-zero](https://developer.mozilla.org/ja/docs/Web/JavaScript/Equality_comparisons_and_sameness#Same-value-zero_equality)算法。

在以下代码中，`NaN`之间的`===`比较结果为`false`，而`Map`的键中`NaN`之间的比较结果是一致的。

```
const map = new Map();
map.set(NaN, "value");
// NaNは===で比較した場合は常にfalse
console.log(NaN === NaN); // => false
// MapはNaN 同士を比較できる
console.log(map.has(NaN)); // => true
console.log(map.get(NaN)); // => "value" 
```

## *Set*

*[Set](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Set)是用于处理集合的内置对象。集合是指不包含重复值的集合。`Set`可以列举添加的值，因此它类似于不包含重复值的数组，常用于保证值的唯一性。然而，与数组不同，集合没有顺序，也不能通过索引访问元素。

### *创建和初始化集合*

*通过`new`创建`Set`对象可以创建一个新的集合。创建的集合一开始没有任何内容。因此，返回集合中元素数量的`size`属性将返回 0。

```
const set = new Set();
console.log(set.size); // => 0 
```

当使用`new`初始化`Set`对象时，可以将初始值传递给构造函数。构造函数参数可以是可迭代对象。

在以下代码中，将可迭代对象数组作为初始值传递。此外，`Set`确保不包含重复的相同值，因此相同的值只存储一次。

```
// "value2"が重複するため、片方は無視される
const set = new Set(["value1", "value2", "value2"]);
// セットのサイズは2になる
console.log(set.size); // => 2 
```

### **值的添加和提取**

*要向创建的集合中添加值，可以使用`add`方法。正如之前所述，集合保证不包含重复的值。因此，当将已存在于集合中的值传递给`add`方法时，它将被忽略。

此外，还有一个`has`方法可以用来检查集合是否包含特定的值。

```
const set = new Set();
// 値の追加
set.add("a");
console.log(set.size); // => 1
// 重複する値は追加されない
set.add("a");
console.log(set.size); // => 1
// 値の存在確認
console.log(set.has("a")); // => true
console.log(set.has("b")); // => false 
```

要从集合中删除值，可以使用`delete`方法。`delete`方法会删除传递给它的值。此外，还有`clear`方法可以删除集合中所有的值。

```
const set = new Set();
set.add("a");
set.add("b");
console.log(set.size); // => 2
set.delete("a");
console.log(set.size); // => 1
set.clear();
console.log(set.size); // => 0 
```

### **集合的遍历**

*要遍历集合中的值，可以使用`forEach`方法。`forEach`方法会按照集合的插入顺序遍历集合中所有的元素。

```
const set = new Set(["a", "b"]);
const results = [];
set.forEach((value) => {
    results.push(value);
});
console.log(results); // => ["a","b"] 
```

集合中创建迭代器对象的`keys`、`values`、`entries`方法类似于`Map`，但集合中没有与`Map`中的键相当的元素。因此，`keys`方法实际上是`values`方法的别名，它返回一个按插入顺序列出集合中所有值的迭代器对象。同样，`entries`方法返回一个按插入顺序列出`[值, 值]`形式条目的迭代器对象。

```
const set = new Set(["a", "b"]);
// keysで列挙
const keysResults = [];
for (const value of set.keys()) {
    keysResults.push(value);
}
console.log(keysResults); // => ["a","b"]
// entriesで列挙
const entryResults = [];
for (const entry of set.entries()) {
    // entryは[値, 値]という配列
    entryResults.push(entry);
}
console.log(entryResults); // => [["a","a"], ["b", "b"]] 
```

由于`Set`对象自身也是一个可迭代的对象，因此可以使用`for...of`语句进行遍历。使用`for...of`语句遍历`Set`对象时，值也会按照集合的插入顺序被取出。由于`keys`、`values`、`entries`方法可以获取相同的信息，因此基本上可以通过遍历`Set`对象本身来代替。

```
const set = new Set(["a", "b"]);
const results = [];
for (const value of set) {
    results.push(value);
}
console.log(results); // => ["a","b"] 
```

### **WeakSet**

*[WeakSet](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)*是一个包含有值引用的集合。`WeakSet`与`Set`类似，但它不是可迭代的，因此不能遍历添加的值。换句话说，`WeakSet`只能进行值的添加和删除、存在性检查等操作。可以说它是一个专注于检查数据唯一性的集合。

此外，由于**WeakSet**的特性是只包含有值的引用，因此可以作为其值的只能是引用类型的对象和**Symbol**(es2023)。

## **总结**

本章我们学习了**Map**和**Set**。

+   `Map`是一个内置对象，用于处理由键值对组成的集合。

+   **Map**的键可以避免与原型对象的属性和名称冲突，从而避免意外的映射。

+   `WeakMap`是一个与键为弱引用的`Map`类似的内置对象。

+   `Set`是一个内置对象，用于处理不包含重复值的有序集合。

+   `WeakSet`是一个与值为弱引用的`Set`类似的内置对象。

> ^(es2023). ES2023 中已经修改了规范，使得**Symbol**也可以被处理。 ↩*************
