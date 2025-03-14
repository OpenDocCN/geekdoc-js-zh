# 第四章：函数

函数是 JavaScript 编程中最核心的工具之一。将一段程序包装在一个值中的概念有很多用途。它为我们提供了一种结构化大型程序的方法，减少重复，将名称与子程序关联，以及将这些子程序相互隔离。

函数最明显的用途是定义新词汇。在散文中，创造新词通常被视为不良风格，但在编程中这是不可或缺的。

典型的成人英语使用者的词汇量大约有 20,000 个单词。很少有编程语言内置 20,000 个命令。而可用的词汇往往被定义得更为精确，因此在灵活性上不如人类语言。因此，我们`必须`引入新词，以避免过于冗长。

### 定义函数

函数定义是一种常规绑定，其中绑定的值是一个函数。例如，以下代码将`square`定义为指向一个生成给定数字平方的函数：

```js
const square = function(x) {
  return x * x;
};
console.log(square(12));
// → 144
```

函数是通过以关键字`function`开头的表达式创建的。函数有一组`参数`（在此例中，仅为`x`）和一个`主体`，主体包含在调用函数时要执行的语句。以这种方式创建的函数主体必须始终用大括号包裹，即使它仅由一个语句组成。

函数可以有多个参数或根本没有参数。在以下示例中，`makeNoise`没有列出任何参数名，而`roundTo`（将`n`四舍五入到`step`的最近倍数）列出了两个：

```js
const makeNoise = function() {
  console.log("Pling!");
};

makeNoise();
// → Pling!

const roundTo = function(n, step) {
  let remainder = n % step;
  return n - remainder + (remainder < step / 2 ? 0 : step);
};

console.log(roundTo(23, 10));
// → 20
```

一些函数，比如`roundTo`和`square`，会产生一个值，而有些则不会，比如`makeNoise`，它唯一的结果是一个副作用。返回语句决定了函数返回的值。当控制流遇到这样的语句时，它会立即跳出当前函数，并将返回的值传递给调用该函数的代码。如果`return`关键字后没有表达式，函数将返回`undefined`。没有返回语句的函数，例如`makeNoise`，也同样返回`undefined`。

函数的参数表现得像常规绑定，但它们的初始值由函数的`调用者`提供，而不是函数内部的代码。

### 绑定与作用域

每个绑定都有一个`作用域`，即绑定可见的程序部分。对于在任何函数、块或模块外部定义的绑定（见第十章），作用域是整个程序——你可以在任何地方引用这些绑定。这些被称为`全局`绑定。

为函数参数创建的绑定或在函数内部声明的绑定只能在该函数内引用，因此它们被称为`局部`绑定。每次调用函数时，这些绑定的新实例都会被创建。这在函数之间提供了一些隔离——每个函数调用都在自己的小世界中运行（其局部环境），并且通常可以在不需要了解全局环境中发生的事情的情况下理解。

使用`let`和`const`声明的绑定实际上是局部于其声明所在的`块`，因此如果在循环内部创建了其中一个，循环前后的代码无法“看到”它

```js
let x = 10;   // Global
if (true) {
  let y = 20; // Local to block
  var z = 30; // Also global
}
```

每个作用域可以“向外”查看周围的作用域，因此在示例中的块内部可以看到`x`。例外情况是当多个绑定具有相同名称时——在这种情况下，代码只能看到最内层的绑定。例如，当`halve`函数内部的代码引用`n`时，它看到的是它`自己的` `n`，而不是全局的`n`。

```js
const halve = function(n) {
  return n / 2;
};

let n = 10;
console.log(halve(100));
// → 50
console.log(n);
// → 10
```

### 嵌套作用域

JavaScript不仅区分全局绑定和局部绑定。块和函数可以在其他块和函数内部创建，从而产生多重局部性。

例如，这个函数——输出制作一批鹰嘴豆泥所需的成分——内部还有另一个函数：

```js
const hummus = function(factor) {
  const ingredient = function(amount, unit, name) {
    let ingredientAmount = amount * factor;
    if (ingredientAmount > 1) {
      unit += "s";
 }
    console.log(`${ingredientAmount} ${unit} ${name}`);
  };
  ingredient(1, "can", "chickpeas");
  ingredient(0.25, "cup", "tahini");
  ingredient(0.25, "cup", "lemon juice");
  ingredient(1, "clove", "garlic");
  ingredient(2, "tablespoon", "olive oil");
  ingredient(0.5, "teaspoon", "cumin");
};
```

