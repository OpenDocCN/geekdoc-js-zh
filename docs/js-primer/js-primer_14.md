# 函数和声明

> 原文：[`jsprimer.net/basic/function-declaration/`](https://jsprimer.net/basic/function-declaration/)

函数是将一系列操作（一组语句）作为一个处理单元组合在一起的功能。 通过使用函数，可以通过调用一次定义的函数来执行相同的操作，而不是每次都重复编写。

到目前为止，我们使用的控制台输出 Console API 也是一个函数。 `console.log`将“将接收到的值输出到控制台”这一过程封装为一个函数。

本章将讨论函数的定义和调用方式。

## [](#function-declaration)*函数声明*

*在 JavaScript 中，使用`function`关键字来定义函数。 以`function`开头的语句称为**函数声明**，可以定义如下函数。

```
// 関数宣言
function 関数名(仮引数 1, 仮引数 2) {
    // 関数が呼び出されたときの処理
    // ...
    return 関数の返り値;
}
// 関数呼び出し
const 関数の結果 = 関数名(引数 1, 引数 2);
console.log(関数の結果); // => 関数の返り値 
```

函数由以下 4 个元素组成。

+   函数名 - 可用的名称与变量名相同（参考“变量名规则”）

+   形式参数 - 传递给函数调用的值存储在的变量中。 如果有多个，则用`,`（逗号）分隔

+   函数体 - 写有`{`和`}`的函数处理的地方

+   函数的返回值 - 调用函数时返回给调用者的值

声明的函数可以通过在函数名后加括号来调用。 当调用函数时与参数一起调用时，应该使用`函数名(参数 1, 参数 2)`，如果有多个参数，则用`,`（逗号）分隔。

在函数体中，可以通过`return`语句返回任意值作为函数的执行结果。

在下面的代码中，定义了一个名为`double`的函数，该函数将接收到的值加倍返回。 `double`函数定义了一个名为`num`的形式参数，并将值`10`作为参数传递给函数调用。 参数`num`被赋值为`10`，然后将其加倍的值通过`return`语句返回。

```
function double(num) {
    return num * 2;
}
// `double`関数の返り値は、`num`に`10`を入れて`return`文で返した値
console.log(double(10)); // => 20 
```

当函数执行`return`语句时，函数内部将不再执行后续操作。 此外，如果函数不需要返回值，则可以省略`return`语句。 省略`return`语句时，将返回未定义的值`undefined`。

```
function fn() {
    // 何も返り値を指定してない場合は`undefined`を返す
    return;
    // すでにreturnされているため、この行は実行されません
}
console.log(fn()); // => undefined 
```

如果函数不需要返回任何值，则可以省略`return`语句。 省略`return`语句时，将返回值`undefined`。

```
function fn() {
}

console.log(fn()); // => undefined 
```

## [](#function-arguments)*函数的参数*

*在 JavaScript 中，即使定义函数时的形式参数数量与实际调用时的参数数量不同，也可以调用函数。 因此，需要了解参数数量不匹配时的行为。 此外，还将看到关于为省略参数指定默认值的默认参数语法。

### [](#function-less-arguments)*呼叫时的参数过少*

*如果调用定义函数时的形式参数比实际调用时的参数少，则多余的形式参数将被赋值为`undefined`。

在下面的代码中，定义了一个简单地返回传递值的`echo`函数。 `echo`函数定义了形式参数`x`，但如果在调用时没有传递参数，则形式参数`x`将包含`undefined`。

```
function echo(x) {
    return x;
}

console.log(echo(1)); // => 1
console.log(echo()); // => undefined 
```

即使是接受多个参数的函数，多余的形式参数也将包含`undefined`。

在下面的代码中，定义了一个接收两个参数并将它们作为数组返回的`argumentsToArray`函数。 在这种情况下，如果只传递一个值作为参数，则剩余的形式参数将被赋值为`undefined`。

```
function argumentsToArray(x, y) {
    return [x, y];
}

console.log(argumentsToArray(1, 2)); // => [1, 2]
// 仮引数のxには1、yにはundefinedが入る
console.log(argumentsToArray(1)); // => [1, undefined] 
```

### [](#function-default-parameters)*[ES2015] 默认参数*

*默认参数允许在未传递与形式参数对应的参数时指定默认分配的值。 可以通过为每个形式参数指定默认值的语法`形式参数 = 默认值`来指定默认值。

```
function 関数名(仮引数 1 = デフォルト値 1, 仮引数 2 = デフォルト値 2) {

} 
```

在下面的代码中，定义了一个简单地返回传递值的`echo`函数。 与之前的`echo`函数不同，这里为形式参数`x`指定了默认值。 因此，如果调用`echo`函数��没有传递参数，则形式参数`x`将被赋值为`"默认值"`。

```
function echo(x = "デフォルト値") {
    return x;
}

console.log(echo(1)); // => 1
console.log(echo()); // => "デフォルト値" 
```

在 ES2015 之前，经常使用 OR 运算符（`||`）来指定默认值。

```
function addPrefix(text, prefix) {
    const pre = prefix || "デフォルト:";
    return pre + text;
}

console.log(addPrefix("文字列")); // => "デフォルト:文字列"
console.log(addPrefix("文字列", "カスタム:")); // => "カスタム:文字列" 
```

然而，使用 OR 运算符（`||`）指定默认值存在一个问题。 使用 OR 运算符（`||`）时，如果左操作数为 falsy 值，则将评估右操作数。 falsy 值是指在转换为布尔值后为`false`的值。（参考“隐式类型转换”章节）。

+   `false`

+   `undefined`

+   `null`

+   `0`

+   `0n`

+   `NaN`

+   `""`（空字符串）

使用 OR 运算符（`||`）时，即使将`prefix`指定为空字符串，也会使用默认值。 这种行为非常难以理解，可能导致错误。

```
function addPrefix(text, prefix) {
    const pre = prefix || "デフォルト:";
    return pre + text;
}

// falsyな値を渡すとデフォルト値が入ってしまう
console.log(addPrefix("文字列")); // => "デフォルト:文字列"
console.log(addPrefix("文字列", "")); // => "デフォルト:文字列"
console.log(addPrefix("文字列", "カスタム:")); // => "カスタム:文字列" 
```

使用默认参数编写可以避免这种行为，因此更加安全。 默认参数将在未传递参数时使用默认值。

```
function addPrefix(text, prefix = "デフォルト:") {
    return prefix + text;
}
// falsyな値を渡してもデフォルト値は代入されない
console.log(addPrefix("文字列")); // => "デフォルト:文字列"
console.log(addPrefix("文字列", "")); // => "文字列"
console.log(addPrefix("文字列", "カスタム:")); // => "カスタム:文字列" 
```

此外，从 ES2020 开始，还可以使用 Nullish coalescing 运算符(`??`)来避免 OR 运算符（`||`）的问题并指定默认值。

```
function addPrefix(text, prefix) {
    // prefixがnullまたはundefinedの時、デフォルト値を返す
    const pre = prefix ?? "デフォルト:";
    return pre + text;
}

console.log(addPrefix("文字列")); // => "デフォルト:文字列"
// falsyな値でも意図通りに動作する
console.log(addPrefix("文字列", "")); // => "文字列"
console.log(addPrefix("文字列", "カスタム:")); // => "カスタム:文字列" 
```

### [](#function-more-arguments)*呼叫时的参数过多*

*如果函数的形式参数比调用时的参数多，则多余的参数将被简单地忽略。

在下面的代码中，定义了一个接收两个参数并返回它��之和的`add`函数。 `add`函数只有两个形式参数。 因此，即使传递了 3 个或更多的参数，第三个参数及以后的参数都将被简单地忽略。

```
function add(x, y) {
    return x + y;
}
add(1, 3); // => 4
add(1, 3, 5); // => 4 
```

## [](#variable-arguments)*可变参数*

*在函数中，参数的数量可能不固定，可能希望接受任意数量的参数。 例如，`Math.max(...args)`接受任意数量的参数，并返回接收到的参数中的最大数字的函数。 这种接受任意数量而不是固定数量的参数称为**可变参数**。

```
// Math.maxは可変長引数を受け取る関数
const max = Math.max(1, 5, 10, 20);
console.log(max); // => 20 
```

为了实现可变参数，可以使用特殊变量`arguments`，该变量仅在函数内部可引用。

### [](#rest-parameters)*[ES2015] Rest parameters*

*Rest parameters*是指在虚拟参数名之前加上`...`的虚拟参数，也称为剩余参数。 Rest parameters 将传递给函数的值分配为数组。

在下面的代码中，`fn`函数定义了`...args`作为 Rest parameters。 当调用`fn`函数时，参数的值将被分配给`args`作为数组。

```
function fn(...args) {
    // argsは、渡された引数が入った配列
    console.log(args); // => ["a", "b", "c"]
}
fn("a", "b", "c"); 
```

Rest parameters 可以与常规参数一起定义。 在与其他参数组合时，Rest parameters 必须作为最后一个参数进行定义。

在下面的代码中，第一个参数被分配给`arg1`，其余的参数被分配给`restArgs`作为数组。

```
function fn(arg1, ...restArgs) {
    console.log(arg1); // => "a"
    console.log(restArgs); // => ["b", "c"]
}
fn("a", "b", "c"); 
```

Rest parameters 是指将参数收集到数组中的语法。

Spread 语法是指在数组前面加上`...`的语法，将数组的值展开作为函数的参数传递。 在下面的代码中，我们展开了`array`数组，并将其作为参数传递给`fn`函数。

```
function fn(x, y, z) {
    console.log(x); // => 1
    console.log(y); // => 2
    console.log(z); // => 3
}
const array = [1, 2, 3];
// Spread 構文で配列を引数に展開して関数を呼び出す
fn(...array);
// 次のように書いたのと同じ意味
fn(array[0], array[1], array[2]); 
```

### [](#arguments)*`arguments`*

*处理可变参数的方法之一是使用函数内部可以引用的特殊变量`arguments`。 `arguments`是包含传递给函数的参数值的类似数组的对象。 类似数组的对象可以通过索引访问其元素，但不是`Array`，因此是一种特殊的对象，无法使用`Array`方法。

在下面的代码中，`fn`函数没有定义参数。 但是，在函数内部，可以使用名为`arguments`的变量引用传递的参数，就像数组一样。

```
function fn() {
    // `arguments`はインデックスを指定して各要素にアクセスできる
    console.log(arguments[0]); // => "a"
    console.log(arguments[1]); // => "b"
    console.log(arguments[2]); // => "c"
}
fn("a", "b", "c"); 
```

在使用 Rest parameters 的环境中，没有必要使用`arguments`变量。 `arguments`变量存在以下问题。

+   箭头函数中无法使用（有关箭头函数的详细信息，请参见下文）

+   由于是类似数组的对象，因此无法使用 Array 的方法

+   无法仅通过参数来判断函数是否接受可变长度参数

`arguments`变量与参数定义无关，而是包含所有传递的参数。 因此，在仅查看函数参数定义部分时，很难知道实际函数需要的参数。 Rest parameters 则可以清楚地表示是否接受可变长度参数。

因此，如果需要可变长度参数，请使用 Rest parameters 而不是`arguments`变量进行实现。

## [](#function-destructuring)*[ES2015] 函数的参数和解构*

*在函数参数中，也可以使用解构赋值（Destructuring assignment）。 解构赋值是一种语法，用于从对象或数组中获取属性，并重新定义为变量。*

次のコードでは、関数の引数として`user`オブジェクトを渡し、`id`プロパティをコンソールへ出力しています。

```
function printUserId(user) {
    console.log(user.id); // => 42
}
const user = {
    id: 42
};
printUserId(user); 
```

通过函数参数使用解构赋值，可以将此代码写为下面的样子。 下面的代码中，`printUserId`函数接收一个对象作为参数。 我们将传入的`user`对象的`id`属性定义为变量`id`。

```
// 第 1 引数のオブジェクトから`id`プロパティを変数`id`として定義する
function printUserId({ id }) {
    console.log(id); // => 42
}
const user = {
    id: 42
};
printUserId(user); 
```

在对象分解赋值的赋值运算符（`=`）中，左侧定义要定义的变量，右侧从对象中分配相应的属性。 将函数参数视为左侧，要传递给函数的参数视为右侧，就可以看到它们的语法几乎相同。

```
const user = {
    id: 42
};
// オブジェクトの分割代入
const { id } = user;
console.log(id); // => 42
// 関数の引数の分割代入
function printUserId({ id }) {
    console.log(id); // => 42
}
printUserId(user); 
```

函数参数中的解构赋值也适用于数组。 在下面的代码中，传递给参数的数组的第一个元素将分配给`first`，第二个元素将分配给`second`。

```
function print([first, second]) {
    console.log(first); // => 1
    console.log(second); // => 2
}
const array = [1, 2];
print(array); 
```

## [](#first-class-function)*函数是对象*

*在 JavaScript 中，函数被称为函数对象，是一种对象。 函数不同于普通对象，因为您可以通过在函数名称后加上`()`来调用函数并调用封装的处理。*

另一方面，如果没有附加`()`调用，则可以引用函数作为对象。 此外，函数可以与其他值一样分配给变量或传递给函数作为参数。

在下面的代码中，我们先将定义的`fn`函数分配给变量`myFunc`，然后调用它。

```
function fn() {
    console.log("fnが呼び出されました");
}
// 関数`fn`を`myFunc`変数に代入している
const myFunc = fn;
myFunc(); 
```

函数可以作为值进行操作，这称为**第一类函数**。

在上述代码中，我们先声明了函数，然后将其分配给变量。 但是，您也可以直接定义函数作为值。 当定义函数作为值时，可以使用与函数声明相同的`function`关键字方法和箭头函数方法。 这两种方法都被称为**函数表达式**，因为它们将函数视为表达式（分配给的值）。

### [](#function-expression)*函数表达式*

*函数表达式是指将函数作为值分配给变量的表达式。 函数声明是语句，而函数表达式则将函数视为值。 这与字符串或数字等变量声明的方式相同。*

```
// 関数式
const 変数名 = function() {
    // 関数を呼び出したときの処理
    // ...
    return 関数の返り値;
}; 
```

在函数表达式中，右侧的函数名可以省略`function`关键字。 这是因为定义的函数表达式可以通过变量名引用。 另一方面，在函数声明中，无法省略右侧的函数名。

```
// 関数式は変数名で参照できるため、"関数名"を省略できる
const 変数名 = function() {
};
// 関数宣言では"関数名"は省略できない
function 関数名() {
} 
```

在函数表达式中，您可以将无名称的函数分配给变量。 此类无名称函数称为**匿名函数**（或者无名函数）。

当然，您也可以为函数表达式命名。 但是，无法从外部调用该函数。 由于可以从函数内部调用，因此用于递归调用函数等。

```
// factorialは関数の外から呼び出せる名前
// innerFactは関数の外から呼び出せない名前
const factorial = function innerFact(n) {
    if (n === 0) {
        return 1;
    }
    // innerFactを再帰的に呼び出している
    return n * innerFact(n - 1);
};
console.log(factorial(3)); // => 6 
```

### [](#arrow-function)*[ES2015] Arrow Function*

*関数式には`function`キーワードを使った方法以外に、Arrow Functionと呼ばれる書き方があります。 名前のとおり矢印のような`=>`（イコールと大なり記号）を使い、無名関数を定義する構文です。 次のように、`function`キーワードを使った関数式とよく似た書き方をします。

```
// Arrow Functionを使った関数定義
const 変数名 = () => {
    // 関数を呼び出したときの処理
    // ...
    return 関数の返す値;
}; 
```

Arrow Functionには書き方にいくつかのパターンがありますが、`function`キーワードに比べて短く書けるようになっています。 また、Arrow Functionには省略記法があり、次の場合にはさらに短く書けます。

+   関数の仮引数が1つのときは`()`を省略できる

+   関数の処理が1つの式である場合に、ブロックと`return`文を省略できる

    +   その式の評価結果を`return`の返り値とする

```
// 仮引数の数と定義
const fnA = () => { /* 仮引数がないとき */ };
const fnB = (x) => { /* 仮引数が1つのみのとき */ };
const fnC = x => { /* 仮引数が1つのみのときは()を省略可能 */ };
const fnD = (x, y) => { /* 仮引数が複数のとき */ };
// 値の返し方
// 次の２つの定義は同じ意味となる
const mulA = x => { return x * x; }; // ブロックの中でreturn
const mulB = x => x * x;            // 1 行のみの場合はreturnとブロックを省略できる 
```

Arrow Functionについては次のような特徴があります。

+   名前をつけることができない（常に無名関数）

+   `this`が静的に決定できる（詳細は「関数とスコープ」の章で解説します）

+   `function`キーワードに比べて短く書くことができる

+   `new`できない（コンストラクタ関数ではない）

+   `arguments`変数を参照できない

たとえば`function`キーワードの関数式では、値を返すコールバック関数を次のように書きます。 配列の`map`メソッド��、配列の要素を順番にコールバック関数へ渡し、そのコールバック関数が返した値を新しい配列にして返します。

```
const array = [1, 2, 3];
// 1,2,3と順番に値が渡されコールバック関数（無名関数）が処理する
const doubleArray = array.map(function(value) {
    return value * 2; // 返した値をまとめた配列ができる
});
console.log(doubleArray); // => [2, 4, 6] 
```

Arrow Functionでは処理が1つの式だけである場合に、`return`文を省略し暗黙的にその式の評価結果を`return`の返り値とします。 また、Arrow Functionは仮引数が1つである場合は`()`を省略できます。 このような省略はコールバック関数を多用する場合にコードの見通しを良くします。

次のコードは、先ほどの`function`キーワードで書いたコールバック関数と同じ結果になります。

```
const array = [1, 2, 3];
// 仮引数が1つなので`()`を省略できる
// 関数の処理が1つの式なので`return`文を省略できる
const doubleArray = array.map(value => value * 2);
console.log(doubleArray); // => [2, 4, 6] 
```

Arrow Functionは`function`キーワードの関数式に比べて、できることとできないことがはっきりしています。 たとえば、`function`キーワードでは非推奨としていた`arguments`変数を参照できますが、Arrow Functionでは参照できなくなっています。 Arrow Functionでは、人による解釈や実装の違いが生まれにくくなります。

また、`function`キーワードとArrow Functionの大きな違いとして、`this`という特殊なキーワードに関する挙動の違いがあります。 `this`については「関数とスコープ」の章で解説しますが、Arrow Functionではこの`this`の問題の多くを解決できるという利点があります。

そのため、Arrow Functionで問題ない場合はArrow Functionで書き、そうでない場合は`function`キーワードを使うことを推奨します。

## [](#function-overwrite)*[コラム] 同じ名前の関数宣言は上書きされる*

*関数宣言で定義した関数は、関数の名前でのみ区別されます。 そのため、同じ名前の関数を複数回宣言した場合には、後ろで宣言された関数によって上書きされます。

次のコードでは、`fn`という関数名を2つ定義していますが、最後に定義された`fn`関数が優先されています。 また、仮引数の定義が異なっていても、関数の名前が同じなら上書きされます。

```
function fn(x) {
    return `最初の関数 x: ${x}`;
}
function fn(x, y) {
    return `最後の関数 x: ${x}, y: ${y}`;
}
console.log(fn(2, 10)); // => "最後の関数 x: 2, y: 10" 
```

このように、同じ関数名で複数の関数を定義することは、関数を上書きしてしまうため避けるべきです。 引数の違いで関数を呼び分けたい場合は、別々の名前で関数を定義するか関数の内部で引数の値で処理を分岐する必要があります。

この関数定義の上書きは`function`キーワードでの関数宣言と`var`キーワードを使った関数式のみで発生します。 一方で、`const`や`let`では同じ変数名の定義はエラーとなるため、このような関数定義の上書きもエラーとなります。

```
const fn = (x) => {
    return `最初の関数 x: ${x}`;
};
// constは同じ変数名を定義できないため、構文エラーとなる
const fn = (x, y) => {
    return `最後の関数 x: ${x}, y: ${y}`;
}; 
```

関数の上書きを避けたい場合は、`const`と関数式を使って関数を定義することで、意図しない上書きが発生しにくくなります。

## [](#callback)*コールバック関数*

*関数はファーストクラスであるため、その場で作った無名関数を関数の引数（値）として渡すことができます。 引数として渡される関数のことを**コールバック関数**と呼びます。 一方、コールバック関数を引数として使う関数やメソッドのことを**高階関数**と呼びます。

```
function 高階関数(コールバック関数) {
    コールバック関数();
} 
```

たとえば、配列の`forEach`メソッドはコールバック関数を引数として受け取る高階関数です。 `forEach`メソッドは、配列の各要素に対してコールバック関数を一度ずつ呼び出します。

```
const array = [1, 2, 3];
const output = (value) => {
    console.log(value);
};
array.forEach(output);
// 次のように実行しているのと同じ
// output(1); => 1
// output(2); => 2
// output(3); => 3 
```

毎回、関数を定義してその関数をコールバック関数として渡すのは、少し手間がかかります。 そこで、関数はファーストクラスであることを利用して、コールバック関数となる無名関数をその場で定義して渡せます。

```
const array = [1, 2, 3];
array.forEach((value) => {
    console.log(value);
}); 
```

コールバック関数は非同期処理においてもよく利用されます。 非同期処理におけるコールバック関数の利用方法については「非同期処理」の章で解説します。

## [](#method)*メソッド*

*オブジェクトのプロパティである関数を**メソッド**と呼びます。 JavaScriptにおいて、関数とメソッドの機能的な違いはありません。 しかし、呼び方を区別したほうがわかりやすいため、ここではオブジェクトのプロパティである関数をメソッドと呼びます。

次のコードでは、`obj`の`method1`プロパティと`method2`プロパティに関数を定義しています。 この`obj.method1`プロパティと`obj.method2`プロパティがメソッドです。

```
const obj = {
    method1: function() {
        // `function`キーワードでのメソッド
    },
    method2: () => {
        // Arrow Functionでのメソッド
    }
}; 
```

次のように空オブジェクトの`obj`を定義してから、`method`プロパティへ関数を代入してもメソッドを定義できます。

```
const obj = {};
obj.method = function() {
}; 
```

メソッドを呼び出す場合は、関数呼び出しと同様に`オブジェクト.メソッド名()`と書くことで呼び出せます。

```
const obj = {
    method: function() {
        return "this is method";
    }
};
console.log(obj.method()); // => "this is method" 
```

### [](#shorthand-for-method)*[ES2015] メソッドの短縮記法*

*先ほどの方法では、プロパティに関数を代入するという書き方になっていました。 ES2015からは、メソッドとしてプロパティを定義するための短縮した書き方が追加されています。

次のように、オブジェクトリテラルの中で `メソッド名(){ /*メソッドの処理*/ }` と書くことができます。

```
const obj = {
    method() {
        return "this is method";
    }
};
console.log(obj.method()); // => "this is method" 
```

この書き方はオブジェクトのメソッドだけではなく、クラスのメソッドと共通の書き方となっています。 メソッドを定義する場合は、できるだけこの短縮記法に統一したほうがよいでしょう。

## [](#function-declaration-summary)*まとめ*

*この章では、次のことについて学びました。

+   関数の宣言方法

+   関数を値として使う方法

+   関数式とArrow Function

+   コールバック関数

+   メソッドの定義方法

我们学习了基本函数的定义和函数作为值。在 JavaScript 中，经常处理异步处理，在这种情况下会使用回调函数。通过使用箭头函数，可以将回调函数编写得更加简洁。

JavaScript 中的方法是指对象属性中的函数。从 ES2015 开始，已经添加了定义方法的语法，因此我们将利用它。
