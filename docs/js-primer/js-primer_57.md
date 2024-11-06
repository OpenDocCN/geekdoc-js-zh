# 附录: JavaScript 速查表

> 原文：[`jsprimer.net/cheatsheet/`](https://jsprimer.net/cheatsheet/)

关于 JavaScript 语言功能的速查表。

+   言語機能

    +   评论

    +   数据

    +   文字文字

    +   字符串

    +   数据访问

    +   运算符

    +   函数和行为

    +   控制流

    +   模块

    +   其他

+   指南

    +   项目结构

## [](#language-feature)*语言功能*

*### [](#comments)*评论*

*关于评论的写法。

| 代码示例 | 描述 | 解释 | MDN |
| --- | --- | --- | --- |
| `// xxx` | 单行评论 | 评论 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E3%82%B3%E3%83%A1%E3%83%B3%E3%83%88) |
| `/* xxx */` | 多行评论 | 评论 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E3%82%B3%E3%83%A1%E3%83%B3%E3%83%88) |
| `<!-- xxx -->` | [ES2015] 类似 HTML 的评论 | 评论 |   |

### [](#data-structures)*数据结构*

*変数宣言について。

| 代码示例 | 描述 | 解释 | MDN |
| --- | --- | --- | --- |
| `const x` | [ES2015] **变量声明**。不能重新赋值给`x` | 变量和声明 | [const](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/const) |
| `let x` | [ES2015] **变量声明**。类似`const`，但`x`可以重新赋值 | 变量和声明 | [let](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/let) |
| `var x` | **变量声明**。传统的变量声明方法 | 变量和声明 | [var](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/var) |

### [](#literal)*文字*

*关于表示数据结构的文字。

