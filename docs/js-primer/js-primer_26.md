# 类

> 原文：[`jsprimer.net/basic/class/`](https://jsprimer.net/basic/class/)

“类”一词有多种含义，因此在这里，指代可以定义**结构**、**行为**和**状态**的东西。此外，在本章中，表示概念时称为**类**，而表示类相关结构（编写的代码）时称为`class`语法。

**类**是定义**行为**或**状态**的**结构**。从类中可以创建称为实例的对象，实例继承类中定义的**行为**，**状态**会随着行为而变化。这听起来很抽象，但这实际上是以前用对象或函数表达的东西。在 JavaScript 中，ES2015 之前没有`class`语法，而是用函数来表示类似类的东西并处理。

ES2015 引入了用于表示类的`class`语法，但用`class`语法定义的类实际上是函数对象的一种。`class`语法使用基于原型的继承机制用函数来表示类。因此，`class`语法可以说是对创建类和继承进行模式化的写法。

此外，像函数有函数声明和函数表达式定义方法一样，类也有类声明和类表达式。这样，函数和类有很多相似之处。

本章将学习`class`语法中的类定义、继承和类的性质。

## [](#class-declaration)*类的定义*

*定义类可以使用`class`语法。类的定义方法有类声明和类表达式两种。

首先来看使用类声明定义类的方法。

在类声明语句中，使用`class`关键字，可以像`class クラス名{ }`这样定义类的**结构**。

类必须有一个构造函数，并以`constructor`命名的方法来定义。构造函数是创建类实例时初始化实例相关**状态**的方法。`constructor`方法中定义的处理会在类实例化时自动调用。

```
class MyClass {
    constructor() {
        // コンストラクタ関数の処理
        // インスタンス化されるときに自動的に呼び出される
    }
} 
```

另一种定义方法，即类表达式，是定义类作为值的方法。在类表达式中可以省略类名。这与函数表达式中的匿名函数相同。

```
const MyClass = class MyClass {
    constructor() {}
};

const AnonymousClass = class {
    constructor() {}
}; 
```

在构造函数内没有处理的情况下，可以省略构造函数的描述。省略后，也会自动定义一个空的构造函数，因此类中总是存在构造函数。

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

## [](#class-instance)*类的实例化*

*类可以使用`new`运算符创建实例对象。用`class`语法定义的类创建实例的过程称为**实例化**。要判断一个实例是否是由指定的类创建的，可以使用`instanceof`运算符。

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

这样定义的类是空的，没有处理，所以定义一个具有值的类。

类中在构造函数中执行实例的初始化处理。构造函数在用`new`运算符实例化时自动调用。构造函数中的`this`是新创建的实例对象。

在下面的代码中，定义了一个名为`Point`的类，该类具有`x`坐标和`y`坐标的值。在构造函数（`constructor`）中，将实例对象的`x`和`y`属性赋值并初始化。

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

创建`Point`类的实例时使用`new`运算符。`new`运算符可以像函数调用一样传递参数。`new`运算符的参数传递给类的`constructor`方法（构造函数）。然后，在构造函数中执行实例对象的初始化处理。

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

这样，从类创建实例时必须使用`new`运算符。

一方面，类不能像普通函数一样调用。这是因为类的构造函数是初始化实例（`this`）的地方，与普通函数的作用不同。

```
class MyClass {
    constructor() {}
}
// クラスは関数として呼び出すことはできない
MyClass(); // => TypeError: class constructors must be invoked with |new| 
```

此外，构造函数可以使用`return`语句返回任意对象，但不应这样做。因为通常情况下，使用`new`运算符调用类，期望其返回类实例。

按照下面的代码，构造函数中返回的值将成为使用`new`运算符调用时的返回值。这种写法容易引起混淆，应避免使用。

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

### [](#class-name-start-upper-case)*[注意] 类名以大写字母开头*

*在 JavaScript 中，习惯上类名以大写字母开头。这与变量名使用驼峰式命名的习惯相同，名称本身没有特别的规则。将类名大写，其实例以小写开头，这样可以避免名称冲突，这是被广泛接受的理由。

```
class Thing {}
const thing = new Thing(); 
```

### [](#class-vs-function)*[专栏] `class`语法与函数的区别*

*在 ES2015 之前，这些类不是用`class`语法，而是用函数来表示的。这种表示方法因人而异，这也是引入`class`语法统一记法的理由之一。

下面的代码是使用函数实现类的一个示例。这个函数中的类表示省略了继承机制等，但与`class`语法很相似。

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

一个显著的区别是，用`class`语法定义的类不能像函数一样调用。类是使用`new`运算符实例化的，所以这是防止类误用的规范。另一方面，函数中的类表示只是函数，当然可以像函数一样调用。

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

这样，用函数实现类似类的东西时，会出现可以像函数一样调用的一个问题。为了避免这种问题，类使用`class`语法来实现。

## [](#class-prototype-method-definition)*类的原型方法定义*

*类的行为可以通过方法来定义。`constructor`方法是在初始化时调用的特殊方法，但在`class`结构中，可以自由地为类定义方法。这个类中定义的方法是创建的实例所具有的行为。

如下所示，在`class`结构中可以为类定义方法。要引用类的方法，可以使用与`constructor`方法相同的方式使用`this`。在这个类的这个方法中，`this`与我们在“函数与 this”章节中学到的函数的`this`一样，都引用了基对象。

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

在定义类的原型方法时，与对象中的方法不同，不能像`key : value`那样使用`:`分隔符来定义方法。也就是说，以下写法会导致语法错误（`SyntaxError`）。

```
// クラスでは次のようにメソッドを定義できない
class クラス {
   // SyntaxError
   メソッド: () => {}
   // SyntaxError
   メソッド: function(){}
} 
```

在这种方法定义的语法中，为类定义的方法是**共享的方法**。在实例间共享的方法被称为**原型方法**。

在下面的代码中，我们为`Counter`类定义了`increment`方法。这时，`Counter`类的实例各自持有不同的状态（`count`属性）。

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

这里的`increment`方法被定义为原型方法。原型方法是实例间（`counterA`和`counterB`）共享的。因此，可以知道每个实例的`increment`方法的引用是相同的。

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

为什么原型方法在实例间共享，这与类的继承机制密切相关。关于原型方法的机制将在后面进行说明。

在这里，如果我们理解了使用以下这种结构定义类方法，那么在各个实例中定义共享的原型方法就不会有问题。

```
class クラス {
    メソッド() {
        // このメソッドはプロトタイプメソッドとして定義される
    }
} 
```

## [](#class-accessor-property)*定义类的访问器属性*

*可以为类定义方法。但是，方法需要像`实例名.方法名()`这样调用。在类中，可以定义在引用属性或向属性赋值时调用的特殊方法。这个方法像属性一样表现，因此被称为**访问器属性**。

在这个代码中，我们定义了用于属性引用（getter）和属性赋值（setter）的访问器属性。访问器属性只需在方法名（属性名）前加上`get`或`set`即可。getter（`get`）没有参数，但必须返回一个值。setter（`set`）的参数是赋给属性的值，但不需要返回值。

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

在下面的代码中，我们将`NumberWrapper`类的`value`属性定义为访问器属性。可以看到，当访问`value`属性时，分别调用了定义的 getter 和 setter。这个访问器属性实际上是在读取和写入`NumberWrapper`实例的`_value`属性。

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

### [](#underbar-private-property)*[专栏] 以下划线（_）开头的属性名*

*在 NumberWrapper 的`value`的访问器属性中实际读写的是`_value`属性。这样，将不需要直接读写的外部属性命名为以下划线（_）开头，只是习惯问题，并没有语法上的意义。

从 ECMAScript 2022 开始，添加了用于定义外部直接读写不希望进行的私有属性的 Private 类字段结构。在 Private 类字段结构中，在属性名前加上`#`（井号）记号。因此，随着 Private 类字段结构的利用增加，以`_`开头命名属性的习惯将逐渐消失。

关于私有类字段结构，将在后面进行说明。

### [](#array-like-length)*使用访问器属性再现`Array.prototype.length`*

*不使用 getter 或 setter 实现起来比较困难的是`Array.prototype.length`属性。当向`Array`的`length`属性赋值时，其后的元素会自动删除。*

在下面的代码中，当减小数组的元素数（`length`属性）时，数组元素被删除。

```
const array = [1, 2, 3, 4, 5];
// 要素数を減らすと、インデックス以降の要素が削除される
array.length = 2;
console.log(array.join(", ")); // => "1, 2"
// 要素数だけを増やしても、配列の中身は空要素が増えるだけ
array.length = 5;
console.log(array.join(", ")); // => "1, 2, , , " 
```

试着实现一个可以再现`length`属性行为的`ArrayLike`类。`Array`的`length`属性在向`length`属性赋值时会执行以下操作。

+   当指定的**元素数**小于当前元素数时，更改**元素数**，并删除数组的末尾元素。

+   当指定的**元素数**大于当前元素数时，只更改**元素数**，实际数组元素保持不变。

通过在`ArrayLike`的`length`属性 setter 中实现元素的添加或删除，可以实现对类似数组的`length`属性的实现。

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

这样，访问器属性虽然像属性一样，但在实际访问时可以实现与其他属性联动的行为。

## [](#public-class-fields)*[ES2022] 公共类字段*

*在类中，我们在`constructor`方法中介绍了如何初始化类状态，即实例的属性。在之前介绍的`Counter`类中，我们在`constructor`方法中将`count`属性的初始值定义为`0`。

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

在这个`Counter`中，由于没有通过`new`运算符传递任何参数来初始化，因此没有在`constructor`方法中定义任何参数。在这种情况下，即使不写`constructor`方法，也无法初始化属性，因此存在一些麻烦的问题。

在 ES2022 中，为了更清晰地声明类实例拥有的属性初始化，新增了**类字段**结构。

类字段定义类实例拥有的属性，其结构如下。

```
class クラス {
    プロパティ名 = プロパティの初期値;
} 
```

将`Counter`类重写为类字段，如下所示。将`count`属性定义为类字段，其初始值为`0`。

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

类字段定义的是类实例拥有的属性。因此，与在`constructor`方法中定义`this.count = 0`类似，结果几乎相同。在类字段中定义的属性可以在类内部像其他属性一样通过`this.属性名`进行引用。

类字段可以与`constructor`方法一起使用。在下面的代码中，分别使用类字段和`constructor`方法定义了实例属性。

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

此外，类字段的初始化处理会在之后进行`constructor`中属性定义的处理。因此，如果存在相同名称的属性定义，则`constructor`方法中的定义会覆盖属性。

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

就像`publicField`属性这样，可以在类外部访问的属性被定义为**公共类字段**。

### [](#declare-class-fields)*使用类字段声明属性的存在*

*在类字段中，属性的初始值是可选的。因此，可以像下面这样定义省略了初始值的公共类字段。

```
class MyClass {
    // myPropertyはundefinedで初期化される
    myProperty;
} 
```

在这种情况下，`myProperty`被初始化为`undefined`。省略了初始值的类字段定义可以用于明确指出类实例拥有的属性。

在调用`load`方法之前，`Loader`类的`loadedContent`属性的值是`undefined`。使用类字段可以显式地表达`Loader`类的实例拥有`loadedContent`属性。

```
class Loader {
    loadedContent;
    load() {
        this.loadedContent = "読み込んだコンテンツ内容";
    }
} 
```

在 JavaScript 中，即使对象的属性在初始化时不存在，也可以在之后进行赋值来创建它。因此，即使像下面这样实现`Loader`类，其意义也是相同的。

```
class Loader {
    load() {
        this.loadedContent = "読み込んだコンテンツ内容";
    }
} 
```

然而，如果这样实现，使用`Loader`类的用户可能不知道`loadedContent`属性的存在，除非在`load`方法中读取。为此，可以使用类字段显式地表达“`Loader`类拥有`loadedContent`属性”这一事实。通过显式地定义属性，可以实现代码补全或使代码更易于阅读等好处。

### [](#this-in-class-fields)*类字段中的`this`表示类的实例*

*类字段的初始值可以写任意表达式，也可以使用`this`。在类字段中，`this`引用的是该类的实例。

在下面的代码中，将`up`字段的初始值指定为`increment`方法。由于 JavaScript 中函数也可以作为值处理，因此调用`up`方法会调用`increment`方法。

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

类字段中的`this`与箭头函数结合非常强大。

在下面的代码中，将`up`方法定义为箭头函数，并在函数内部调用`this.increment`方法。在箭头函数中定义的函数中的`this`不会因调用方式的不同而变化（请参阅“使用箭头函数处理回调函数”）。因此，无论以何种方式调用`up`方法，`this`都会指向类的实例，因此可以确保调用`increment`方法。

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

### [](#difference-between-class-fields-and-instance-property)*[专栏] 类字段与实例属性的区别*

在类字段中定义的属性或方法将被定义为类的实例属性。因此，类字段在`constructor`中为`this`添加属性时，在意义上几乎相同，并且可以将其视为一种使结构更易于理解的语法。

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

然而，严格来说，这两个属性定义之间还是存在差异的。通过以下方式定义类字段和`constructor`中`this`的 setter，可以了解这种差异。

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

对于类字段名的 setter 不会调用，而对于`this.property`的赋值会调用 setter。这是因为类字段不是通过使用`=`进行赋值来定义的，而是使用[Object.defineProperty](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)方法来定义属性。使用`Object.defineProperty`定义属性时，setter 会被忽略，属性会被定义。setter 会对`=`赋值做出反应。因此，在`constructor`中对`this.property`的赋值会调用 setter。

即使是相同的属性定义，由于属性定义的方式略有不同，因此存在这种行为差异。然而，为了避免编写需要意识到这种差异的代码，最好是避免这样做。实际上，从外观上很难意识到这种差异，而且编写旨在引起注意的代码会使复杂性增加。

## [](#private-class-fields)*[ES2022] 私有类字段*

*使用类字段结构如下所示，定义的属性在类实例化后也可以从外部访问。因此，被称为**公共类字段**。

```
class クラス {
    プロパティ名 = プロパティの初期値;
} 
```

另一方面，也存在一些不希望外部访问的实例属性。ES2022 中已经添加了定义此类私有属性的结构。

私有类字段是通过在字段名前加上 `#` 来定义的类字段。

```
class クラス {
    // プライベートなプロパティは#をつける
    #フィールド名 = プロパティの初期値;
} 
```

定义了私有类字段后，可以通过 `this.#fieldName` 来引用。

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

让我们更具体地看看私有类字段的用法。以之前出现的 `NumberWrapper` 类为例，使用私有类字段来重写它。在原始的 `NumberWrapper` 类中，实际值是通过 `_value` 属性进行读写操作的。在这种情况下，`_value` 属性可以从外部访问，因此定义的 getter 和 setter 会被忽略。

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

在私有类字段中，可以通过在字段名前加上 `#` 来定义不希望外部访问的属性。下面的代码中，`#value` 被定义为私有属性，因此会引发语法错误，无法从外部访问。使用私有类字段，在利用类时，必须通过 getter 和 setter 来引用 `#value`，而不是直接引用。

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

使用私有类字段可以声明不希望从类外部访问的属性。这可以防止对实现类的不当使用，或者防止从类外部直接修改属性的状态。

此外，私有类字段中，如果中间有值进入，则字段声明是必需的。在下面的代码中，`#loadedContent` 实际上是在 `load` 方法调用时填充的。在公共类字段中，字段定义是可选的，但在私有类字段中，`#loadedContent` 字段的定义是必需的。换句话说，在私有类字段中，需要在定义类的阶段明确列出类中存在的所有私有类字段。

```
class PrivateLoader {
    // 途中で値が入る場合でも最初に`undefined`で初期化されるフィールドの定義が必須
    #loadedContent;
    load() {
        this.#loadedContent = "読み込んだコンテンツ内容";
    }
} 
```

## [](#static-method)*静态方法*

*实例方法是通过实例化类来使用的。另一方面，也有可以不实例化类就使用的静态方法（类方法）。

静态方法的定义方法是在方法名前加上 `static`。

```
class クラス {
    static メソッド() {
        // 静的メソッドの処理
    }
}
// 静的メソッドの呼び出し
クラス.メソッド(); 
```

在下面的代码中，定义了一个名为 `ArrayWrapper` 的类来包装数组。`ArrayWrapper` 接受数组作为构造函数的参数进行初始化。这个类定义了一个静态方法 `ArrayWrapper.of`，可以接受数组中的元素本身作为参数来实例化。

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

在类的静态方法中，`this` 指向的是该类自身。因此，之前的代码可以用 `new this` 代替 `new ArrayWrapper`。

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

这样，静态方法中的 `this` 指向的是类自身，因此无法引用类的实例。因此，静态方法用于编写创建类实例的处理或与类相关的处理。

### [](#static-class-fields)*[ES2022] 静态类字段*

*ES2022 中添加的类字段中，可以使用静态类字段来在类本身上定义，而不是在实例上定义。

静态类字段只需在字段前加上 `static`。使用静态类字段定义的属性将成为类的属性。在下面的代码中，使用公共静态类字段为 `Colors` 类本身定义了属性。

```
class Colors {
    static GREEN = "緑";
    static RED = "赤";
    static BLUE = "青";
}
// クラスのプロパティとして参照できる
console.log(Colors.GREEN); // => "緑" 
```

此外，私有类字段也可以静态使用。私有静态类字段用于在类本身上定义属性，而不希望从外部引用。私有静态类字段只需在字段前加上 `static`。

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

## [](#two-instance-method-definition)*在原型中定义的方法与在实例中定义的方法的区别*

到目前为止，我们已经看到了两种方法：在原型对象中定义的方法和通过类字段定义的实例方法。原型方法的定义方法是将方法定义在特殊对象原型对象中。另一方面，通过类字段定义的方法是为类的实例定义方法。

两种方法定义方法中，都可以通过 `new` 操作符从实例化的对象中调用方法。这一点是相同的。

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

然而，这两种方法定义方法的区别在于方法定义的对象不同。

首先，我们来看这两种方法分别定义在不同的地方。下面的代码中，`ConflictClass` 类分别对原型和实例定义了同名的方法 `method`。

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

总结来说，在这种情况下，定义在实例对象上的 `method` 被调用。此时，如果使用 `delete` 操作符删除实例的 `method` 属性，那么接下来会调用原型方法的 `method`。

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

从这个执行结果中，我们可以得出以下结论。

+   原型方法和实例对象的方法定义不会相互覆盖，两者都存在。

+   实例对象的方法比原型对象的方法有更高的优先级。

两者都很难注意到，但这个行为是 JavaScript 的重要机制，因此理解它很重要。

这种行为是通过称为 **原型对象** 的特殊对象和称为 **原型链** 的机制实现的。由于两者都带有 **原型**，因此它们似乎是一组机制。

在下一节中，我们将探讨 **原型对象** 和 **原型链** 的机制。

## [](#prototype)*原型对象*

***原型方法**和**实例对象的方法**可以同时定义，但它们的方法不会相互覆盖。因为原型方法定义在**原型对象**上，而实例对象的方法定义在**实例对象**上。

关于原型对象，我在“原型对象”章节中简单介绍过，但在这里我将重新解释。

**原型对象**是 JavaScript 中函数对象的`prototype`属性自动创建的特殊对象。由于类也是函数对象的一种，因此会自动创建`prototype`属性的原型对象。

在下面的代码中，可以看到函数或类自身的`prototype`属性会自动创建原型对象。

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

`class`语法的`method`定义是作为原型对象的属性定义的。

在下面的代码中，可以看到类的方法定义在原型对象上。此外，类必须定义`constructor`方法（构造函数）。这个`constructor`方法也定义在原型对象上，这个`constructor`属性引用了类自身。

```
class MyClass {
    method() {}
}

console.log(typeof MyClass.prototype.method === "function"); // => true
// クラスのconstructorはクラス自身を参照する
console.log(MyClass.prototype.constructor === MyClass); // => true 
```

正如这样，原型方法是在原型对象上定义的，与实例对象的`method`属性是不同的对象定义的。因此，即使使用不同的方法定义方法，也不会发生覆盖。

## [](#prototype-chain)*原型链*

*使用`class`语法定义的原型方法是定义在原型对象上的。即使实例（对象）上没有定义方法，也可以从实例调用类的原型方法。

```
class MyClass {
    method() {
        console.log("プロトタイプのメソッド");
    }
}
const instance = new MyClass();
instance.method(); // "プロトタイプのメソッド" 
```

可以通过**原型链**机制从原型对象中定义的原型方法调用实例。

+   在实例创建时，将原型对象的引用保存到实例的`[[Prototype]]`内部属性中的处理

+   当从实例引用属性（或方法）时，会进行到`[[Prototype]]`内部属性为止的探索处理

### [](#write-prototype-chain)*实例创建与原型链*

*在通过`new`运算符创建实例时，实例会保存对类原型对象的引用。这时，实例对类原型对象的引用是保存在实例对象的`[[Prototype]]`这个内部属性中的。

`[[Prototype]]`内部属性是 ECMAScript 规范中定义的内部表示，因此不能像普通属性那样访问。在这里，为了说明，我们使用`[[属性名]]`这种格式来表示 ECMAScript 规范上存在的内部属性。

虽然不能像普通属性那样访问`[[Prototype]]`内部属性，但可以通过`Object.getPrototypeOf`方法引用`[[Prototype]]`内部属性。

在下面的代码中，可以看到正在获取`instance`对象的`[[Prototype]]`内部属性。确认获取的结果引用了类的原型对象。

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

这里重要的是，实例知道它是由哪个类创建的，以及它知道其类的原型对象。

#### [](#inner-property)*[注意] `[[Prototype]]`内部属性读写*

*可以通过`Object.getPrototypeOf(对象)`读取`对象`的`[[Prototype]]`。另一方面，可以通过`Object.setPrototypeOf(对象, 原型对象)`将`对象`的`[[Prototype]]`设置为`原型对象`。此外，存在一个特殊的访问器属性`__proto__`，可以像普通属性一样处理`[[Prototype]]`内部属性。

然而，通常不会直接读写这些`[[Prototype]]`内部属性。此外，由于它们可以改变现有内置对象的操作，因此不应该随意处理。

### [](#read-prototype-chain)*属性的引用与原型链*

*查看原型对象的属性是如何从实例引用的。

当引用对象的属性时，即使对象自身没有属性，探索也不会结束。还会继续探索对象`[[Prototype]]`内部属性（规范上的内部属性）引用的原型对象。这与在指定作用域中找不到变量时，向外层作用域进行探索的作用域链类似。

也就是说，当对象探索属性时，会按照以下顺序进行，检查每个对象。如果所有对象都没有找到，则返回`undefined`。

1.  `instance`对象自身

1.  `instance`对象的`[[Prototype]]`引用（原型对象）

1.  如果哪里都没有，则返回`undefined`

在这段代码中，实例对象本身没有`method`属性。因此，实际引用的是类原型对象的`method`属性。

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

正如这样，即使实例对象没有定义`method`，也可以调用类的原型对象的`method`。在引用这个属性时，会从对象自身按顺序查找`[[Prototype]]`内部属性，这种按顺序查找的机制被称为**原型链**。

原型链的机制可以用模拟代码表示如下。

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

通过`extends`继承时，子类的原型对象的`[[Prototype]]`内部属性将设置为父类的原型对象。在此代码中，`Child.prototype`对象的`[[Prototype]]`内部属性将设置为`Parent.prototype`。

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
