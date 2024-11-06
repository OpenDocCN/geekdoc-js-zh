# 第一部分：基本语法

> 原文：[`jsprimer.net/basic/`](https://jsprimer.net/basic/)

第一部分将介绍 JavaScript 的基本语法和内置对象的用法。

## [](#summary)*目录*

*### [](#introduction)*JavaScript 简介*

*简要介绍了 JavaScript 的用途和具有的语言特性。

### [](#comments)*注释*

*介绍了 JavaScript 中注释的写法。注释不会被作为程序评估，因此用于对源代码进行说明。 

### [](#variables)*变量和声明*

*介绍了在 JavaScript 中声明变量的方法。将介绍 const、let、var 等声明变量的方法。将介绍这些操作的区别和使用方法。

### [](#read-eval-print)*值的评估和显示*

*介绍了在浏览器中执行 JavaScript 代码的方法。还将介绍如何解决程序执行时遇到的错误。将错误分为两种，语法错误和运行时错误，并介绍各自的问题和解决方法。

### [](#data-type)*数据类型和语法*

*介绍了 JavaScript 值的数据类型。数据类型大致分为原始类型和对象，将分别介绍每种数据类型并附有简单的代码示例。某些数据类型提供了用于简单描述数据的语法，也将一并介绍这些语法。

### [](#operator)*运算符*

*介绍了 JavaScript 中的运算符。由于运算符是用符号表示的，因此很难搜索。本章总结了主要的运算符。当遇到不熟悉的运算符时，只需重新阅读即可，无需逐个阅读。

### [](#implicit-coercion)*隐式类型转换*

*JavaScript 的隐式类型转换是导致意外行为的原因之一。将介绍隐式类型转换导致的具体代码示例以及难以预测的情况。还将介绍显式转换的方法。

### [](#function-declaration)*函数和声明*

*介绍了定义 JavaScript 函数的方法。将介绍函数声明、函数表达式、箭头函数等定义函数或方法的方法。还将介绍如何处理函数参数以及函数和方法的区别。

### [](#statement-expression)*语句和表达式*

*介绍了构成 JavaScript 源代码的语句和表达式的概念。通过理解语句和表达式的区别，可以了解何时应该加入分号。虽然内容较为抽象，但即使不完全理解也没有问题。

### [](#condition)*条件分支*

*介绍了用于处理 JavaScript 条件分支的 if 语句和 switch 语句。还将介绍如何解决嵌套条件分支导致可读性下降的问题。

### [](#loop)*循环和迭代处理*

*介绍了使用 for 循环和 while 循环进行迭代处理。还将介绍 Array 方法也可以实现相同的迭代处理。

### [](#object)*对象*

*Object 是构成对象基础的要素。将介绍对象和属性的创建、更新、删除等基本操作。

### [](#prototype-object)*原型对象*

*JavaScript 具有特殊的原型对象。通过原型对象，将方法等特性从一个对象继承到另一个对象。将介绍通过原型对象实现的继承机制。

### [](#array)*数组*

*数组是可以按顺序存储值的对象。将介绍数组的创建、更新、删除等基本操作以及实际用法。还将介绍数组中破坏性方法和非破坏性方法的区别。

### [](#string)*字符串*

*介绍了使用字符串文字创建字符串以及基本的字符串操作，如搜索和替换。还将介绍与正则表达式结合使用的字符串操作以及使用标记模板函数进行模板处理等。

### [](#string-unicode)*字符串和 Unicode*

*介绍了与 JavaScript 采用的 Unicode 相关的 String 方法。将介绍 String 方法和处理字符串时需要意识到的 UTF-16 编码。

### [](#wrapper-object)*包装对象*

*介绍了 JavaScript 中原始类型的值如何调用内置对象的方法等机制，即包装对象。将确认原始类型如何在运行时转换为对象。

### [](#function-scope)*函数和作用域*

*介绍了决定变量等可引用范围的概念，称为作用域。将介绍块作用域、函数作用域等的作用以及多个作用域重叠时变量引用的决定方式。还将介绍与作用域相关的特性，如闭包。

### [](#function-this)*函数和 this*

*介绍了 JavaScript 中关键字`this`的工作原理。由于`this`的引用取决于条件，因此将根据不同条件介绍`this`的工作方式。介绍了如何通过代码示例使看似复杂的`this`行为变得可预测。

### [](#class)*类*

*介绍了 JavaScript 中类的定义和继承方法。介绍了基于原型的语言 JavaScript 是如何实现继承等功能的。

### [](#error-try-catch)*异常处理*

*介绍了 JavaScript 中的异常处理。介绍了 try...catch 语法的用法和 Error 对象。还介绍了在发生错误时如何阅读错误消息以及调试方法。

### [](#async)*异步处理:回调/Promise/Async 函数*

*介绍了 JavaScript 中的异步处理。介绍了同步处理和异步处理的区别以及为什么异步处理变得重要。介绍了使用回调风格、Promise、Async 函数进行异步处理的方法。

### [](#map-and-set)*Map/Set*

*介绍了处理数据集合的内置对象 Map 和 Set。从创建对象的方法和更新方法到实际使用案例进行介绍。

### [](#json)*JSON*

*介绍了基于 JavaScript 对象字面量创建的数据格式 JSON。还介绍了如何在 JavaScript 中读写 JSON 的内置对象的用法。

### [](#date)*日期*

*介绍了处理日期和时间的内置对象 Date。

### [](#math)*数学*

*介绍了提供数学常数和函数的内置对象 Math。

### [](#module)*ECMAScript 模块*

*介绍了 JavaScript 模块（ECMAScript 模块）。

### [](#ecmascript)*ECMAScript*

*介绍了 JavaScript 规范 ECMAScript。介绍了 ECMAScript 的历史以及规范制定的过程。

### [](#other-parts)*第一部分: 结语*

*在第一部分结束时，简要介绍了书中未涉及的其他语法和内置对象。还介绍了如何查找不熟悉的语法和内置对象的用法。