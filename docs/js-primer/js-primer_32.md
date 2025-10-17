# 数学

> 原文：[`jsprimer.net/basic/math/`](https://jsprimer.net/basic/math/)

本章将介绍 JavaScript 中提供数学常数和函数的内置对象[Math](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math)。

## [](#math-object)*Math 对象*

*`Math`对象是内置对象，但不是构造函数。换句话说，`Math`对象不会创建实例，所有常数和函数都作为`Math`对象的静态属性和方法提供。例如，`Math.PI`属性表示圆周率π，`Math.sin`方法计算正弦值。下面的例子计算了 90 度的正弦值。90 度的正弦值为 1，因此`sin90`变量将返回 1。

```
const rad90 = Math.PI * 90 / 180;
const sin90 = Math.sin(rad90);
console.log(sin90); // => 1 
```

三角函数等许多函数和常数都是通过`Math`对象提供的。本章将介绍其中一些常用的，并附带使用案例。更详尽的解释请参考[MDN 参考文档](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math)。

### [](#create-random-number)*生成随机数*

*`Math`对象的主要用途之一是通过[Math.random](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/random)方法生成随机数。`Math.random`方法在 0 到 1 之间返回伪随机浮点数。

```
for (let i = 0; i < 5; i++) {
    // 毎回ランダムな浮動小数点数を返す
    console.log(Math.random());
} 
```

下面的例子中，使用`Math.random`方法生成任意范围内的随机数。

```
// minからmaxまでの乱数を返す関数
function getRandom(min, max) {
    return Math.random() * (max - min) + min;
}
// 1 以上 5 未満の浮動小数点数を返す
console.log(getRandom(1, 5)); 
```

### [](#compare-number)*比较数字大小*

*[Math.max](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/max)方法会返回传入的多个数字中的最大值。同样，[Math.min](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/min)方法会返回传入的多个数字中的最小值。

```
console.log(Math.max(1, 10)); // => 10
console.log(Math.min(1, 10)); // => 1 
```

这些方法可以接受可变数量的参数，因此可以比较任意数量的数字。在从数字数组中提取最大值和最小值时，使用`...`（展开语法）可以简洁地编写。

```
const numbers = [1, 2, 3, 4, 5];
console.log(Math.max(...numbers)); // => 5
console.log(Math.min(...numbers)); // => 1 
```

### [](#convert-to-integer)*转换为整数*

*`Math`对象有几种方法可用于将数字舍入为整数。其中常见的有向下取整的`Math.floor`方法、向上取整的`Math.ceil`方法，以及四舍五入的`Math.round`方法。

[Math.floor](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/floor)方法返回小于或等于传入数的最大整数。这种函数称为**底函数**。正数`1.3`将变为`1`，而负数`-1.3`将舍入为更小的整数`-2`。

[Math.ceil](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/ceil)方法返回大于或等于传入数的最小整数。这种函数称为**天花板函数**。正数`1.3`将变为`2`，而负数`-1.3`将舍入为更大的整数`-1`。

[Math.round](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/round)方法执行常见的四舍五入操作。如果小数部分小于`0.5`，则向下舍入，否则向上舍入。

```
// 床関数
console.log(Math.floor(1.3)); // => 1
console.log(Math.floor(-1.3)); // => -2
// 天井関数
console.log(Math.ceil(1.3)); // => 2
console.log(Math.ceil(-1.3)); // => -1
// 四捨五入
console.log(Math.round(1.3)); // => 1
console.log(Math.round(1.6)); // => 2
console.log(Math.round(-1.3)); // => -1 
```

此外，[Math.trunc](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Math/trunc)方法^([ES2015])会简单地截断传入数字的小数部分，返回整数。因此，当参数为正值时，结果与`Math.floor`方法相同，否则与`Math.ceil`方法相同。

```
// 単純に小数部分を切り落とす
console.log(Math.trunc(1.3)); // => 1
console.log(Math.trunc(-1.3)); // => -1 
```

## [](#conclusion)*总结*

*本章介绍了`Math`对象。所介绍的方法只是`Math`对象的一部分，还有其他方法可供使用。

+   `Math`提供数学常数和函数的内置对象

+   `Math`不是构造函数，无法实例化

+   提供了生成伪随机数、比较数字、进行数值计算等方法*****