| 代码示例 | 描述 | 解释 | MDN |
| --- | --- | --- | --- |
| `true` 或 `false` | **布尔值** | 数据类型和文字 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E8%AB%96%E7%90%86%E5%80%A4%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB) |
| `123` | **10 进制**的整数文字 | 数据类型和文字 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E6%95%B0%E5%80%A4%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB) |
| `123n` | [ES2020] 表示巨大整数的 BigInt 文字 | 数据类型和文字 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E6%95%B0%E5%80%A4%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB) |
| `0b10` | [ES2015] **2 进制**的整数文字 | 数据类型和文字 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E6%95%B0%E5%80%A4%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB) |
| `0o777` | [ES2015] **8 进制**的整数文字 | 数据类型和文字 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E6%95%B0%E5%80%A4%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB) |
| `0x30A2` | **16 进制**的整数文字 | 数据类型和文字 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E6%95%B0%E5%80%A4%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB) |
| `123_456` | [ES2021] 数值文字中的**数字分隔���** | 数据类型和文字 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E6%95%B0%E5%80%A4%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB) |
| `{ k: v }` | 创建属性名为`k`，属性值为`v`的**对象** | 对象 | [语法和数据类型](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Grammar_and_types#object_literals) |
| *`{ n }`* | [ES2015] 创建属性名为`n`，属性值为`n`的**对象** | 对象 | [对象初始化器](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Object_initializer#%E3%83%97%E3%83%AD%E3%83%91%E3%83%86%E3%82%A3%E3%81%AE%E5%AE%9A%E7%BE%A9) |
| `[x, y]` | 创建具有初始值`x`和`y`的**数组对象** | 数组 | [语法和数据类型](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Grammar_and_types#array_literals) |
| `/pattern/` | 创建具有`pattern`的**正则表达式对象** | 字符串 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E6%AD%A3%E8%A6%8F%E8%A1%A8%E7%8F%BE%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB) |
| `null` | `null`值 | 数据类型和文字 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#null_%E3%83%AA%E3%83%86%E3%83%A9%E3%83%AB) |

### [](#string)*字符串*

*文字列について。

| 代码示例 | 描述 | 解释 | MDN |
| --- | --- | --- | --- |
| `"xxx"` | 双引号的**字符串文字**。换行等使用`\`和特定字符的组合表示。 | 字符串 | [语法和数据类型](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Grammar_and_types#string_literals) |
| `'xxx'` | 单引号的**字符串文字**。与`"xxx"`的意义相同。 | 字符串 | [语法和数据类型](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Grammar_and_types#string_literals) |
| ``xxx`` | [ES2015] 模板字符串文字。可以包含换行符 | 字符串 | [模板文字（模板字符串）](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Template_literals) |
| *``${x}``* | [ES2015] 展开模板字符串文字中变量`x`的值 | 字符串 | [模板文字（模板字符串）](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Template_literals) |

### [](#data-access)*数据访问*

*数据访问。

| 代码示例 | 描述 | 解释 | MDN |
| --- | --- | --- | --- |
| `array[0]` | 数组的**索引访问** | 数组 | [Array](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Array#%E6%B7%BB%E5%AD%97%E3%81%AB%E3%82%88%E3%82%8B%E9%85%8D%E5%88%97%E3%81%AE%E8%A6%81%E7%B4%A0%E3%81%B8%E3%81%AE%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9) |
| `obj["x"]` | 对象的**属性访问**（方括号表示法） | 对象 | [属性访问器](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Property_Accessors) |
| `obj.x` | 对象的**属性访问**（点表示法） | 对象 | [属性访问器](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Property_Accessors) |
| `obj?.x` | [ES2020] 对象的**属性访问**（可选链操作符） | 对象 | [可选链 (?.)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Optional_chaining) |
| `const { x } = obj;` | [ES2015] 对象的**解构赋值** | 对象 | [解构赋值](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) |
| `const [ x ] = array;` | [ES2015] 使用数组的**解构赋值** | 数组 | [解构赋值](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) |
| `f(...array)` | [ES2015] 使用**扩展语法**扩展数组 | 数组 | [展开语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax) |
| `f({ ...obj })` | [ES2018] 使用**扩展语法**扩展对象 | 对象 | [展开语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Spread_syntax) |

### [](#operator)*操作符*

*关于操作符。*

| 代码示例 | 描述 | 解释 | MDN |
| --- | --- | --- | --- |
| `+` | 加号/字符串连接操作符 | 操作符 | [加法 (+)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Addition) |
| `-` | 减法操作符 | 操作符 | [减法 (-)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Subtraction) |
| `*` | 乘法操作符 | 操作符 | [乘法 (*)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Multiplication) |
| `/` | 除法操作符 | 操作符 | [除法 (/)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Division) |
| `%` | 取模操作符 | 操作符 | [取模 (%)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Remainder) |
| `**` | [ES2016] 指数操作符 | 操作符 | [指数 (**)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Exponentiation) |
| `++` | 自增操作符 | 操作符 | [自增](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Increment) |
| `--` | 自减操作符 | 操作符 | [自减 (--)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Decrement) |
| `===` | 严格等于操作符 | 操作符 | [严格相等 (===)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Strict_equality) |
| `!==` | 严格不等于操作符 | 操作符 | [严格不等于 (!==)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Strict_inequality) |
| `==` | 等于操作符 | 操作符 | [等于 (==)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Equality) |
| `!=` | 不等于操作符 | 操作符 | [不等于 (!=)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Inequality) |
| `>` | 大于操作符 | 操作符 | [大于 (>)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Greater_than) |
| `>=` | 大なりイコール演算子/以上 | 演算子 | [大なりイコール (>=)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Greater_than_or_equal) |
| `<` | 小なり演算子/より小さい | 演算子 | [小なり (<)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Less_than) |
| `<=` | 小なりイコール演算子/以下 | 演算子 | [小なりイコール (<=)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Less_than_or_equal) |
| `&` | ビット論理積 | 演算子 | [ビット論理積 (&)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Bitwise_AND) |
| `&#124;` | ビット論理和 | 演算子 | [ビット論理和 (&#124;)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Bitwise_OR) |
| `^` | ビット排他的論理和 | 演算子 | [ビット排他的論理和 (^)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Bitwise_XOR) |
| `~` | ビット否定 | 演算子 | [ビット否定 (~)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Bitwise_NOT) |
| `<<` | 左シフト演算子 | 演算子 | [左シフト (<<)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Left_shift) |
| `>>` | 右シフト演算子 | 演算子 | [右シフト (>>)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Right_shift) |
| `>>>` | ゼロ埋め右シフト演算子 | 演算子 | [符号なし右シフト (>>>)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Unsigned_right_shift) |
| `&&` | AND 演算子 | 演算子 | [論理積 (&&)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Logical_AND) |
| `??` | [ES2020] Nullish coalescing 演算子 | 演算子 | [Null 合体演算子 (??)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing) |
| `&#124;&#124;` | OR 演算子 | 演算子 | [論理和 (&#124;&#124;)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Logical_OR) |
| `(x)` | グループ演算子 | 演算子 | [グループ化演算子 ( )](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Grouping) |
| `x, y` | カンマ演算子 | 演算子 | [カンマ演算子 (,)](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Comma_Operator) |

### [](#function-behavior)*関数と挙動*

*関数の定義と挙動について。

| サンプル | 説明 | 解説 | MDN |
| --- | --- | --- | --- |
| `function f(){}` | **関数**宣言 | 関数と宣言 | [関数宣言](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/function) |
| `const f = function(){};` | **関数**式 | 関数と宣言 | [関数式](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/function) |
| `const f = () => {};` | [ES2015] **Arrow Function**の宣言 | 関数と宣言 | [アロー関数式](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Arrow_functions) |
| `async function f(){}` | [ES2017] **异步函数**的声明 | 异步处理 | [异步函数](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/async_function) |
| *`const f = async function(){};`* | [ES2017] 使用**异步函数**声明的函数表达式 | 异步处理 | [异步函数表达式](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/async_function) |
| *`const f = async () => {};`* | [ES2017] 使用箭头函数声明的**异步函数** | 异步处理 | [异步函数表达式](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/async_function) |
| `function f(x, y){}` | 函数中的形参声明 | 函数和声明 | [函数声明](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/function) |
| *`function f(x = 1, y = 2){}`* | **默认参数**，指定参数未传递时的初始值。 | 函数和声明 | [默认参数](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Default_parameters) |
| *`function f([x, y]){}`* | [ES2015] 函数参数中的数组**解构赋值**。从参数数组中将索引为`0`的值赋给`x`，索引为`1`的值赋给`y`。 | 函数和声明 | [解构赋值](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#%E9%85%8D%E5%88%97%E3%81%AE%E5%88%86%E5%89%B2%E4%BB%A3%E5%85%A5) |
| *`function f({ x, y }){}`* | [ES2015] 函数参数中的对象**解构赋值**。从参数对象中接收`x`和`y`属性。 | 函数和声明 | [解构赋值](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E5%88%86%E5%89%B2%E4%BB%A3%E5%85%A5) |
| *`function f(...args)){}`* | [ES2015] **可变长参数**/**Rest parameters**。以数组形式接收传递的参数 | 函数和声明 | [剩余参数](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/rest_parameters) |
| `const o = { method: function(){} };` | **方法定义** | 函数和声明 | [方法定义](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Method_definitions#%E8%A7%A3%E8%AA%AC) |
| `const o = { method(){} };` | [ES2015] **方法定义**的简写 | 函数和声明 | [方法定义](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Functions/Method_definitions#%E8%A7%A3%E8%AA%AC) |
| `class X{}` | [ES2015] **类**声明 | 类 | [class](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/class) |
| `const X = class X{};` | [ES2015] **类**表达式 | 类 | [类表达式](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/class) |
| *`class X{ method(){} }`* | [ES2015] 定义类的**实例方法** | 类 | [类](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Classes#%E3%83%97%E3%83%AD%E3%83%88%E3%82%BF%E3%82%A4%E3%83%97%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89) |
| *`class X{ get x(){}, set x(v){} }`* | [ES2015] 类的**存取器方法**定义 | 类 | [对象操作](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Working_with_Objects#defining_getters_and_setters) |
| *`class X{ property = 1; }`* | [ES2022] 类的**公共类字段**定义 | 类 | [公共类字段](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Classes/Public_class_fields) |
| *`class X{ #privateField = 1; }`* | [ES2022] 类的**私有类字段**定义 | 类 | [私有类字段](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Classes/Private_class_fields) |
| *`class X{ static method(){} }`* | [ES2015] 类的**静态方法**定义 | 类 | [static](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Classes/static) |
| `class C extends P{}` | [ES2015] 类的**继承** | 类 | [extends](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Classes/extends) |
| *`super`* | [ES2015] 引用**父类** | 类 | [super](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/super) |
| `fn()` | 函数调用 | 函数和声明 | [函数](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Functions#calling_functions) |
| `fn`xxx`` | [ES2015] 标签函数调用 | 字符串 | [模板文字（模板字符串）](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Template_literals#%E3%82%BF%E3%82%B0%E4%BB%98%E3%81%8D%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88) |
| `new X()` | 使用`new`运算符从类（具有构造函数的对象）创建实例 | 类 | [new 运算符](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/new) |

### [](#control-flow)*控制流*

*控制流的控制结构。

| 例 | 描述 | 解释 | MDN |
| --- | --- | --- | --- |
| `while(x){}` | **while 循环**。当`x`为`true`时执行迭代处理 | 循环和迭代处理 | [while](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/while) |
| `do{}while(x);` | **do...while 循环**。当`x`为`true`时执行迭代处理。无论`x`如何，都会执行第一次处理 | 循环和迭代处理 | [do...while](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/do...while) |
| `for(let x=0;x < y ;x++){}` | **for 循环**。当`x < y`为`true`时执行迭代处理 | 循环和迭代处理 | [for](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/for) |
| `for(const p in o){}` | **for...in 循环**。对对象（`o`）的属性(`p`)进行迭代处理 | 循环和迭代处理 | [for...in](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/for...in) |
| `for(const x of iter){}` | [ES2015] **for...of 循环**。对迭代器(`iter`)进行迭代处理 | 循环和迭代处理 | [for...of](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/for...of) |
| `if(x){/*A*/}else{/*B*/}` | **条件表达式**。如果`x`为`true`，则执行 A，否则执行 B | 条件分支 | [if...else](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/if...else) |
| `switch(x){case "A":{/*A*/} "B":{/*B*/}}` | **switch 语句**。如果`x`为`"A"`，则执行 A，如果为"B"，则执行 B | 条件分支 | [switch](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/switch) |
| `x ? A: B` | **条件（三元）运算符**。如果`x`为`true`，则执行`A`，否则执行`B` | 条件分支 | [条件（三元）运算符](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) |
| `break` | **break 语句**。结束当前循环迭代，跳出循环。 | 条件分支 | [break](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/break) |
| `continue` | **continue 语句**。结束当前循环迭代，进入下一个循环。 | 条件分支 | [continue](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/continue) |
| `try{}catch(e){}finally{}` | `try...catch`语法 | 异常处理 | [try...catch](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/try...catch) |
| `throw new Error("xxx")` | `throw`语句 | 异常处理 | [throw](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/throw) |

### [](#module)*模块*

*关于 ECMAScript 模块。

| 代码 | 描述 | 解释 | MDN |
| --- | --- | --- | --- |
| `import x from "./x.js"` | [ES2015] **デフォルトインポート** | ECMAScriptモジュール | [Import](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import) |
| `import { x } from "./x.js"` | [ES2015] **命名导入** | ECMAScript 模块 | [Import](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import) |
| `import { x as y } from "./x.js"` | [ES2015] 名前付きインポート的**别名** | ECMAScript 模块 | [Import](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import) |
| `import * as x from "./x.js"` | [ES2015] 导入**所有命名导出并取别名** | ECMAScript 模块 | [Import](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import) |
| `import "./x.js"` | [ES2015] 为了**副作用**而导入 | ECMAScript 模块 | [Import](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import) |
| `export default x` | [ES2015] **默认导出** | ECMAScript 模块 | [Export](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export) |
| `export { x }` | [ES2015] **命名导出** | ECMAScript 模块 | [Export](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export) |
| `export { x as y }` | [ES2015] 命名导出的**别名** | ECMAScript 模块 | [Export](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export) |
| `export { x } from "./x.js"` | [ES2015] **重新导出**命名导出 | ECMAScript 模块 | [Export](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export) |
| `export * from "./x.js"` | [ES2015] **重新导出**所有命名导出 | ECMAScript 模块 | [Export](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export) |
| `export * as ns from "./x.js"` | [ES2020] **导入所有命名导出**，并以`ns`的名称**重新导出** | ECMAScript 模块 | [Export](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/export) |

### [](#miscellaneous)*其他*

*| 代码示例 | 描述 | 解释 | MDN |

| --- | --- | --- | --- |
| --- | --- | --- | --- |
| `x;` | **语句** | 语句和表达式 | [词法语法](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Lexical_grammar#%E8%87%AA%E5%8B%95%E3%82%BB%E3%83%9F%E3%82%B3%E3%83%AD%E3%83%B3%E6%8C%BF%E5%85%A5) |
| `{ }` | 代码块 | 语句和表达式 | [块](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/block) |

## [](#guide)*指南*

*### [](#project-anatomy)*项目结构*

*关于 JavaScript 基本项目布局、文件、文件夹结构的内容。

| 名称 | 描述 |
| --- | --- |
| src/ | 项目的源代码 |
| ~~index.js~~ | 应用程序的默认入口点 |
| test/ | 测试代码。通常用于放置针对`src/`的单元测试 |
| ~~index.test.js~~ | 应用程序的单元测试文件。例如) `index-test.js`、`indexSpec.js`等 |
| node_modules/ | 项目依赖的 npm 模块安装位置 |
| package.json | 项目的配置文件。包括名称、描述、脚本、依赖模块等 |
| package-lock.json | npm 处理的依赖模块锁定文件 |

**参考**

+   待办事项应用*************