成分函数内部的代码可以看到外部函数的`factor`绑定。但它的局部绑定，例如`unit`或`ingredientAmount`，在外部函数中不可见。

块内可见的绑定集合由该块在程序文本中的位置决定。每个局部作用域也可以看到包含它的所有局部作用域，所有作用域都可以看到全局作用域。这种绑定可见性的处理方式称为`词法作用域`。

### 函数作为值

函数绑定通常只是程序中某个特定部分的名称。这种绑定定义一次，永不改变。这使得函数和其名称之间容易混淆。

但二者是不同的。函数值可以执行其他值能够做的所有操作——你可以在任意表达式中使用它，而不仅仅是调用它。可以将函数值存储在新的绑定中，作为参数传递给一个函数，等等。类似地，持有函数的绑定仍然只是一个常规绑定，如果不是常量，它可以被分配一个新值，如下所示：

```js
let launchMissiles = function() {
  missileSystem.launch("now");
};
if (safeMode) {
  launchMissiles = function() {/* do nothing */};
}
```

在第五章中，我们将讨论通过将函数值传递给其他函数可以做的有趣事情。

### 声明符号

创建函数绑定有一种稍短的方式。当在语句开始时使用`function`关键字时，它的工作方式是不同的。

```js
function square(x) {
  return x * x;
}
```

这是一个函数`声明`。该语句定义了绑定`square`，并指向给定的函数。它稍微容易写一点，并且在函数后不需要分号。

这种形式的函数定义有一个细微之处。

```js
console.log("The future says:", future());

function future() {
  return "You'll never have flying cars";
}
```

前面的代码可以正常工作，即使函数定义在使用它的代码`下面`。函数声明并不是常规自上而下控制流的一部分。它们在概念上被移到作用域的顶部，可以被该作用域内的所有代码使用。这在某些情况下很有用，因为它提供了按最清晰的方式排列代码的自由，而不必担心在使用之前定义所有函数。

### 箭头函数

函数还有第三种表示法，它看起来与其他两种非常不同。它使用一个箭头（`=>`），由一个等号和一个大于号字符组成（不要与大于或等于运算符混淆，该运算符写作`>=`）。

```js
const roundTo = (n, step) => {
  let remainder = n % step;
  return n - remainder + (remainder < step / 2 ? 0 : step);
};
```

箭头位于参数列表`之后`，并后接函数的主体。它表达了类似于“这个输入（参数）产生这个结果（主体）”的意思。

当只有一个参数名称时，可以省略参数列表周围的括号。如果主体是单个表达式而不是用大括号括起来的代码块，那么该表达式将从函数中返回。这意味着这两种对`square`的定义做的是同样的事情：

```js
const square1 = (x) => { return x * x; };
const square2 = x => x * x;
```

当箭头函数没有任何参数时，它的参数列表只是一个空的括号。

```js
const horn = () => {
  console.log("Toot");
};
```

语言中同时存在箭头函数和函数表达式没有深层原因。除了一个我们将在第六章中讨论的小细节，它们做的是同样的事情。箭头函数是在 2015 年添加的，主要是为了以较少的冗长方式编写小的函数表达式。我们将在第五章中经常使用它们。

### 调用栈

控制在函数中的流动方式有些复杂。让我们仔细看看。以下是一个简单的程序，进行了一些函数调用：

```js
function greet(who) {
  console.log("Hello " + who);
}
greet("Harry");
console.log("Bye");
```

运行这个程序大致是这样的：对`greet`的调用使控制跳到该函数的开始（第 2 行）。该函数调用`console.log`，`console.log`接管控制，完成它的工作，然后将控制返回到第 2 行。在那里，它到达`greet`函数的末尾，因此返回到调用它的地方——第 4 行。之后的行再次调用`console.log`。在返回后，程序到达结束。

我们可以将控制流示意性地展示如下：

```js
not in function
  in greet
    in console.log
  in greet
not in function
  in console.log
not in function
```

因为函数在返回时必须跳回调用它的地方，所以计算机必须记住调用发生的上下文。在一种情况下，`console.log`在完成时必须返回到`greet`函数。在另一种情况下，它返回到程序的末尾。

