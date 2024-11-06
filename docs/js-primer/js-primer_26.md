# クラス

> 原文：[`jsprimer.net/basic/class/`](https://jsprimer.net/basic/class/)

「クラス」と一言にいってもさまざまであるため、こ��では**構造**、**動作**、**状態**を定義できるものを指すことにします。 また、この章では概念を示す場合は**クラス**と呼び、クラスに関する構文（記述するコード）のことを`class`構文と呼びます。

**クラス**とは**動作**や**状態**を定義した**構造**です。 クラスからはインスタンスと呼ばれるオブジェクトを作成でき、インスタンスはクラスに定義した**動作**を継承し、**状態**は動作によって変化します。 とても抽象的なことに思えますが、これは今までオブジェクトや関数を使って表現してきたものです。 JavaScriptではES2015より前までは`class`構文はなく、関数を使ってクラスのようなものを表現して扱っていました。

ES2015でクラスを表現するための`class`構文が導入されましたが、この`class`構文で定義したクラスは関数オブジェクトの一種です。 `class`構文ではプロトタイプベースの継承の仕組みを使って関数でクラスを表現しています。 そのため、`class`構文はクラスを作るための関数定義や継承をパターン化した書き方と言えます。

また、関数の定義方法として関数宣言文と関数式があるように、クラスにもクラス宣言文とクラス式があります。 このように関数とクラスは似ている部分が多いです。

この章では、`class`構文でのクラスの定義や継承、クラスの性質について学んでいきます。

## [](#class-declaration)*クラスの定義*

*クラスを定義するには`class`構文を使います。 クラスの定義方法にはクラス宣言文とクラス式があります。

まずは、クラス宣言文によるクラスの定義方法を見ていきます。

クラス宣言文では`class`キーワードを使い、`class クラス名{ }`のようにクラスの**構造**を定義できます。

クラスは必ずコンストラクタを持ち、`constructor`という名前のメソッドとして定義します。 コンストラクタとは、そのクラスからインスタンスを作成する際にインスタンスに関する**状態**の初期化を行うメソッドです。 `constructor`メソッドに定義した処理は、クラスをインスタンス化したときに自動的に呼び出されます。

```
class MyClass {
    constructor() {
        // コンストラクタ関数の処理
        // インスタンス化されるときに自動的に呼び出される
    }
} 
```

もうひとつの定義方法であるクラス式は、クラスを値として定義する方法です。 クラス式ではクラス名を省略できます。これは関数式における無名関数と同じです。

```
const MyClass = class MyClass {
    constructor() {}
};

const AnonymousClass = class {
    constructor() {}
}; 
```

コンストラクタ関数内で、何も処理がない場合はコンストラクタの記述を省略できます。 省略した場合でも自動的に空のコンストラクタが定義されるため、クラスにはコンストラクタが必ず存在します。

```
class MyClassA {
    constructor() {
        // コンストラクタの処理が必要なら書く
    }
}
// コンストラクタの処理が不要な場合は省略できる
class MyClassB {
} 
```

## [](#class-instance)*クラスのインスタンス化*

*クラスは`new`演算子でインスタンスであるオブジェクトを作成できます。 `class`構文で定義したクラスからインスタンスを作成することを**インスタンス化**と呼びます。 あるインスタンスが指定したクラスから作成されたものかを判定するには`instanceof`演算子が利用できます。

```
class MyClass {
}
// `MyClass`をインスタンス化する
const myClass = new MyClass();
// 毎回新しいインスタンス(オブジェクト)を作成する
const myClassAnother = new MyClass();
// それぞれのインスタンスは異なるオブジェクト
console.log(myClass === myClassAnother); // => false
// クラスのインスタンスかどうかは`instanceof`演算子で判定できる
console.log(myClass instanceof MyClass); // => true
console.log(myClassAnother instanceof MyClass); // => true 
```

このままでは何も処理がない空のクラスなので、値を持ったクラスを定義してみましょう。

クラスではインスタンスの初期化処理をコンストラクタ関数で行います。 コンストラクタ関数は`new`演算子でインスタンス化する際に自動的に呼び出されます。 コンストラクタ関数内での`this`はこれから新しく作るインスタンスオブジェクトとなります。

次のコードでは、`x`座標と`y`座標の値を持つ`Point`というクラスを定義しています。 コンストラクタ関数（`constructor`）の中でインスタンスオブジェクト（`this`）の`x`と`y`プロパティに値を代入して初期化しています。

```
class Point {
    // コンストラクタ関数の仮引数として`x`と`y`を定義
    constructor(x, y) {
        // コンストラクタ関数における`this`はインスタンスを示すオブジェクト
        // インスタンスの`x`と`y`プロパティにそれぞれ値を設定する
        this.x = x;
        this.y = y;
    }
} 
```

この`Point`クラスのインスタンスを作成するには`new`演算子を使います。 `new`演算子には関数呼び出しと同じように引数を渡すことができます。 `new`演算子の引数はクラスの`constructor`メソッド（コンストラクタ関数）の仮引数に渡されます。 そして、コンストラクタの中ではインスタンスオブジェクト（`this`）の初期化処理を行います。

```
class Point {
    // 2\. コンストラクタ関数の仮引数として`x`には`3`、`y`には`4`が渡る
    constructor(x, y) {
        // 3\. インスタンス(`this`)の`x`と`y`プロパティにそれぞれ値を設定する
        this.x = x;
        this.y = y;
        // コンストラクタではreturn 文は書かない
    }
}

// 1\. コンストラクタを`new`演算子で引数とともに呼び出す
const point = new Point(3, 4);
// 4\. `Point`のインスタンスである`point`の`x`と`y`プロパティには初期化された値が入る
console.log(point.x); // => 3
console.log(point.y); // => 4 
```

このようにクラスからインスタンスを作成するには必ず`new`演算子を使います。

一方、クラスは通常の関数として呼ぶことができません。 これは、クラスのコンストラクタはインスタンス（`this`）を初期化する場所であり、通常の関数とは役割が異なるためです。

```
class MyClass {
    constructor() {}
}
// クラスは関数として呼び出すことはできない
MyClass(); // => TypeError: class constructors must be invoked with |new| 
```

また、コンストラクタ関数は`return`文で任意のオブジェクトを返すことが可能ですが、行うべきではありません。 なぜなら、クラスを`new`演算子で呼び出し、その評価結果はクラスのインスタンスを期待するのが一般的であるためです。

次のコードのようにコンストラクタで返した値が`new`演算子で呼び出した際の返り値となります。 このような書き方は混乱を生むため避けるべきです。

```
// 非推奨の例: コンストラクタで値を返すべきではない
class Point {
    constructor(x, y) {
        // `this`の代わりにただのオブジェクトを返せる
        return { x, y };
    }
}

// `new`演算子の結果はコンストラクタ関数が返したただのオブジェクト
const point = new Point(3, 4);
console.log(point); // => { x: 3, y: 4 }
// Pointクラスのインスタンスではない
console.log(point instanceof Point); // => false 
```

### [](#class-name-start-upper-case)*[Note] クラス名は大文字ではじめる*

*JavaScriptでは慣習としてクラス名には大文字ではじまる名前をつけます。 これは、変数名にキャメルケースを使う慣習があるのと同じで、名前自体に特別なルールがあるわけではありません。 クラス名を大文字にしておき、そのインスタンスは小文字で開始すれば名前が被らないという合理的な理由で好まれています。

```
class Thing {}
const thing = new Thing(); 
```

### [](#class-vs-function)*[コラム] `class`構文と関数でのクラスの違い*

*ES2015より前はこれらのクラスを`class`構文ではなく、関数で表現していました。 その表現方法は人によってさまざまで、これも`class`構文という統一した記法が導入された理由の1つです。

次のコードは、関数でクラスを実装した1つの例です。 この関数でのクラス表現は、継承の仕組みなどは省かれていますが、`class`構文とよく似ています。

```
// コンストラクタ関数
const Point = function PointConstructor(x, y) {
    // インスタンスの初期化処理
    this.x = x;
    this.y = y;
};

// `new`演算子でコンストラクタ関数から新しいインスタンスを作成
const point = new Point(3, 4); 
```

大きな違いとして、`class`構文で定義したクラスは関数として呼び出すことができません。 クラスは`new`演算子でインスタンス化して使うものなので、これはクラスの誤用を防ぐ仕様です。 一方、関数でのクラス表現はただの関数なので、当然関数として呼び出せます。

```
// 関数でのクラス表現
function MyClassLike() {
}
// 関数なので関数として呼び出せる
MyClassLike();

// `class`構文でのクラス
class MyClass {
}
// クラスは関数として呼び出すと例外が発生する
MyClass(); // => TypeError: class constructors must be invoked with |new| 
```

このように、関数でクラスのようなものを実装した場合には、関数として呼び出せてしまう問題があります。 このような問題を避けるためにもクラスは`class`構文を使って実装します。

## [](#class-prototype-method-definition)*クラスのプロトタイプメソッドの定義*

*クラスの**動作**はメソッドによって定義できます。 `constructor`メソッドは初期化時に呼ばれる特殊なメソッドですが、`class`構文ではクラスに対して自由にメソッドを定義できます。 このクラスに定義したメソッドは作成したインスタンスが持つ動作となります。

次のように`class`構文ではクラスに対してメソッドを定義できます。 メソッドの中からクラスのインスタンスを参照するには、`constructor`メソッドと同じく`this`を使います。 このクラスのメソッドにおける`this`は「関数とthis」の章で学んだメソッドと同じくベースオブジェクトを参照します。

```
class クラス {
    メソッド() {
        // ここでの`this`はベースオブジェクトを参照
    }
}

const インスタンス = new クラス();
// メソッド呼び出しのベースオブジェクト(`this`)は`インスタンス`となる
インスタンス.メソッド(); 
```

クラスのプロトタイプメソッド定義では、オブジェクトにおけるメソッドとは異なり`key : value`のように`:`区切りでメソッドを定義できないことに注意してください。 つまり、次のような書き方は構文エラー（`SyntaxError`）となります。

```
// クラスでは次のようにメソッドを定義できない
class クラス {
   // SyntaxError
   メソッド: () => {}
   // SyntaxError
   メソッド: function(){}
} 
```

このメソッド定義の構文でクラスに対して定義したメソッドは、クラスの各インスタンスから**共有されるメソッド**となります。 このインスタンス間で共有されるメソッドのことを**プロトタイプメソッド**と呼びます。

次のコードでは、`Counter`クラスに`increment`メソッドを定義しています。 このときの`Counter`クラスのインスタンスは、それぞれ別々の状態（`count`プロパティ）を持ちます。

```
class Counter {
    constructor() {
        this.count = 0;
    }
    // `increment`メソッドをクラスに定義する
    increment() {
        // `this`は`Counter`のインスタンスを参照する
        this.count++;
    }
}
const counterA = new Counter();
const counterB = new Counter();
// `counterA.increment()`のベースオブジェクトは`counterA`インスタンス
counterA.increment();
// 各インスタンスの持つプロパティ(状態)は異なる
console.log(counterA.count); // => 1
console.log(counterB.count); // => 0 
```

このときの`increment`メソッドはプロトタイプメソッドとして定義されています。 プロトタイプメソッドは各インスタンス間(`counterA`と`counterB`)で共有されます。 そのため、次のように各インスタンスの`increment`メソッドの参照先は同じとなっていることがわかります。

```
class Counter {
    constructor() {
        this.count = 0;
    }
    increment() {
        this.count++;
    }
}
const counterA = new Counter();
const counterB = new Counter();
// 各インスタンスオブジェクトのメソッドは共有されている(同じ関数を参照している)
console.log(counterA.increment === counterB.increment); // => true 
```

プロトタイプメソッドがなぜインスタンス間で共有されているのかは、クラスの継承の仕組みと密接に関係しています。 プロトタイプメソッドの仕組みについては後ほど解説します。

ここでは、次のような構文でクラスにメソッドを定義すると、各インスタンスで共有されるプロトタイプメソッドとして定義されるということが理解できていれば問題ありません。

```
class クラス {
    メソッド() {
        // このメソッドはプロトタイプメソッドとして定義される
    }
} 
```

## [](#class-accessor-property)*クラスのアクセッサプロパティの定義*

*クラスに対してメソッドを定義できますが、メソッドは`インスタンス名.メソッド名()`のように呼び出す必要があります。 クラスでは、プロパティの参照（getter）、プロパティへの代入（setter）時に呼び出される特��なメソッドを定義できます。 このメソッドはプロパティのように振る舞うため**アクセッサプロパティ**と呼ばれます。

次のコードでは、プロパティの参照（getter）、プロパティへの代入（setter）に対するアクセッサプロパティを定義しています。 アクセッサプロパティはメソッド名（プロパティ名）の前に`get`または`set`をつけるだけです。 getter（`get`）には仮引数はありませんが、必ず値を返す必要があります。 setter（`set`）の仮引数にはプロパティへ代入する値が入りますが、値を返す必要はありません。

```
class クラス {
    // getter
    get プロパティ名() {
        return 値;
    }
    // setter
    set プロパティ名(仮引数) {
        // setterの処理
    }
}
const インスタンス = new クラス();
インスタンス.プロパティ名; // getterが呼び出される
インスタンス.プロパティ名 = 値; // setterが呼び出される 
```

次のコードでは、`NumberWrapper`クラスの`value`プロパティをアクセッサプロパティとして定義しています。 `value`プロパティへアクセスした際にそれぞれ定義したgetterとsetterが呼ばれているのがわかります。 このアクセッサプロパティで実際に読み書きされているのは、`NumberWrapper`インスタンスの`_value`プロパティとなります。

```
class NumberWrapper {
    constructor(value) {
        this._value = value;
    }
    // `_value`プロパティの値を返すgetter
    get value() {
        console.log("getter");
        return this._value;
    }
    // `_value`プロパティに値を代入するsetter
    set value(newValue) {
        console.log("setter");
        this._value = newValue;
    }
}

const numberWrapper = new NumberWrapper(1);
// "getter"とコンソールに表示される
console.log(numberWrapper.value); // => 1
// "setter"とコンソールに表示される
numberWrapper.value = 42;
// "getter"とコンソールに表示される
console.log(numberWrapper.value); // => 42 
```

### [](#underbar-private-property)*[コラム] `_`（アンダーバー）から始まるプロパティ名*

*NumberWrapperの`value`のアクセッサプロパティで実際に読み書きしているのは、`_value`プロパティです。 このように、外から直接読み書きしてほしくないプロパティを`_`（アンダーバー）から始まる名前にするのはただの習慣であるため、構文としての意味はありません。

ECMAScript 2022から、外から直接読み書きしてほしくないプライベートなプロパティを定義するPrivateクラスフィールド構文が追加されました。 Privateクラスフィールド構文では`#`（ハッシュ）記号をプロパティ名の前につけます。 そのため、外から直接読み書きしてほしくないプロパティを`_`からはじめるという慣習は、Privateクラスフィールド構文の利用が進むにつれて使われなくなっていくと考えています。

Privateクラスフィールド構文については、この後に解説します。

### [](#array-like-length)*`Array.prototype.length`をアクセッサプロパティで再現する*

*getterやsetterを利用しないと実現が難しいものとして、`Array.prototype.length`プロパティがあります。 Arrayの`length`プロパティへ値を代入すると、そのインデックス以降の要素は自動的に削除される仕様になっています。

次のコードでは、配列の要素数（`length`プロパティ）を小さくすると配列の要素が削除されています。

```
const array = [1, 2, 3, 4, 5];
// 要素数を減らすと、インデックス以降の要素が削除される
array.length = 2;
console.log(array.join(", ")); // => "1, 2"
// 要素数だけを増やしても、配列の中身は空要素が増えるだけ
array.length = 5;
console.log(array.join(", ")); // => "1, 2, , , " 
```

この`length`プロパティの挙動を再現する`ArrayLike`クラスを実装してみます。 Arrayの`length`プロパティは、`length`プロパティへ値を代入した際に次のようなことを行っています。

+   現在要素数より小さな**要素数**が指定された場合、その**要素数**を変更し、配列の末尾の要素を削除する

+   現在要素数より大きな**要素数**が指定された場合、その**要素数**だけを変更し、配列の実際の要素はそのままにする

ArrayLikeの`length`プロパティのsetterで要素の追加や削除を実装することで、配列のような`length`プロパティを実装できます。

```
/**
 * 配列のようなlengthを持つクラス
 */
class ArrayLike {
    constructor(items = []) {
        this._items = items;
    }

    get items() {
        return this._items;
    }

    get length() {
        return this._items.length;
    }

    set length(newLength) {
        const currentItemLength = this.items.length;
        // 現在要素数より小さな`newLength`が指定された場合、指定した要素数となるように末尾を削除する
        if (newLength < currentItemLength) {
            this._items = this.items.slice(0, newLength);
        } else if (newLength > currentItemLength) {
            // 現在要素数より大きな`newLength`が指定された場合、指定した要素数となるように末尾に空要素を追加する
            this._items = this.items.concat(new Array(newLength - currentItemLength));
        }
    }
}

const arrayLike = new ArrayLike([1, 2, 3, 4, 5]);
// 要素数を減らすとインデックス以降の要素が削除される
arrayLike.length = 2;
console.log(arrayLike.items.join(", ")); // => "1, 2"
// 要素数を増やすと末尾に空要素が追加される
arrayLike.length = 5;
console.log(arrayLike.items.join(", ")); // => "1, 2, , , " 
```

このようにアクセッサプロパティでは、プロパティのようでありながら実際にアクセスした際には他のプロパティと連動する動作を実現できます。

## [](#public-class-fields)*[ES2022] Publicクラスフィールド*

*クラスでは、`constructor`メソッドの中でクラスの状態であるインスタンスのプロパティを初期化することを紹介しました。 先ほども紹介した`Counter`クラスでは、`constructor`メソッドの中で`count`プロパティの初期値を`0`として定義しています。

```
class Counter {
    constructor() {
        this.count = 0;
    }
    increment() {
        this.count++;
    }
} 
```

この`Counter`では`new`演算子で何も引数を渡すことなく初期化するため、`constructor`メソッドには仮引数を定義していません。 このような場合でも、`constructor`メソッドを書かないとプロパティの初期化ができないためわずらわしいという問題がありました。

ES2022で，クラスのインスタンスが持つプロパティの初期化をわかりやすく宣言的にする構文として，**クラスフィールド**構文が追加されました。

クラスフィールドは，クラスのインスタンスが持つプロパティを定義する次のような構文です。

```
class クラス {
    プロパティ名 = プロパティの初期値;
} 
```

クラスフィールドを使って先ほどの`Counter`クラスを書き直してみると次のようになります。 `count`プロパティをクラスフィールドとして定義して，その初期値は`0`としています。

```
class Counter {
    count = 0;
    increment() {
        this.count++;
    }
}
const counter = new Counter();
counter.increment();
console.log(counter.count); // => 1 
```

クラスフィールドで定義するのは，クラスのインスタンスが持つプロパティです。 そのため，`constructor`メソッドの中で`this.count = 0`のように定義した場合と結果的にはほとんど同じ意味となります。 クラスフィールドで定義したプロパティは，クラス内から他のプロパティと同じように`this.プロパティ名`で参照できます。

クラスフィールドは`constructor`メソッドでの初期化と併用が可能です。 次のコードでは，クラスフィールドと`constructor`メソッドでそれぞれインスタンスのプロパティを定義しています。

```
// 別々のプロパティ名がそれぞれ定義される
class MyClass {
    publicField = 1;
    constructor(arg) {
        this.property = arg;
    }
}
const myClass = new MyClass(2);
console.log(myClass.publicField); // => 1
console.log(myClass.property); // => 2 
```

また，クラスフィールドでの初期化処理が行われ，そのあと`constructor`でのプロパティの定義という処理順となります。 そのため，同じプロパティ名への定義がある場合は，`constructor`メソッド内での定義でプロパティは上書きされます。

```
// 同じプロパティ名の場合は、constructorでの代入が後となる
class OwnClass {
    publicField = 1;
    constructor(arg) {
        this.publicField = arg;
    }
}
const ownClass = new OwnClass(2);
console.log(ownClass.publicField); // => 2 
```

この`publicField`プロパティのように，クラスの外からアクセスできるプロパティを定義するクラスフィールドを**Publicクラスフィールド**と呼びます。

### [](#declare-class-fields)*クラスフィールドを使ってプロパティの存在を宣言する*

*クラスフィ��ルドでは，プロパティの初期値は省略可能となっています。 そのため，次のように初期値を省略したPublicクラスフィールドも定義できます。

```
class MyClass {
    // myPropertyはundefinedで初期化される
    myProperty;
} 
```

このときの`myProperty`は`undefined`で初期化されます。 この初期値を省略したクラスフィールドの定義は，クラスのインスタンスが持つプロパティを明示するために利用できます。

次の`Loader`クラスは，`load`メソッドを呼び出すまでは，`loadedContent`プロパティの値は`undefined`です。 クラスフィールドを使えば，`Loader`クラスのインスタンスは，`loadedContent`というプロパティを持っていることを宣言的に表現できます。

```
class Loader {
    loadedContent;
    load() {
        this.loadedContent = "読み込んだコンテンツ内容";
    }
} 
```

JavaScriptでは，オブジェクトのプロパティは初期化時に存在していなくても，後から代入すれば作成できてしまいます。 そのため，次のように`Loader`クラスを実装しても意味は同じです。

```
class Loader {
    load() {
        this.loadedContent = "読み込んだコンテンツ内容";
    }
} 
```

しかし，このように実装してしまうと`Loader`クラスを利用する側は，`loadedContent`プロパティの存在を`load`メソッドの中まで読まないとわからないという問題があります。 これに対して，クラスフィールドを使って「`Loader`クラスは`loadedContent`というプロパティを持っている」ということを宣言的に表現できます。 宣言的にプロパティを定義することで，エディターでのコード補完が可能になったり，コードを読む人に優しいというメリットがあります。

### [](#this-in-class-fields)*クラスフィールドでの`this`はクラスのインスタンスを示す*

*クラスフィールドの初期値には任意の式が書け，`this`も利用できます。 クラスフィールドでの`this`は，そのクラスのインスタンスを参照します。

次のコードでは，`up`フィールドの初期値に`increment`メソッドを指定しています。 JavaScriptでは関数も値として扱えるため，`up`メソッドを呼び出すと`increment`メソッドが呼び出されます。

```
class Counter {
    count = 0;
    // upはincrementメソッドを参照している
    up = this.increment;
    increment() {
        this.count++;
    }
}
const counter = new Counter();
counter.up(); // 結果的にはincrementメソッドが呼び出される
console.log(counter.count); // => 1 
```

クラスフィールドでの`this`は，Arrow Functionと組み合わせると強力です。

次のコードでは，`up`メソッドをArrow Functionとして定義し，関数内では`this.increment`メソッドを呼び出しています。 Arrow Functionで定義した関数における`this`は，どのような呼び出し方をしても変化しません（「Arrow Functionでコールバック関数を扱う」を参照）。 そのため，`up`メソッドはどのような呼び方をした場合でも`this`がクラスのインスタンスとなるため，確実に`increment`メソッドを呼び出せます。

```
class Counter {
    count = 0;
    // クラスフィールドでの`this`はクラスのインスタンスとなる
    // upメソッドは、クラスのインスタンスに定義される
    up = () => {
        this.increment();
    };
    increment() {
        this.count++;
    }
}
const counter = new Counter();
// Arrow Functionなので、thisはクラスのインスタンスに固定されている
const up = counter.up;
up();
console.log(counter.count); // => 1
// 通常のメソッド定義では、`this`が`undefined`となってしまうため例外が発生する
const increment = counter.increment;
increment(); // Error: Uncaught TypeError: this is undefined 
```

### [](#difference-between-class-fields-and-instance-property)*[コラム] クラスフィールドとインスタンスのプロパティの違い*

*クラスフィールドで定義したプロパティやメソッドは，クラスのインスタンスにプロパティとして定義されます。 そのため，クラスフィールドは，`constructor`の中で`this`に対してプロパティを追加するのと意味的にはほぼ同じで，見た目がわかりやすくな���た構文と捉えることができます。

```
class ExampleClass {
    fieldMethod = () => {
        console.log("クラスフィールドで定義されたメソッド");
    };
    constructor() {
        this.propertyMethod = () => {
            console.log("インスタンスにプロパティとして定義されたメソッド");
        };
    }
} 
```

しかし，厳密にはこのふたつのプロパティ定義には異なる点はあります。 次のように，クラスフィールドと`constructor`の中で`this`に追加するプロパティ名に対するsetterを定義してみるとこの違いがわかります。

```
class ExampleClass {
    field = "フィールド";
    constructor() {
        this.property = "コンストラクタ";
    }
    // クラスフィールド名に対応するsetter
    set field(value) {
        console.log("fieldで定義された値", value);
    }
    // thisのプロパティ名に対応するsetter
    set property(value) {
        console.log("consctrutorで代入された値", value);
    }
}
// set fieldは呼び出されない
// 一方で、set propertyは呼び出される
const example = new ExampleClass(); 
```

クラスフィールド名に対するsetterは呼び出されないのに対して，`this.property`への代入に対するsetterは呼び出されています。 これは，クラスフィールドは`=`を使った代入で定義されるのではなく，[Object.defineProperty](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)メソッドを使ってプロパティが定義されるという違いがあります。 `Object.defineProperty`を使ったプロパティの定義では，setterは無視してプロパティが定義されます。 setterは`=`での代入に反応します。そのため，`constructor`の中での`this.property`への代入に対してはsetterが呼び出されます。

同じプロパティの定義であっても，プロパティの定義の仕組みが微妙に異なる点から，このような挙動の違いが存在しています。 しかし，この違いを意識するようなコードを書くことは避けたほうが安全です。 実際に見た目からこの違いを意識するのは難しく，それを意識させるようなコードは複雑性が高いためです。

## [](#private-class-fields)*[ES2022] Privateクラスフィールド*

*クラスフィールド構文で次のように書くと、定義したプロパティはクラスをインスタンス化した後に外からも参照できます。 そのため、Publicクラスフィールドと呼ばれます。

```
class クラス {
    プロパティ名 = プロパティの初期値;
} 
```

一方で外からアクセスされたくないインスタンスのプロパティも存在します。 そのようなプライベートなプロパティを定義する構文もES2022で追加されています。

Privateクラスフィールドは、次のように`#`をフィールド名の前につけたクラスフィールドを定義します。

```
class クラス {
    // プライベートなプロパティは#をつける
    #フィールド名 = プロパティの初期値;
} 
```

定義したPrivateクラスフィールドは、`this.#フィールド名`で参照できます。

```
class PrivateExampleClass {
    #privateField = 42;
    dump() {
        // Privateクラスフィールドはクラス内からのみ参照できる
        console.log(this.#privateField); // => 42
    }
}
const privateExample = new PrivateExampleClass();
privateExample.dump(); 
```

もう少し具体的なPrivateクラスフィールドの使い方を見ていきます。 アクセッサプロパティの例でも登場した`NumberWrapper`をPrivateクラスフィールドを使って書き直してみます。 元々の`NumberWrapper`クラスでは、`_value`プロパティに実際の値を読み書きしていました。 この場合、`_value`プロパティは、外からもアクセスできてしまうため、定義したgetterとsetterが無視できてしまいます。

```
class NumberWrapper {
    // Publicクラスフィールドなのでクラスの外からアクセスができる
    _value;
    constructor(value) {
        this._value = value;
    }
    // `_value`プロパティの値を返すgetter
    get value() {
        return this._value;
    }
    // `_value`プロパティに値を代入するsetter
    set value(newValue) {
        this._value = newValue;
    }
}
const numberWrapper = new NumberWrapper(1);
// _valueプロパティは外からもアクセスできる
console.log(numberWrapper._value); // => 1 
```

Privateクラスフィールドでは、外からアクセスされたくないプロパティを`#`をつけてクラスフィールドとして定義します。 次のコードでは、`#value`はプライベートプロパティとなっているため、構文エラーが発生し外からアクセスできなくなることが確認できます。 Privateクラスフィールドを使うことで、クラスを利用する際はgetterとsetterを経由しないと`#value`を参照できなくなりました。

```
class NumberWrapper {
    // valueはPrivateクラスフィールドとして定義
    #value;
    constructor(value) {
        this.#value = value;
    }
    // `#value`フィールドの値を返すgetter
    get value() {
        return this.#value;
    }
    // `#value`フィールドに値を代入するsetter
    set value(newValue) {
        this.#value = newValue;
    }
}

