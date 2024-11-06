# 原型对象

> 原文：[`jsprimer.net/basic/prototype-object/`](https://jsprimer.net/basic/prototype-object/)

在“对象”章节中，我们已经看到了如何处理对象。 在其中，即使是空对象，也可以调用`toString`方法等。

```
const obj = {};
console.log(obj.toString()); // "[object Object]" 
```

尽管仅仅定义了一个空的对象文字，但我们却可以调用`toString`方法。 这个方法到底是在哪里实现的呢？

此外，在 JavaScript 中除了`toString`之外，还有一些自动实现的方法。 这些嵌入式的方法称为内置方法。

本章将确定内置方法的实现位置，以及为什么可以从`Object`的实例中调用。 由于详细的机制将在“类”章节中重新解释，因此本章的目的是了解大致的操作流程。

## [](#object-is-origin)*`Object`是所有的起源*

*`Object`与其他对象（如`Array`、`String`、`Function`等）有所不同的特点在于，其他对象都继承自`Object`。

更准确地说，几乎所有的对象都继承自`Object.prototype`属性中定义的`prototype`对象。 `prototype`对象是在创建所有对象时自动添加的特殊对象。 `Object`的`prototype`对象可以看作是提供给所有对象使用的基础对象。

![所有对象都继承自`Object`的`prototype`](img/a5724e6230250e26ed28c35e386f54c9.png)

让我们具体看看是什么意思。

如前所述，`toString`方法定义在`Object`的`prototype`对象中。 我们可以引用`Object.prototype.toString`方法的实现本身。

```
// `Object.prototype`オブジェクトに`toString`メソッドの定義がある
console.log(typeof Object.prototype.toString); // => "function" 
```

这些嵌入在`prototype`对象中的方法被称为**原型方法**。 本书有时会将这样的原型方法缩写为“Object 的`toString`方法”。

`Object`的实例会继承自这个`Object.prototype`对象中定义的方法和属性。 也就是说，通过对象文字或`new Object`实例化的对象可以使用`Object.prototype`中定义的内容。

在下面的代码中，我们从对象文字（object literal）创建（实例化）的对象中引用了`Object.prototype.toString`方法。 这样一来，实例的`toString`方法和`Object.prototype.toString`就会是相同的。

```
const obj = {
    "key": "value"
};
// `obj`インスタンスは`Object.prototype`に定義されたものを継承する
// `obj.toString`は継承した`Object.prototype.toString`を参照している
console.log(obj.toString === Object.prototype.toString); // => true
// インスタンスからプロトタイプメソッドを呼び出せる
console.log(obj.toString()); // => "[object Object]" 
```

像`Object.prototype`中定义的`toString`方法一样，这些方法在创建实例时会自动继承，因此可以从`Object`的实例中调用。 这使得即使是使用对象文字创建的空对象，也可以调用`Object.prototype.toString`等方法。

这种从实例引用到原型对象上定义的方法的机制称为**原型链**。 关于原型链的机制，我们将在“类”章节中进行讨论，因此在这里只需要了解从实例中调用原型方法即可。

### [](#prototype-shorthand-syntax)*[专栏] 关于`Object#toString`的缩写表示*

*在本书中，由于每次都写上`prototype`会显得冗长，因此有时会使用“Object 的`toString`方法”这样的缩写形式。 在除了本书之外的其他文本中，可能会使用`Object.prototype.toString`表示为`Object#toString`这样的形式，使用`#`代替`prototype`。

`#`被用作`prototype`的缩写表示，因为`#`并不是 JavaScript 语法中的符号。 详细信息将在“类”章节中解释，但是在 ES2022 中，`#`已经被添加到 JavaScript 的语法中，`#`这个符号有了具体的含义。 在 ES2022 之后，如果使用`#`作为`prototype`的缩写表示来进行解释，可能会产生不同的含义。

因此，本书不使用`Object#toString`这样的缩写形式来表示`Object.prototype.toString`，而是保持原样。

### [](#same-method-name-order)*原型方法和实例方法的优先级*

*有时，实例对象中也可能定义了与原型方法同名的方法。 在这种情况下，实例定义的方法将优先调用。

下面的代码定义了一个名为`customObject`的`Object`实例，并定义了`toString`方法。 执行代码后，我们发现实例方法优先于原型方法被调用。

```
// オブジェクトのインスタンスにtoStringメソッドを定義
const customObject = {
    toString() {
        return "custom value";
    }
};
console.log(customObject.toString()); // => "custom value" 
```

如此一来，当实例和原型对象中有相同名称的方法时，实例的方法将优先。

### [](#diff-in-operator-and-object-hasown)*`Object.hasOwn`静态方法和`in`运算符的区别*

在“对象”章节中，我们已经学习了`Object.hasOwn`静态方法和`in`运算符的行为差异。 这两种行为的差异与本章介绍的原型对象有关。

`Object.hasOwn`静态方法用于判断指定对象本身是否具有指定属性。而`in`运算符则会搜索对象本身是否具有该属性，如果没有，则会继续搜索其原型对象（`prototype`对象）。换句话说，`in`运算符不区分实例中实现的方法和原型对象中实现的方法。

下面的代码示例使用`Object.hasOwn`静态方法和`in`运算符分别判断空对象是否具有`toString`方法。`Object.hasOwn`静态方法返回`false`，而`in`运算符返回`true`，因为`toString`方法存在于原型对象中。

```
const obj = {};
// `obj`というオブジェクト自体に`toString`メソッドが定義されているわけではない
console.log(Object.hasOwn(obj, "toString")); // => false
// `in`演算子は指定されたプロパティ名が見つかるまで親をたどるため、`Object.prototype`まで見にいく
console.log("toString" in obj); // => true 
```

如下所示，如果实例具有`toString`方法，则`Object.hasOwn`静态方法也会返回`true`。

```
// オブジェクトのインスタンスにtoStringメソッドを定義
const obj = {
    toString() {
        return "custom value";
    }
};
// オブジェクトのインスタンスが`toString`メソッドを持っている
console.log(Object.hasOwn(obj, "toString")); // => true
console.log("toString" in obj); // => true 
```

### [](#create-method)*明确对象的继承源的`Object.create`方法*

*通过使用`Object.create`方法，可以创建一个新对象，该对象继承自指定的`prototype`对象。

通过前面的解释，我们知道对象字面量会自动继承`Object.prototype`对象。通过使用`Object.create`方法，可以创建新对象，如下所示。

```
// const obj = {} と同じ意味
const obj = Object.create(Object.prototype);
// `obj`は`Object.prototype`を継承している
// そのため、`obj.toString`と`Object.prototype.toString`は同じとなる
console.log(obj.toString === Object.prototype.toString); // => true 
```

### [](#inherit-object)*Array 也继承自 Object*

*`Object`和`Object.prototype`的关系与内置对象`Array`和`Array.prototype`的关系类似。同样地，数组（`Array`）的实例也会继承`Array.prototype`。此外，由于`Array.prototype`也继承了`Object.prototype`，因此数组（`Array`）的实例也会继承`Object.prototype`。

> `Array`的实例 → `Array.prototype` → `Object.prototype`

使用`Object.create`方法来表示`Array`和`Object`之间的关系。这段伪代��中，`Array`构造函数的实现等与实际情况有所不同，请注意这只是一个概念。

```
// このコードはイメージです！
// `Array`コンストラクタ自身は関数でもある
const Array = function() {};
// `Array.prototype`は`Object.prototype`を継承している
Array.prototype = Object.create(Object.prototype);
// `Array`のインスタンスは、`Array.prototype`を継承している
const array = Object.create(Array.prototype);
// `array`は`Object.prototype`を継承している
console.log(array.hasOwnProperty === Object.prototype.hasOwnProperty); // => true 
```

因此，由于`Array`的实例也继承自`Object.prototype`，因此可以使用`Object.prototype`中定义的方法。

下面的代码示例展示了从`Array`实例中访问`Object.prototype.hasOwnProperty`方法。

```
const array = [];
// `Array`のインスタンス -> `Array.prototype` -> `Object.prototype`
console.log(array.hasOwnProperty === Object.prototype.hasOwnProperty); // => true 
```

可以访问`hasOwnProperty`方法的原因也是由于原型链的机制。

请记住，`Object.prototype`是所有对象的父对象。因此，要记住`Array`和`String`等实例也可以使用`Object.prototype`的方法。

此外，`Array.prototype`等也定义了各自的方法。例如，`Array.prototype.toString`方法就是其中之一。因此，在数组的实例中调用`toString`方法时，会优先调用`Array.prototype.toString`。

```
const numbers = [1, 2, 3];
// `Array.prototype.toString`が定義されているため、`Object.prototype.toString`とは異なる出力形式となる
console.log(numbers.toString()); // => "1,2,3" 
```

## [](#not-inherit-object)*不继承`Object.prototype`的对象*

*虽然说`Object`是所有对象的父对象，但也存在例外情况。

尽管是一种惯用法，但通过`Object.create(null)`可以创建不继承`Object.prototype`的对象，从而创建一个真正**空的对象**。

```
// 親がnull、つまり親がいないオブジェクトを作る
const obj = Object.create(null);
// Object.prototypeを継承しないため、hasOwnPropertyが存在しない
console.log(obj.hasOwnProperty); // => undefined 
```

`Object.create`方法是从 ES5 引入的。通过`Object.create(null)`这种惯用法，一些库等曾将其用作`Map`对象的替代品。`Map`是用于保存键值对的对象。

普通对象也具有类似`Map`的特性，因为一开始就存在一些属性可以访问。这是因为`Object`的实例默认会继承`Object.prototype`，因此在创建对象时就存在像`toString`这样的属性名。因此，通过`Object.create(null)`创建不继承`Object.prototype`的对象，这些对象曾被用作`Map`的替代品。

```
// 空オブジェクトを作成
const obj = {};
// "toString"という値を定義してないのに、"toString"が存在している
console.log(obj["toString"]);// Function
// Mapのような空オブジェクト
const mapLike = Object.create(null);
// toStringキーは存在しない
console.log(mapLike["toString"]); // => undefined 
```

然而，自 ES2015 起，可以使用真正的`Map`，因此不再需要使用`Object.create(null)`来代替`Map`。关于`Map`的详细信息将在“Map/Set”章节中介绍。

由`Object.create(null)`创建的空对象也是 ES2022 引入`Object.hasOwn`静态方法的原因之一。

如下所示，不继承`Object.prototype`的对象无法调用`Object.prototype.hasOwnProperty`方法。因此，在检查对象是否具有属性时，无法简单地使用`hasOwnProperty`方法。

```
// Mapのような空オブジェクト
const mapLike = Object.create(null);
// `Object.prototype`を継承していないため呼び出すと例外が発生する
console.log(mapLike.hasOwnProperty("key")); // => Error: hasOwnPropertyメソッドは呼び出せない 
```

自 ES2022 起，`Object.hasOwn`静态方法可以使用，无论目标对象是否继承自`Object.prototype`。

```
// Mapのような空オブジェクト
const mapLike = Object.create(null);
// keyは存在しない
console.log(Object.hasOwn(mapLike, "key")); // => false 
```

这样，不依赖于目标对象的`Object.hasOwn`静态方法修正了`hasOwnProperty`方法的缺点。

## [](#conclusion)*总结*

*在本章中，我们学习了有关原型对象的知识。

+   原型对象会在创建对象时自动创建

+   `Object`的原型对象中定义了诸如`toString`等原型方法

+   大多数对象通过继承`Object.prototype`来调用`toString`方法等

+   原型方法和实例方法中，实例方法优先级更高

+   使用`Object.create`方法可以创建一个不继承原型对象的对象

确认了如何引用原型对象中定义的方法。关于这个原型的详细机制，我们将在“类”章节中再次解释。