计算机存储这个上下文的地方是`调用栈`。每次调用函数时，当前上下文会被存储在这个栈的顶部。当函数返回时，它从栈中移除顶部的上下文，并使用该上下文继续执行。

存储这个栈需要计算机内存中的空间。当栈增长得太大时，计算机会出现“栈空间不足”或“递归过多”等错误消息。以下代码通过向计算机提出一个非常棘手的问题来说明这一点，这会导致两个函数之间发生无限的往返。或者说，如果计算机有无限的栈，这将是无限的。实际上，我们会耗尽空间，或者“使栈溢出”。

```js
function chicken() {
  return egg();
}
function egg() {
  return chicken();
}
console.log(chicken() + " came first.");
// → ??
```

### 可选参数

以下代码是允许的，并且没有任何问题地执行：

```js
function square(x) { return x * x; }
console.log(square(4, true, "hedgehog"));
// → 16
```

我们仅用一个参数定义了`平方`。然而，当我们用三个参数调用它时，语言并不会抱怨。它会忽略额外的参数，计算第一个参数的平方。

`JavaScript`对于你可以传递给函数的参数数量极为宽松。如果你传递太多，额外的参数会被忽略。如果你传递的参数太少，缺失的参数会被赋值为`undefined`。

这样做的缺点是，可能——甚至很可能——你会不小心向函数传递错误数量的参数。而且没有人会告诉你。好处是，你可以利用这种行为，使得一个函数可以接受不同数量的参数。例如，这个`minus`函数尝试模仿`-`运算符，可以处理一个或两个参数：

```js
function minus(a, b) {
  if (b === undefined) return -a;
  else return a - b;
}

console.log(minus(10));
// → -10
console.log(minus(10, 5));
// → 5
```

如果在参数后写一个`=`运算符，后面跟一个表达式，当未提供参数时，该表达式的值将替代参数。例如，这个版本的`roundTo`使得它的第二个参数变为可选。如果你不提供它或传递值`undefined`，它将默认为`1`。

```js
function roundTo(n, step = 1) {
  let remainder = n % step;
  return n - remainder + (remainder < step / 2 ? 0 : step);
};
console.log(roundTo(4.5));
// → 5
console.log(roundTo(4.5, 2));
// → 4
```

下一章将介绍一种方法，通过这种方法，函数体可以访问它所接收到的所有参数列表。这是非常有帮助的，因为它允许函数接受任意数量的参数。例如，`console.log`就是这样做的，输出它所接收到的所有值。

```js
console.log("C", "O", 2);
// → C O 2
```

### `闭包`

将函数视为值的能力，加上每次调用函数时局部绑定会被重新创建的事实，提出了一个有趣的问题：当创建它们的函数调用不再活跃时，局部绑定会发生什么？

以下代码展示了这个例子。它定义了一个函数`wrapValue`，创建了一个局部绑定。然后返回一个可以访问并返回这个局部绑定的函数。

```js
function wrapValue(n) {
  let local = n;
  return () => local;
}

let wrap1 = wrapValue(1);
let wrap2 = wrapValue(2);
console.log(wrap1());
// → 1
console.log(wrap2());
// → 2
```

这是允许的，并且按你所希望的那样工作——两个绑定实例仍然可以被访问。这种情况很好地展示了局部绑定是为每次调用重新创建的，不同的调用不会互相影响各自的局部绑定。

这一特性——能够引用包围作用域中特定实例的局部绑定——被称为`闭包`。引用周围局部作用域中绑定的函数被称为`闭包`。这种行为不仅让你不必担心绑定的生命周期，而且使得以某种创造性的方式使用函数值成为可能。

通过稍微修改，我们可以将之前的示例转变为一种创建可以乘以任意数值的函数的方法。

```js
function multiplier(factor) {
  return number => number * factor;
}

let twice = multiplier(2);
console.log(twice(5));
// → 10
```

在 `wrapValue` 示例中的显式局部绑定其实并不是必需的，因为参数本身就是一个局部绑定。

以这种方式思考程序需要一些练习。一个好的心理模型是将函数值视为包含其主体中的代码以及创建它时的环境。当被调用时，函数主体看到的是它创建时的环境，而不是它被调用时的环境。