const numberWrapper = new NumberWrapper(1);
// クラスの外からPrivateクラスフィールドには直接はアクセスできない
console.log(numberWrapper.#value); // => SyntaxError: reference to undeclared private field or method #value 
```

Privateクラスフィールドを使うことで、クラスの外からアクセスさせたくないプロパティを宣言できます。 これは、実装したクラスの意図しない使われ方を防いだり、クラスの外からプロパティの状態を直接書き換えるといった行為を防げます。

また、Privateクラスフィールドでは、途中から値が入る場合でもフィールドの宣言が必須となっています。 次のコードでは、`#loadedContent`に実際に値が入るのは`load`メソッドが呼び出されたときです。 Publicクラスフィールドではフィールドの定義は省略可能でしたが、Privateクラスフィールドでは`#loadedContent`フィールドの定義が必須となっています。 言い換えると、Privateクラスフィールドでは、クラスを定義した段階でクラスに存在するすべてのPrivateクラスフィールドを明示する必要があります。

```
class PrivateLoader {
    // 途中で値が入る場合でも最初に`undefined`で初期化されるフィールドの定義が必須
    #loadedContent;
    load() {
        this.#loadedContent = "読み込んだコンテンツ内容";
    }
} 
```

## [](#static-method)*静的メソッド*

*インスタンスメソッドは、クラスをインスタンス化して利用します。 一方、クラスをインスタンス化せずに利用できる静的メソッド（クラスメソッド）もあります。

静的メソッドの定義方法はメソッド名の前に、`static`をつけるだけです。

```
class クラス {
    static メソッド() {
        // 静的メソッドの処理
    }
}
// 静的メソッドの呼び出し
クラス.メソッド(); 
```

次のコードでは、配列をラップする`ArrayWrapper`というクラスを定義しています。 `ArrayWrapper`はコンストラクタの引数として配列を受け取って初期化しています。 このクラスに配列ではなく要素そのものを引数に受け取ってインスタンス化できる`ArrayWrapper.of`という静的メソッドを定義します。

```
class ArrayWrapper {
    // new 演算子で引数が渡されたなかった場合の初期値は空配列
    constructor(array = []) {
        this.array = array;
    }

    // rest parametersとして要素を受けつける
    static of(...items) {
        return new ArrayWrapper(items);
    }

    get length() {
        return this.array.length;
    }
}

// 配列を引数として渡している
const arrayWrapperA = new ArrayWrapper([1, 2, 3]);
// 要素を引数として渡している
const arrayWrapperB = ArrayWrapper.of(1, 2, 3);
console.log(arrayWrapperA.length); // => 3
console.log(arrayWrapperB.length); // => 3 
```

クラスの静的メソッドにおける`this`は、そのクラス自身を参照します。 そのため、先ほどのコードは`new ArrayWrapper`の代わりに`new this`と書くこともできます。

```
class ArrayWrapper {
    constructor(array = []) {
        this.array = array;
    }

    static of(...items) {
        // `this`は`ArrayWrapper`を参照する
        return new this(items);
    }

    get length() {
        return this.array.length;
    }
}

const arrayWrapper = ArrayWrapper.of(1, 2, 3);
console.log(arrayWrapper.length); // => 3 
```

このように静的メソッドでの`this`はクラス自身を参照するため、クラスのインスタンスは参照できません。 そのため静的メソッドは、クラスのインスタンスを作成する処理やクラスに関係する処理を書くために利用されます。

### [](#static-class-fields)*[ES2022] 静的クラスフィールド*

*ES2022で追加されたクラスフィールドでは、インスタンスではなくクラス自体に定義する静的クラスフィールドも利用できます。

静的クラスフィールドは、フィールドの前に`static`をつけるだけです。 静的クラスフィールドで定義したプロパティは、クラス自体のプロパティとして定義されます。 次のコードでは、Public 静的クラスフィールドを使って`Colors`クラス自体にプロパティを定義しています。

```
class Colors {
    static GREEN = "緑";
    static RED = "赤";
    static BLUE = "青";
}
// クラスのプロパティとして参照できる
console.log(Colors.GREEN); // => "緑" 
```

また、Privateクラスフィールドも静的に利用できます。 Private 静的クラスフィールドは、クラス自体にプロパティを定義したいが、そのプロパティを外から参照されたくない場合に利用します。 Private 静的クラスフィールドはフィールドの前に、`static`をつけるだけです。

```
class MyClass {
    static #privateClassProp = "This is private";
    static outputPrivate() {
        // クラス内からはPrivate 静的クラスフィールドで定義したプロパティを参照できる
        console.log(this.#privateClassProp);
    }
}
MyClass.outputPrivate(); 
```

## [](#two-instance-method-definition)*プロトタ��プに定義したメソッドとインスタンスに定義したメソッドの違い*

*ここまでで、プロトタイプメソッドの定義とクラスフィールドを使ったインスタンスに対するメソッドの定義の2 種類を見てきました。 プロトタイプメソッドの定義方法は、メソッドをプロトタイプオブジェクトという特殊なオブジェクトに定義します。 一方で、クラスフィールドで定義したメソッドは、クラスのインスタンスに対してメソッドを定義します。

どちらのメソッド定義方法でも、`new`演算子でインスタンス化したオブジェクトからメソッドを呼び出すことができる点は同じです。

```
class ExampleClass {
    // クラスフィールドを使い、インスタンスにメソッドを定義
    instanceMethod = () => {
        console.log("インスタンスメソッド");
    };
    // メソッド構文を使い、プロトタイプオブジェクトにメソッドを定義
    prototypeMethod() {
        console.log("プロトタイプメソッド");
    }
}
const example = new ExampleClass();
// どちらのメソッドもインスタンスから呼び出せる
example.instanceMethod();
example.prototypeMethod(); 
```

しかしこの2つのメソッドの定義方法は、メソッドの定義先となるオブジェクトが異なります。

まず、この2 種類のメソッドがそれぞれ別の場所へと定義されていることを見ていきます。 次のコードでは、`ConflictClass`クラスに`method`という同じ名前のメソッドをプロトタイプメソッドとインスタンスに対してそれぞれ定義しています。

```
class ConflictClass {
    // インスタンスオブジェクトに`method`を定義
    method = () => {
        console.log("インスタンスオブジェクトのメソッド");
    };

    // クラスのプロトタイプメソッドとして`method`を定義
    method() {
        console.log("プロトタイプのメソッド");
    }
}

const conflict = new ConflictClass();
conflict.method(); // どちらの`method`が呼び出される？ 
```

結論から述べると、この場合はインスタンスオブジェクトに定義した`method`が呼び出されます。 このとき、インスタンスの`method`プロパティを`delete`演算子で削除すると、今度はプロトタイプメソッドの`method`が呼び出されます。

```
class ConflictClass {
    // インスタンスオブジェクトに`method`を定義
    method = () => {
        console.log("インスタンスオブジェクトのメソッド");
    };

    method() {
        console.log("プロトタイプメソッド");
    }
}

const conflict = new ConflictClass();
conflict.method(); // "インスタンスオブジェクトのメソッド"
// インスタンスの`method`プロパティを削除
delete conflict.method;
conflict.method(); // "プロトタイプメソッド" 
```

この実行結果から次のことがわかります。

+   プロトタイプメソッドとインスタンスオブジェクトのメソッドは上書きされずにどちらも定義されている

+   インスタンスオブジェクトの��ソッドがプロトタイプオブジェクトのメソッドよりも優先して呼ばれている

どちらも注意深く意識しないと気づきにくいですが、この挙動はJavaScriptの重要な仕組みであるため理解することは重要です。

この挙動は**プロトタイプオブジェクト**と呼ばれる特殊なオブジェクトと**プロトタイプチェーン**と呼ばれる仕組みで成り立っています。 どちらも**プロトタイプ**とついていることからわかるように、2つで1 組のような仕組みです。

次のセクションでは、**プロトタイプオブジェクト**と**プロトタイプチェーン**とはどのような仕組みなのかを見ていきます。

## [](#prototype)*プロトタイプオブジェクト*

***プロトタイプメソッド**と**インスタンスオブジェクトのメソッド**を同時に定義しても、互いのメソッドは上書きされるわけでありません。 なぜなら、プロトタイプメソッドは**プロトタイプオブジェクト**へ、インスタンスオブジェクトのメソッドは**インスタンスオブジェクト**へそれぞれ定義されるためです。

プロトタイプオブジェクトについては「プロトタイプオブジェクト」の章で簡単に紹介していましたが、改めて解説していきます。

**プロトタイプオブジェクト**とは、JavaScriptの関数オブジェクトの`prototype`プロパティに自動的に作成される特殊なオブジェクトです。 クラスも一種の関数オブジェクトであるため、自動的に`prototype`プロパティにプロトタイプオブジェクトが作成されています。

次のコードでは、関数やクラス自身の`prototype`プロパティに、プロトタイプオブジェクトが自動的に作成されていることがわかります。

```
function fn() {
}
// `prototype`プロパティにプロトタイプオブジェクトが存在する
console.log(typeof fn.prototype === "object"); // => true

class MyClass {
}
// `prototype`プロパティにプロトタイプオブジェクトが存在する
console.log(typeof MyClass.prototype === "object"); // => true 
```

`class`構文のメソッド定義は、このプロトタイプオブジェクトのプロパティとして定義されます。

次のコードでは、クラスのメソッドがプロトタイプオブジェクトに定義されていることを確認できます。 また、クラスには`constructor`メソッド（コンストラクタ）が必ず定義されます。 この`constructor`メソッドもプロトタイプオブジェクトに定義されており、この`constructor`プロパティはクラス自身を参照します。

```
class MyClass {
    method() {}
}

console.log(typeof MyClass.prototype.method === "function"); // => true
// クラスのconstructorはクラス自身を参照する
console.log(MyClass.prototype.constructor === MyClass); // => true 
```

このように、プロトタイプメソッドはプロトタイプオブジェクトに定義され、インスタンスオブジェクトのメソッドとは異なるオブジェクトに定義されています。そのため、それぞれの方法でメソッドを定義しても、上書きされることはありません。

## [](#prototype-chain)*プロトタイプチェーン*

*`class`構文で定義したプロトタイプメソッドはプロトタイプオブジェクトに定義されます。 しかし、インスタンス（オブジェクト）にはメソッドが定義されていないのに、インスタンスからクラスのプロトタイプメソッドを呼び出せます。

```
class MyClass {
    method() {
        console.log("プロトタイプのメソッド");
    }
}
const instance = new MyClass();
instance.method(); // "プロトタイプのメソッド" 
```

インスタンスからプロトタイプメソッドを呼び出せるのは**プロトタイプチェーン**と呼ばれる仕組みによるものです。 プロトタイプチェーンは2つの処理から成り立ちます。

+   インスタンス作成時に、インスタンスの`[[Prototype]]`内部プロパティへプロトタイプオブジェクトの参照を保存する処理

+   インスタンスからプロパティ（またはメソッド）を参照するときに、`[[Prototype]]`内部プロパティまで探索する処理

### [](#write-prototype-chain)*インスタンス作成とプロトタイプチェーン*

*クラスから`new`演算子によってインスタンスを作成する際に、インスタンスにはクラスのプロトタイプオブジェクトへの参照が保存されます。 こ���とき、インスタンスからクラスのプロトタイプオブジェクトへの参照は、インスタンスオブジェクトの`[[Prototype]]`という内部プロパティに保存されます。

`[[Prototype]]`内部プロパティはECMAScriptの仕様で定められた内部的な表現であるため、通常のプロパティのようにはアクセスできません。 ここでは説明のために、`[[プロパティ名]]`という書式でECMAScriptの仕様上に存在する内部プロパティを表現しています。

`[[Prototype]]`内部プロパティへプロパティのようにはアクセスできませんが、`Object.getPrototypeOf`メソッドで`[[Prototype]]`内部プロパティを参照できます。

次のコードでは、`instance`オブジェクトの`[[Prototype]]`内部プロパティを取得しています。 その取得した結果がクラスのプロトタイプオブジェクトを参照していることを確認できます。

```
class MyClass {
    method() {
        console.log("プロトタイプのメソッド");
    }
}
const instance = new MyClass();
// `instance`の`[[Prototype]]`内部プロパティは`MyClass.prototype`と一致する
const MyClassPrototype = Object.getPrototypeOf(instance);
console.log(MyClassPrototype === MyClass.prototype); // => true 
```

ここで重要なのは、インスタンスはどのクラスから作られたかやそのクラスのプロトタイプオブジェクトを知っているということです。

#### [](#inner-property)*[Note] `[[Prototype]]`内部プロパティを読み書きする*

*`Object.getPrototypeOf(オブジェクト)`で`オブジェクト`の`[[Prototype]]`を読み取ることができます。 一方、`Object.setPrototypeOf(オブジェクト, プロトタイプオブジェクト)`で`オブジェクト`の`[[Prototype]]`に`プロトタイプオブジェクト`を設定できます。 また、`[[Prototype]]`内部プロパティを通常のプロパティのように扱え���`__proto__`という特殊なアクセッサプロパティが存在します。

しかし、これらの`[[Prototype]]`内部プロパティを直接読み書きすることは通常の用途では行いません。 また、既存のビルトインオブジェクトの動作なども変更できるため、不用意に扱うべきではないでしょう。

### [](#read-prototype-chain)*プロパティの参照とプロトタイプチェーン*

*プロトタイプオブジェクトのプロパティがどのようにインスタンスから参照されるかを見ていきます。

オブジェクトのプロパティを参照するときに、オブジェクト自身がプロパティを持っていない場合でも、そこで探索が終わるわけではありません。 オブジェクトの`[[Prototype]]`内部プロパティ（仕様上の内部的なプロパティ）の参照先であるプロトタイプオブジェクトに対しても探索を続けます。 これは、スコープに指定した識別子の変数がなかった場合に外側のスコープへと探索するスコープチェーンと良く似た仕組みです。

つまり、オブジェクトがプロパティを探索するときは次のような順番で、それぞれのオブジェクトを調べます。 すべてのオブジェクトにおいて見つからなかった場合の結果は`undefined`を返します。

1.  `instance`オブジェクト自身

1.  `instance`オブジェクトの`[[Prototype]]`の参照先（プロトタイプオブジェクト）

1.  どこにもなかった場合は`undefined`

次のコードでは、インスタンスオブジェクト自身は`method`プロパティを持っていません。 そのため、実際に参照しているのはクラスのプロトタイプオブジェクトの`method`プロパティです。

```
class MyClass {
    method() {
        console.log("プロトタイプのメソッド");
    }
}
const instance = new MyClass();
// インスタンスには`method`プロパティがないため、プロトタイプオブジェクトの`method`が参照される
instance.method(); // "プロトタイプのメソッド"
// `instance.method`の参照はプロトタイプオブジェクトの`method`と一致する
const Prototype = Object.getPrototypeOf(instance);
console.log(instance.method === Prototype.method); // => true 
```

このように、インスタンスオブジェクトに`method`が定義されていなくても、クラスのプロトタイプオブジェクトの`method`を呼び出すことができます。 このプロパティを参照する際に、オブジェクト自身から`[[Prototype]]`内部プロパティへと順番に探す仕組みのことを**プロトタイプチェーン**と呼びます。

プロトタイプチェーンの仕組みを疑似的なコードとして表現すると次のような動きをしています。

```
// プロトタイプチェーンの動作の疑似的なコード
class MyClass {
    method() {
        console.log("プロトタイプのメソッド");
    }
}
const instance = new MyClass();
// `instance.method()`を実行する場合
// 次のような呼び出し処理が行われている
// インスタンスが`method`プロパティを持っている場合
if (Object.hasOwn(instance, "method")) {
    instance.method();
} else {
    // インスタンスの`[[Prototype]]`の参照先（`MyClass`のプロトタイプオブジェクト）を取り出す
    const prototypeObject = Object.getPrototypeOf(instance);
    // プロトタイプオブジェクトが`method`プロパティを持っている場合
    if (Object.hasOwn(prototypeObject, "method")) {
        // `this`はインスタンス自身を指定して呼び出す
        prototypeObject.method.call(instance);
    }
} 
```

通过原型链机制，我们可以从原型对象中定义的原型方法中调用实例。

普通情况下，无需关注原型对象或原型链等机制。 `class`语法是为了让我们在不关注这些原型的情况下使用类而引入的语法。 但是，在基于原型的 JavaScript 中，了解类是如何通过原型来表示是有好处的。

## [](#extends)*继承*

*通过使用`extends`关键字，您可以继承现有的类。 继承是指定义一个新类，该类继承了父类的**结构**和**功能**。

### [](#class-extends)*继承的类定义*

*通过使用`extends`关键字来定义继承自父类（基类）的**子类**（派生类）。 通过在`class`语法的右侧使用`extends`关键字指定要继承的**父类**（基类），可以定义继承自父类的**子类**（派生类）。

```
class 子クラス extends 親クラス {
} 
```

在下面的代码中，我们定义了继承自`Parent`类的`Child`类。 实例化子类`Child`类与普通类一样使用`new`运算符。

```
class Parent {
}
class Child extends Parent {
}
const instance = new Child(); 
```

### [](#class-super)*`super`*

*要从使用`extends`定义的子类引用父类，需要使用名为`super`的关键字。 我们将看到使用最简单的`super`的示例，以查看构造函数的处理。 

就像在`class`语法中介绍的那样，类必须具有`constructor`方法（构造函数）。 这也适用于继承的子类。

在下面的代码中，在`Child`类的构造函数中，我们调用了`super()`。 `super()`会调用父类的`constructor`方法。

```
// 親クラス
class Parent {
    constructor(...args) {
        console.log("Parentコンストラクタの処理", ...args);
    }
}
// Parentを継承したChildクラスの定義
class Child extends Parent {
    constructor(...args) {
        // Parentのコンストラクタ処理を呼び出す
        super(...args);
        console.log("Childコンストラクタの処理", ...args);
    }
}
const child = new Child("引数 1", "引数 2");
// "Parentコンストラクタの処理", "引数 1", "引数 2"
// "Childコンストラクタの処理", "引数 1", "引数 2" 
```

在`class`语法中，如果`constructor`方法（构造函数）不执行任何操作，则可以省略它。 这也适用于继承的子类。

在下面的代码中，`Child`类的构造函数没有执行任何操作。 因此，可以省略`Child`类的`constructor`方法的定义。

```
class Parent {}
class Child extends Parent {} 
```

如果在子类中省略了`constructor`，则与下面的代码相同。 接收`constructor`方法的所有参数，并将它们按顺序传递给`super`。

```
class Parent {}
class Child extends Parent {
    constructor(...args) {
        super(...args); // 親クラスに引数をそのまま渡す
    }
} 
```

### [](#constructor-order)*构造函数的处理顺序是从父类到子类*

*构造函数的处理顺序是从父类到子类。

在`class`语法中，必须首先执行父类的构造函数处理（调用`super()`），然后执行子类的构造函数处理。 在子类的构造函数中，在引用`this`之前必须先调用`super()`以调用父类的构造函数处理，否则会导致`ReferenceError`。

在下面的代码中，我们分别在`Parent`和`Child`中写入了实例（`this`）的`name`属性。 在子类中，必须先调用`super()`，然后才能引用`this`。 因此，构造函数的处理顺序被限制为从`Parent`到`Child`。

```
class Parent {
    constructor() {
        this.name = "Parent";
    }
}
class Child extends Parent {
    constructor() {
        // 子クラスでは`super()`を`this`に触る前に呼び出さなければならない
        super();
        // 子クラスのコンストラクタ処理
        // 親クラスで書き込まれた`name`は上書きされる
        this.name = "Child";
    }
}
const parent = new Parent();
console.log(parent.name); // => "Parent"
const child = new Child();
console.log(child.name); // => "Child" 
```

### [](#class-fields-inheritance)*类字段继承*

*Public 类字段与构造函数的处理顺序相同，即在初始化父类字段之后初始化子类字段。 Public 类字段是一种定义实例对象属性的语法。 因此，父类中定义的字段也会被定义为实际实例化的对象的属性。

```
class Parent {
    parentField = "親クラスで定義したフィールド";
}
// `Parent`を継承した`Child`を定義
class Child extends Parent {
    childField = "子クラスで定義したフィールド";
}
const instance = new Child();
console.log(instance.parentField); // => "親クラスで定義したフィールド"
console.log(instance.childField); // => "子クラスで定義したフィールド" 
```

如果存在相同名称的字段，则将在子类的字段定义中进行覆盖。

```
class Parent {
    field = "親クラスで定義したフィールド";
}
// `Parent`を継承した`Child`を定義
class Child extends Parent {
    field = "子クラスで定義したフィールド";
}
const instance = new Child();
console.log(instance.field); // => "子クラスで定義したフィールド" 
```

Public 类字段会将父类中定义的字段也定义在子类中。 另一方面，Private 类字段不会将父类中定义的字段定义在子类中。

在下面的代码中，我们试图从子类中引用在父类中定义的 Private 类字段。 但是，我们发现`#parentField`无法被引用，导致语法错误。

```
class Parent {
    #parentField = "親クラスで定義したPrivateフィールド";
}
// `Parent`を継承した`Child`を定義
class Child extends Parent {
    dump() {
        console.log(this.#parentField); // => SyntaxError: reference to undeclared private field or method #parentFeild
    }
}
const instance = new Child();
instance.dump(); 
```

这是因为 Private 类字段的 Private 旨在保护每个类的 Private，因此从继承的类中可以使用 Private 类字段，这会导致 Private 信息泄漏给子类。 在 JavaScript 中，没有一种语���可以定义既不想公开给类外部又想让子类使用的中间限制属性。

这样，严格拒绝从子类外部访问的 Private 称为 hard private。 JavaScript 中的 Private 类字段被视为 hard private。

另一方面，允许子类访问或允许类外部访问的 Private 称为 soft private。 在 JavaScript 中，soft private 需要用户自己使用 WeakMap 或 WeakSet 来实现（请参考“Map/Set”章节）。

### [](#prototype-inheritance)*原型继承*

*在下面的代码中，我们使用`extends`关键字定义了继承自`Parent`类的`Child`类。 由于`Parent`类定义了`method`，因此可以从继承该方法的`Child`类的实例中调用。

```
class Parent {
    method() {
        console.log("Parent.prototype.method");
    }
}
// `Parent`を継承した`Child`を定義
class Child extends Parent {
    // methodの定義はない
}
// `Child`のインスタンスは`Parent`のプロトタイプメソッドを継承している
const instance = new Child();
instance.method(); // "Parent.prototype.method" 
```

这样，通过原型链机制，子类的实例也可以调用父类的原型方法。

通过`extends`继承时，子类的原型对象的`[[Prototype]]`内部属性将设置为父类的原型对象。 在此代码中，`Child.prototype`对象的`[[Prototype]]`内部属性将设置为`Parent.prototype`。

这样，当引用属性时，会按照以下顺序搜索对象。

1.  `instance`对象本身

1.  `Child.prototype`（指向`instance`对象的`[[Prototype]]`）

1.  `Parent.prototype`（指向`Child.prototype`对象的`[[Prototype]]`）

通过这种原型链的机制，`method`属性引用了在`Parent.prototype`对象中定义的方法。

通过 JavaScript 的`class`语法和`extends`关键字，可以继承类的**功能**。`class`语法通过引用原型对象来实现继承。因此，这种继承机制被称为**原型继承**。

### [](#static-inheritance)*静态方法的继承*

*在实例与类的原型对象之间存在原型链。类本身（类的构造函数）也与其父类本身（父类的构造函数）之间存在原型链。

简而言之，静态方法也会被继承。

```
class Parent {
    static hello() {
        return "Hello";
    }
}
class Child extends Parent {}
console.log(Child.hello()); // => "Hello" 
```

当通过`extends`进行继承时，子类构造函数的`[[Prototype]]`内部属性会被设置为父类构造函数。在这段代码中，`Child`构造函数的`[[Prototype]]`内部属性会被设置为`Parent`构造函数。

换句话说，在先前的代码中，如果引用了`Child.hello`属性，将按照以下顺序搜索对象。

1.  `Child`构造函数

1.  `Parent`构造函数（指向`Child`构造函数的`[[Prototype]]`）

由于类的构造函数之间也存在原型链的机制，子类可以调用父类的静态方法。

### [](#super-property)*`super`属性*

*子类调用父类构造函数的处理需要使用`super()`。同样地，从子类的原型方法中，可以通过`super.属性名`来引用父类的原型方法。

在下面的代码中，通过在`Child.prototype.method`中编写`super.method()`，调用了`Parent.prototype.method`。因此，可以通过子类引用父类的原型方法。

```
class Parent {
    method() {
        console.log("Parent.prototype.method");
    }
}
class Child extends Parent {
    method() {
        console.log("Child.prototype.method");
        // `this.method()`だと自分(`this`)のmethodを呼び出して無限ループする
        // そのため明示的に`super.method()`を呼ぶことで、Parent.prototype.methodを呼び出す
        super.method();
    }
}
const child = new Child();
child.method();
// コンソールには次のように出力される
// "Child.prototype.method"
// "Parent.prototype.method" 
```

我们介绍了原型链的概念，该链从实例到类，然后到父类，以搜索方法。在这段代码中，由于`Child.prototype.method`被定义，因此`child.method`将调用`Child.prototype.method`。然后，由于`Child.prototype.method`调用了`super.method`，因此将调用`Parent.prototype.method`。

类的静态方法之间也可以通过`super.method()`来调用。在下面的代码中，从继承了`Parent`的`Child`中调用了父类的静态方法。

```
class Parent {
    static method() {
        console.log("Parent.method");
    }
}
class Child extends Parent {
    static method() {
        console.log("Child.method");
        // `super.method()`で`Parent.method`を呼びだす
        super.method();
    }
}
Child.method();
// コンソールには次のように出力される
// "Child.method"
// "Parent.method" 
```

### [](#instanceof)*继承的判定*

*通过使用`instanceof`运算符，可以确定一个类是否继承了指定的类。

下面的代码确认了`Child`的实例是一个继承了`Child`类和`Parent`类的对象。

```
class Parent {}
class Child extends Parent {}

const parent = new Parent();
const child = new Child();
// `Parent`のインスタンスは`Parent`のみを継承したインスタンス
console.log(parent instanceof Parent); // => true
console.log(parent instanceof Child); // => false
// `Child`のインスタンスは`Child`と`Parent`を継承したインスタンス
console.log(child instanceof Parent); // => true
console.log(child instanceof Child); // => true 
```

更具体的继承用法将在"使用案例:Todo 应用"章节中介绍。

## [](#extends-built-in)*内置对象的继承*

*到目前为止，我们已经学习了如何继承自己定义的类，但是内置对象的构造函数也可以继承。内置对象包括`Array`、`String`、`Object`、`Number`、`Error`、`Date`等。这些内置对象的构造函数可以通过`class`语法进行继承。

下面的代码定义了一个继承自内置对象`Array`并添加了自定义方法的`MyArray`类。继承的`MyArray`类继承了`Array`的方法和状态管理机制。除了继承的性质外，MyArray 类还添加了`first`和`last`等访问器属性。

```
class MyArray extends Array {
    get first() {
        return this.at(0);
    }

    get last() {
        return this.at(-1);
    }
}

// Arrayを継承しているのでArray.fromも継承している
// Array.fromはIterableなオブジェクトから配列インスタンスを作成する
const array = MyArray.from([1, 2, 3, 4, 5]);
console.log(array.length); // => 5
console.log(array.first); // => 1
console.log(array.last); // => 5 
```

继承自`Array`的`MyArray`可以使用`Array`原先具有的`length`属性和`Array.from`方法等。

## [](#conclusion)*总结*

*在本章中，我们学习了有关类的内容。

+   JavaScript 的类是基于原型的

+   类可以使用`class`语法来定义

+   在类中定义的方法可以通过原型对象和原型链的机制来调用

+   类实例的属性定义可以使用类字段

+   使用私有类字段可以定义不希望从外部访问的属性

+   通过定义 getter 和 setter 方法，访问器属性可以像属性一样使用

+   类可以通过`extends`来进行继承

+   类的原型方法和静态方法都会被继承********************************