在这个示例中，`multiplier`被调用并创建了一个环境，其中其因子参数绑定为`2`。它返回的函数值被存储在`twice`中，记住了这个环境，以便在调用时将其参数乘以`2`。

### `递归`

函数

```js
function power(base, exponent) {
  if (exponent == 0) {
    return 1;
  } else {
    return base * power(base, exponent - 1);
  }
}

console.log(power(2, 3));
// → 8
```

这与数学家对指数运算的定义非常接近，并且可以说比我们在第二章中使用的循环更清晰地描述了这一概念。该函数多次调用自身，指数越来越小，以实现重复乘法。

然而，这种实现存在一个问题：在典型的 `JavaScript` 实现中，它的速度大约是使用 `for` 循环版本的三倍慢。运行一个简单的循环通常比多次调用函数更便宜。

关于速度与优雅之间的困境是一个有趣的话题。你可以将其视为人性化和机器友好之间的一种连续体。几乎任何程序都可以通过变得更大、更复杂来加快速度。程序员需要找到一个合适的平衡。

对于`幂函数`而言，一个不优雅的（循环）版本仍然相当简单且易于阅读。用递归函数替代它并没有太大意义。然而，通常情况下，程序处理的概念如此复杂，以至于为了使程序更简洁而牺牲一些效率是有益的。

过于关注效率可能会分散注意力。这又是一个使程序设计复杂化的因素，当你正在做一些已经很困难的事情时，额外需要担心的事可能会让人感到无从下手。

因此，你通常应该从编写一些正确且易于理解的代码开始。如果你担心它太慢——而实际上它通常并不慢，因为大多数代码并不会频繁执行，无法占用显著的时间——你可以在之后进行测量，并在必要时进行改进。

递归并不总是仅仅是循环的一个低效替代方案。有些问题确实用递归比用循环更容易解决。通常这些是需要探索或处理多个“分支”的问题，每个分支可能又会进一步分叉。

考虑这个难题：通过从数字 `1` 开始，反复添加 `5` 或乘以 `3`，可以产生一个无限集合的数字。你会如何写一个函数，给定一个数字，尝试找到这样一系列的加法和乘法来生成那个数字？例如，数字 `13` 可以通过先乘以 `3` 然后加 `5` 两次来得到，而数字 `15` 则根本无法得到。

这里有一个递归解决方案：

```js
function findSolution(target) {
  function find(current, history) {
    if (current == target) {
      return history;
    } else if (current > target) {
      return null;
    } else {
      return find(current + 5, `(${history} + 5)`) ??
             find(current * 3, `(${history} * 3)`);
    }
  }
  return find(1, "1");
}

console.log(findSolution(24));
// → (((1 * 3) + 5) * 3)
```

请注意，这个程序并不一定找到操作的`最短`序列。它在找到任何序列时就会满足。

如果你立刻看不懂这段代码也没关系。我们一步一步来，因为这将是一个很好的递归思维练习。

内部函数 `find` 实际上执行递归。它接受两个参数：当前数字和一个记录我们如何到达这个数字的字符串。如果它找到了解决方案，它会返回一个显示如何达到目标的字符串。如果从这个数字出发找不到解决方案，它会返回`null`。

为此，该函数执行三种操作之一。如果当前数字是目标数字，那么当前历史就是达到该目标的方式，因此会返回。如果当前数字大于目标，继续探索这个分支就没有意义，因为加法和乘法只会使数字变大，因此返回`null`。最后，如果我们仍然低于目标数字，函数会通过调用自身两次尝试从当前数字开始的两个可能路径，一次用于加法，一次用于乘法。如果第一次调用返回的结果不是`null`，那么返回它。否则，返回第二次调用的结果，无论它是否生成字符串或`null`。

为了更好地理解这个函数是如何产生我们所期望的效果的，让我们查看在搜索数字 `13` 的解决方案时所进行的所有对 `find` 的调用。

```js
find(1, "1")
  find(6, "(1 + 5)")
    find(11, "((1 + 5) + 5)")
      find(16, "(((1 + 5) + 5) + 5)")
        too big
      find(33, "(((1 + 5) + 5) * 3)")
        too big
    find(18, "((1 + 5) * 3)")
      too big
  find(3, "(1 * 3)")
    find(8, "((1 * 3) + 5)")
      find(13, "(((1 * 3) + 5) + 5)")
        found!
```

缩进表示调用栈的深度。第一次调用 `find` 时，函数开始通过调用自身来探索以`(1 + 5)`开头的解决方案。该调用将进一步递归，以探索`每个`导致小于或等于目标数字的持续解决方案。由于没有找到一个正好命中的目标，因此返回`null`回到第一次调用。在那里，`??`运算符导致探索`(1 * 3)`的调用发生。这个搜索更幸运——它的第一个递归调用，通过又`一个`递归调用，找到了目标数字。最内层的调用返回一个字符串，而中间调用中的每个`??`运算符将该字符串传递下去，最终返回解决方案。

### `增长函数`

函数引入程序的方式有两种或多种自然的方法。

第一个情况发生在你发现自己多次编写类似代码时。你会更希望不这样做，因为代码越多，隐藏错误的空间就越大，尝试理解程序的人需要阅读的材料也就越多。因此，你提取出重复的功能，为其找到一个好名字，并将其放入一个函数中。

第二种方法是你发现需要一些你尚未编写的功能，这听起来应该有自己的函数。你首先给函数命名，然后编写它的主体。在实际定义函数之前，你甚至可能会开始编写使用该函数的代码。

找到一个好名字为函数命名的难易程度，很好地指示了你试图封装的概念有多清晰。让我们通过一个例子来说明。

我们想写一个程序，打印两个数字：农场上牛和鸡的数量，后面跟上“Cows”和“Chickens”这两个词，并在这两个数字前面填充零，使其始终为三位数。

```js
007 Cows
011 Chickens
```

这要求一个有两个参数的函数——牛的数量和鸡的数量。让我们开始编码吧。

```js
function printFarmInventory(cows, chickens) {
  let cowString = String(cows);
  while (cowString.length < 3) {
    cowString = "0" + cowString;
  }
  console.log(`${cowString} Cows`);
  let chickenString = String(chickens);
  while (chickenString.length < 3) {
    chickenString = "0" + chickenString;
  }
  console.log(`${chickenString} Chickens`);
}
printFarmInventory(7, 11);
```

在字符串表达式后面写`.length`会给我们该字符串的长度。因此，`while`循环不断在数字字符串前面添加零，直到它们至少有三个字符长。

任务完成！但就在我们准备将代码（以及一份高额账单）发送给农民时，她打电话告诉我们她也开始养猪了，能不能请我们扩展软件，以便也能打印猪的数量？

我们当然可以。但是就在我们准备再复制粘贴那四行代码时，我们停下来重新考虑。这一定有更好的方法。下面是第一次尝试：

```js
function printZeroPaddedWithLabel(number, label) {
  let numberString = String(number);
  while (numberString.length < 3) {
    numberString = "0" + numberString;
  }
  console.log(`${numberString} ${label}`);
}

function printFarmInventory(cows, chickens, pigs) {
  printZeroPaddedWithLabel(cows, "Cows");
  printZeroPaddedWithLabel(chickens, "Chickens");
  printZeroPaddedWithLabel(pigs, "Pigs");
}

printFarmInventory(7, 11, 3);
```

它成功了！但是这个名字`printZeroPaddedWithLabel`有点尴尬。它将打印、零填充和添加标签这三件事混为一谈，放入了一个函数中。

与其将程序中重复的部分整体提取出来，不如试着挑出一个单一的`概念`。

```js
function zeroPad(number, width) {
  let string = String(number);
  while (string.length < width) {
    string = "0" + string;
  }
  return string;
}

function printFarmInventory(cows, chickens, pigs) {
  console.log(`${zeroPad(cows, 3)} Cows`);
  console.log(`${zeroPad(chickens, 3)} Chickens`);
  console.log(`${zeroPad(pigs, 3)} Pigs`);
}

printFarmInventory(7, 16, 3);
```

一个名字清晰明显的函数，如`zeroPad`，让阅读代码的人更容易理解其作用。这样的函数在比这个特定程序更多的场景中也很有用。例如，你可以使用它来帮助打印对齐整齐的数字表。

我们的函数应该有多聪明和多功能？我们可以编写任何东西，从只能将数字填充为三个字符宽的简单函数，到一个复杂的通用数字格式化系统，能够处理分数、负数、对齐小数点、用不同字符填充等等。

一个有用的原则是，除非你绝对确定需要，否则不要添加聪明的功能。为你遇到的每一项功能编写通用的“框架”可能会让人心动，但要抵制这种冲动。你不会真正完成任何工作——你会忙于编写从未使用的代码。

### 函数与副作用

函数大致可以分为两类：一类是由于其副作用而被调用，另一类是由于其返回值而被调用（尽管也可能同时具有副作用和返回值）。

在农场示例中，第一个辅助函数`printZeroPaddedWithLabel`因其副作用而被调用：它打印一行。第二个版本`zeroPad`因其返回值而被调用。第二个函数在更多情况下有用并非偶然。创建值的函数比直接执行副作用的函数更容易以新的方式组合。

一个`纯`函数是一种特定类型的值生成函数，它不仅没有副作用，而且不依赖于其他代码的副作用——例如，它不读取可能会改变值的全局绑定。一个纯函数有一个愉快的特性，即在使用相同的参数调用时，它总是产生相同的值（而且不做其他事情）。对这样的函数的调用可以用它的返回值替代，而不改变代码的含义。当你不确定一个纯函数是否正确工作时，可以通过简单地调用它来测试，如果它在那个上下文中工作，那么在任何上下文中都将工作。非纯函数往往需要更多的支架来进行测试。

不必感到羞愧，当你编写不是纯函数的函数时。副作用往往是有用的。例如，`console.log`的纯版本是无法编写的，但`console.log`是非常有用的。在某些操作中，当我们使用副作用时，表达起来也更高效。

### 总结

本章教你如何编写自己的函数。当作为表达式使用时，`function`关键字可以创建一个函数值。当作为语句使用时，它可以用于声明一个绑定，并将函数作为其值。箭头函数是创建函数的另一种方式。

```js
// Define f to hold a function value
const f = function(a) {
  console.log(a + 2);
};
// Declare g to be a function
function g(a, b) {
  return a * b * 3.5;
}

// A less verbose function value
let h = a => a % 3;
```

理解函数的一个关键部分是理解作用域。每个代码块都会创建一个新的作用域。在给定作用域中声明的参数和绑定是局部的，外部不可见。用`var`声明的绑定行为有所不同——它们会进入最近的函数作用域或全局作用域。

将程序执行的任务分离成不同的函数是有帮助的。你将不必如此频繁地重复自己，函数可以通过将代码分组为执行特定任务的片段来帮助组织程序。

### 练习

#### `最小值`

前一章介绍了标准函数`Math.min`，它返回最小的参数。我们现在可以自己编写这样的函数。定义一个名为`min`的函数，它接受两个参数并返回它们的最小值。

#### `递归`

我们已经看到可以使用`%`（余数运算符）来测试一个数字是偶数还是奇数，通过使用`% 2`来查看它是否可以被二整除。这里还有另一种方法来定义一个正整数是偶数还是奇数：

+   零是偶数。

+   `一是奇数。`

+   对于任何其他数字 `N`，其偶性与 `N - 2`相同。

定义一个递归函数`isEven`，符合这个描述。该函数应该接受一个单一参数（一个正整数）并返回一个布尔值。

在`50`和`75`上测试一下。看看它在`-1`上的表现。为什么？你能想到修复这个问题的方法吗？

#### `豆子计数`

你可以通过在字符串后写`[N]`来获取字符串中的第`N`个字符或字母（例如，`string[2]`）。得到的值将是一个只包含一个字符的字符串（例如，“b”）。第一个字符的位置是`0`，这导致最后一个字符位于`string.length - 1`的位置。换句话说，一个两个字符的字符串长度为`2`，其字符的位置分别为`0`和`1`。

编写一个名为`countBs`的函数，它以字符串作为唯一参数并返回一个数字，表示字符串中大写字母`B`的数量。

接下来，编写一个名为`countChar`的函数，行为类似于`countBs`，但它接受一个第二个参数，表示要计数的字符（而不仅仅是计数大写字母`B`）。重写`countBs`以利用这个新函数。

`在两个场合，我被问到：“请问，巴贝奇先生，如果你输入错误的数字，机器会输出正确的答案吗？” [. . .] 我无法正确理解会引发这样问题的那种思想混乱。`

—查尔斯·巴贝奇，`哲学家生平的片段`（1864）

`![图片](img/f0056-01.jpg)`
