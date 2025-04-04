

## 第十二章：5 函数



![](img/opener.png)

正如你在第一章中学到的，*函数*是一个自包含的代码块，用于执行特定任务。我们已经使用了一些 JavaScript 的内置函数，例如 alert 和 console.log，但你也可以创建自己的自定义函数来执行应用程序中特定的任务。然后，你可以*调用*这些函数来运行关联的代码。以这种方式将代码封装成函数，可以使你的编程更加高效，因为你不必每次使用代码时都重复它。

在这一章中，你将学习编写自己函数的不同技巧。你将看到如何向函数提供输入并从中接收输出。你还将看到函数如何作为普通值处理，就像数字或字符串一样。特别是，我们将探索函数如何作为其他高阶函数的输入或输出。

### 声明和调用函数

在使用自定义函数之前，你必须先定义函数的名称和它的功能。一种方法是使用*函数声明*，这是一个定义函数的代码块。为了说明这一点，我们将声明一个简单的函数，名为 sayHello，它接受某人的名字并在控制台中记录一条个性化的问候信息。在 Chrome 中打开 JavaScript 控制台并输入以下内容：

```
**function sayHello(name) {**
 **console.log(`Hello, ${name}!`);**
**}** 
```

函数声明有四个部分。首先，我们使用 function 关键字来告诉 JavaScript 我们正在创建一个函数。接下来，我们给函数命名——在本例中为 sayHello。然后，我们提供一个由逗号分隔的函数参数列表，参数用括号括起来。*参数*是函数执行任务所需要的信息。在这个例子中，我们的函数有一个参数 name，表示该函数需要提供某人的名字来创建问候语。（如果函数没有参数，我们只需写一个空的括号。）最后，我们写出函数体，用大括号括起来。这是当调用函数时应该执行的代码。在我们的示例中，函数体包含一个 console.log 调用，用于打印问候语，并通过模板字符串插入 name 参数的值。

现在我们已经声明了 sayHello 函数，我们可以在任何时候调用它来问候某人。每次调用函数时，我们需要为 name 参数提供一个值。这个值叫做*参数*，并在调用函数时通过括号指定。通过传递不同的参数，我们可以创建不同的个性化问候语。例如：

```
**sayHello("Nick");**
Hello, Nick!
undefined
**sayHello("Mei");**
Hello, Mei!
undefined 
```

第一次调用我们的 sayHello 函数时，我们在函数名后面的括号中传入 "Nick" 作为参数。因此，消息 Hello, Nick! 被打印到控制台。第二次调用函数时，我们传入 "Mei" 作为参数，因此消息 Hello, Mei! 被打印。每次调用时，参数的值会绑定到函数的名称参数上，函数体根据该参数值执行。实际上，你可以将 name 理解为函数内部的一个变量，当函数被调用时，它会根据相应的参数（如 "Nick" 或 "Mei"）赋值。

参数和实参之间的区别微妙但重要。参数是函数输入的通用名称，而实参是在调用函数时传递给函数的实际输入值。每个函数只有一组参数，但每次调用函数时，它可以有一组新的实参。通过这种方式，参数使得函数具有高度的可定制性。例如，sayHello 函数有一个参数 name，但每次调用时都可以传入不同的实参。我们见过它被这样调用：sayHello("Nick") 和 sayHello("Mei")，但可能性是无穷的：sayHello("Kitty")、sayHello("Dolly")、sayHello("world") 等等。

注意，每次调用 sayHello 时，它会输出 undefined 以及自定义的问候语。这个额外的输出行就是函数的返回值。sayHello 返回 undefined，因为我们没有显式给它返回值；接下来我们将学习如何做到这一点。

#### 返回值

*返回值* 是一个函数产生的值，可以在代码的其他地方使用。在许多情况下，你希望一个函数使用参数接收一些输入，处理这些输入并输出结果。这个输出就是返回值。例如，假设我们声明一个函数，接收两个数字并返回它们的和：

```
**function add(x, y) {**
 **return x + y;**
**}** 
```

这个 add 函数有两个参数，x 和 y。函数体由 return 关键字和表达式 x + y 组成。当调用该函数时，JavaScript 会计算这个表达式，将 x 和 y 相加，并返回结果，如下所示：

```
**add(1, 2);**
3 
```

我们用 1 和 2 作为实参调用 add，它们分别成为参数 x 和 y 的值。（实参按给定的顺序与参数一一对应。）该函数将两个实参相加并返回结果 3。

当我们在 Chrome 控制台中调用一个函数时，它的返回值会自动打印出来——但需要注意区分函数显式将文本记录到控制台（就像我们之前看到的 sayHello 做的那样）和函数返回值（就像这里的 add）。当函数使用 console.log 记录一个值时，这个值只存在于日志中；我们无法在后续代码中进一步使用它。相反，当一个函数返回一个值时，我们可以在代码中稍后使用该值。尽管返回值也会在控制台中显示，但这基本上是无关紧要的。它帮助我们看到函数的行为，但记录到控制台并不是 add 函数的主要目的，而 sayHello 函数则不同。

使用函数的返回值的一种方式是将函数调用作为赋值表达式的一部分，这样返回值就会被存储在一个变量中。然后，我们可以在代码中稍后使用这个变量。例如：

```
**let sum** **= add(500, 500);**
undefined
**`I walked ${sum} miles`;**
'I walked 1000 miles' 
```

在这里，我们声明了变量 sum，并将其初始化为 add 函数的返回值，我们用 500 和 500 作为参数调用 add。尽管 add 函数有返回值，控制台显示的是 undefined，因为正如在第二章中讨论的那样，声明变量时总是打印 undefined。然后，我们通过将 sum 嵌入到模板字面量中，利用该函数的返回值，生成字符串"I walked 1000 miles"。

请注意，像当前写法的 sayHello 函数，我们无法做类似的事情。例如，我们不能用它来生成问候语"Hello, Nick!"，然后编写一些代码将该问候语嵌入到一个更长的字符串中。sayHello 函数返回的是 undefined，因为我们没有使用 return 关键字显式地给它返回值。它只是将问候语记录到控制台，而一旦记录完毕，就无法再次访问该问候语。

使用函数的返回值时，并不一定要将其存储在变量中。返回值的函数调用可以在任何可以使用值的地方使用，就像你可以交换使用变量和字面量值一样。例如，前面的示例可以这样重写：

```
**`I walked ${add(500, 500)} miles`;**
'I walked 1000 miles' 
```

在这里，我们不是单独调用 add 并将结果存储在一个变量中，而是从模板字面量内调用该函数。它的返回值直接插入到生成的字符串中，产生与之前相同的消息。通常将返回值存储在变量中会更易读，但两种方式都是有效的。

#### 参数类型

JavaScript 中的函数参数的数据类型是动态变化的。这是因为 JavaScript 是一种*动态类型*的编程语言，其中变量和参数的类型可以在程序运行时发生变化，而与此相对的是*静态类型*语言，其中变量和参数的类型在程序运行之前就已确定。

举个例子，到目前为止我们一直在使用 add 函数来相加数字，但没有什么能阻止我们将其用来连接两个字符串：

```
**add("Hello, ", "world!");**
'Hello, world!' 
```

在这里，我们将两个字符串作为参数传递给函数，因此函数体内的+运算符会被解释为字符串连接，而不是数字加法。因此，函数会将两个字符串合并并返回结果。

从这个角度看，我们也可以传递其他类型的参数，甚至在同一个函数调用中混合数据类型：

```
**add(true, false);**
1
**add(1, '1');**
'11' 
```

在这些情况下，JavaScript 关于类型强制转换的规则（在第二章中讨论过）会起作用。当我们尝试使用 add(true, false)将两个布尔值相加时，JavaScript 会在加法前将布尔值转换为数字 1 和 0，从而得到数字 1。当我们尝试使用 add(1, "1")将数字和字符串相加时，JavaScript 会将两个操作数都转换为字符串并将它们连接起来，得到字符串"11"。

动态类型为 JavaScript 带来了很大的灵活性，但如果不小心，它也可能引发一些令人困惑的 bug。了解你使用的类型非常重要，以确保你没有向期望数字的函数传递字符串类型的参数。

#### 副作用

*副作用*是指函数执行时，除了返回值外，会对函数外部产生影响的任何操作。副作用可以是有意的，也可以是无意的，包括更新函数外部声明的变量值、修改函数外部声明的数组或对象，或者向控制台输出字符串。

一些函数，比如我们的 add 函数，没有副作用，仅仅因为返回值而被调用。另一些函数，比如 sayHello，没有返回值，仅仅因为副作用而被调用。也可以编写同时返回值*并*具有副作用的函数。例如，我们可以重新定义 add 函数，除了返回参数和外，还可以将一些信息记录到控制台并更新一个变量：

```
**let addCalls = 0;**

**function add(x, y) {**
 **addCalls++;**
 **console.log(`x was ${x} and y was ${y}`);**
 **return x + y;**
**}** 
```

在这里，我们声明了变量 addCalls，用于跟踪 add 函数被调用的次数。然后，我们编写了更新后的 add 函数声明。现在该函数会先递增 addCalls，并将参数值记录到控制台中，然后像之前一样返回参数的和。

让我们尝试调用修改后的函数：

```
**let sum = add(Math.PI, Math.E);**
x was 3.141592653589793 and y was 2.718281828459045
**addCalls;**
1
**sum;**
5.859874482048838 
```

该函数调用具有副作用，即在将两个值相加之前，将它们记录到控制台中。它还有一个副作用，就是更新了 addCalls 变量，将其值从 0 改为 1。此外，函数还有一个（非副作用）结果，即返回其参数的和，我们已将其存储在 sum 变量中。

如果我们进一步调用 add，变量 addCalls 将每次递增，从而提供一个函数被调用次数的实时计数。通常，你不需要像这样跟踪函数的调用次数，尽管你可以使用这种机制来限制程序调用某些需要大量处理能力的函数的频率（这种技术被称为*速率限制*）。你可以通过定期重置计数器来实现这一点——也许每分钟一次——并且当计数器超过某个阈值时跳过函数调用。

### 将函数作为参数传递

在 JavaScript 中，函数是*一等公民*，这意味着它们可以像其他值一样使用，例如数字或字符串。例如，你可以将函数存储在变量中，或者将函数作为参数传递给另一个函数。后一种情况尤其常见，因为有许多函数将工作委托给其他函数。当一个函数作为参数传递时，它通常被称为*回调函数*，因为传递它的函数会“回调”它并执行它。

我们将通过 JavaScript 内置的 setTimeout 函数来说明这一点，setTimeout 允许你延迟调用另一个函数。它需要两个参数：一个要调用的函数，和一个等待时间（以毫秒为单位），即在调用该函数之前需要等待的时间。以下是它的工作原理：

```
**function sayHi() {**
 **console.log("Hi!");**
**}**
**setTimeout(sayHi, 2000);**
1
Hi! 
```

首先，我们创建一个没有参数的简单函数 sayHi，它只调用 console.log。然后我们调用 setTimeout，传递 sayHi 函数和数字 2000（表示 2000 毫秒，即 2 秒）作为参数。一旦按下 ENTER，setTimeout 应立即返回一个*超时 ID*——在这种情况下是 1——这是一个唯一的标识符，你可以用它来取消延迟的函数调用（如果需要的话）。然后，经过两秒钟，sayHi 函数被调用，字符串 "Hi!" 被记录到控制台。

> 注意

*要取消通过 setTimeout 延迟的函数调用，请调用 clearTimeout 函数，并将超时 ID 作为参数传递。*

注意，当我们将函数作为参数传递时，我们写下的是它的名字而不带括号：在这种情况下是 sayHi 而不是 sayHi()。没有括号的函数名仅仅是*引用*该函数，而带括号的函数名则实际*调用*该函数。我们可以在 JavaScript 控制台中看到这种区别：

```
❶ **sayHi;**
f sayHi() {
  console.log("Hi!");
}

❷ **sayHi();**
Hi!
undefined 
```

仅仅执行 sayHi; 不带括号 ❶ 会打印出函数的定义，但不会调用它。然而，执行 sayHi(); 带括号 ❷ 会调用 sayHi 函数，打印字符串 "Hi!" 并返回 undefined。

### 其他函数语法

到目前为止，我们在本章中主要关注了使用函数声明来创建函数，但 JavaScript 也支持其他创建函数的方法。函数声明遵循简洁的格式，并使用类似于许多其他编程语言（如 C++ 和 Python）中定义函数的语法。当你编写函数并打算直接调用时，像我们讨论过的 sayHello 和 add 函数，使用函数声明完全没问题。然而，一旦你开始将函数作为值来传递（例如作为参数），其他的创建函数的方式就会变得更加有用。接下来我们将介绍这些方式，首先从函数表达式开始。

#### 函数表达式

*函数表达式*，也称为*函数字面量*，是一种其值为函数的代码字面量，就像 123 是一个其值为数字 123 的字面量一样。而函数声明创建一个函数并将其绑定到一个名称上，函数表达式则是一个求值为（返回）函数的表达式，供你自由使用。

从语法上看，函数表达式与函数声明非常相似，主要有两个不同之处。首先，函数表达式不需要包括名称，尽管你可以根据需要添加名称。没有名称的函数表达式也被称为*匿名函数*。其次，函数表达式不能写在代码行的开头，否则 JavaScript 会认为它是一个函数声明；在 `function` 关键字之前必须有一些代码。这也是为什么函数表达式通常用于函数需要作为值来处理的上下文。

例如，你可以定义一个函数表达式并将其赋值为一个变量的值，所有操作都在一条语句中完成，如下所示：

```
**let addExpression = function (x, y) {**
 **return x + y;**
**};** 
```

在赋值语句的右侧出现`function`关键字，而不是在行首，因此 JavaScript 会将其视为一个函数表达式。在这种情况下，我们将函数表达式赋值给了 `addExpression` 变量。函数本身是匿名的，因为我们没有在`function`关键字后提供一个名称（在接下来的“命名函数表达式”框中，你会看到我们如何这样做的例子）。它有两个参数，x 和 y，括号中指定，就像我们最初的 add 函数一样。函数体返回这两个参数的和，并用大括号括起来，类似于函数声明的函数体，但请注意，我们需要在大括号闭合后加上分号，以表示将函数赋值给变量的语句结束。

尽管函数本身在技术上是匿名的，但它现在已绑定到 `addExpression` 变量。因此，我们可以通过在变量名后加上一对括号并传入必要的参数来调用该函数，就像调用任何命名函数一样：

```
**addExpression(1, 2);**
3 
```

输入 `addExpression(1, 2)` 会调用该函数，返回两个参数的和。

在许多方面，函数表达式和函数声明是可以互换的，所以选择这两种方法中的任何一种通常只是风格问题。例如，将我们的两个数字相加函数定义为函数表达式并将其赋值给一个变量，在大多数情况下等同于最初使用函数声明的方式。不过，当涉及将函数作为参数传递时，函数表达式提供了一些优势。例如，之前我们声明了 sayHi 函数，然后将它的名字传递给 setTimeout 作为参数。更常见的做法是直接在 setTimeout 函数的参数列表中写出一个等效的函数表达式，而不是先将其赋值给一个变量：

```
**setTimeout(function () {**
 **console.log("Hi!");**
**},** **2000);**
2
Hi! 
```

之前我们调用了 setTimeout(sayHi, 2000)，将一个函数名作为第一个参数传递，但这次我们传递的是一个函数表达式。该函数表达式定义了一个匿名函数，用于向控制台打印"Hi!"（这与我们之前声明的 sayHi 函数等效）。注意，`function`关键字不是代码行中的第一个元素，这是函数表达式的一个要求，并且闭括号后跟着一个逗号，因为这个函数表达式是参数列表的一部分。

如前所述，调用 setTimeout 会返回一个超时 ID，这次是 2。然后，当我们的匿名函数在两秒钟后被调用时，"Hi!" 会出现在控制台中。在这种情况下，使用函数表达式更简洁，因为我们不需要在传递给 setTimeout 之前单独定义延迟函数。

#### 箭头函数

JavaScript 还有另一种定义函数的语法，称为*箭头函数表达式*，或简称*箭头函数*。箭头函数是函数表达式的一种更简洁的版本，在大多数情况下，选择使用哪一种完全是风格上的问题。你可以在任何适用普通函数表达式的地方使用箭头函数，并在此过程中节省一些输入。例如，下面是使用箭头函数语法编写一个将两个数字相加的函数：

```
**let addArrow = (x, y) => {**
 **return x + y;**
**};** 
```

箭头函数不使用`function`关键字。相反，它以参数列表开始——在这个例子中是(x, y)——后面跟着一个箭头（=>）和函数体。在这里，我们将箭头函数赋值给 addArrow 变量，这样我们就可以像调用其他函数一样调用它：

```
**addArrow(2, 2);**
4 
```

我们使用*块体*语法定义了 addArrow，其中函数体被放置在大括号之间，每个语句都写在自己缩进的一行上。然而，如果函数体仅包含一个语句，还有一种更简单的语法，称为*简洁体*：

```
**let addArrowConcise = (x, y) => x + y;**
```

在这里，函数体与其余的语句写在同一行，并且没有被大括号包围。同时，`return`关键字是隐式的，这意味着函数体中的表达式（在此例中为 x + y）自动被理解为函数的返回值。这种简洁的函数体语法非常适合编写简单的函数，但如果你的函数体涉及多个语句，你就必须使用块体语法（如果函数有显式的返回值，还需要包含`return`关键字）。

如果箭头函数只有一个参数，你可以通过省略参数名周围的圆括号进一步简化语法：

```
**let squared = x => x * x;**
**squared(3);**
9 
```

这个箭头函数接受一个数字 x，并返回它的平方（x * x）。由于 x 是函数的唯一参数，我们不需要将其放在圆括号中。这对于块体和简洁体语法都适用。

像函数表达式一样，箭头函数提供了一种高效的方式来定义作为参数传递的函数。为了说明这一点，我们将考虑 JavaScript 内置的 setInterval 函数。像 setTimeout 一样，它接受另一个函数和一个时间（以毫秒为单位）作为参数，但与 setTimeout 不同的是，它会重复调用提供的函数，在每次调用之间等待指定的时间。例如，在这里，我们传递给 setInterval 一个箭头函数，它会将字符串 "Beep" 打印到控制台：

```
**setInterval(() => {**
 **console.log("Beep");**
**}, 1000);**
3
Beep 
```

我们的箭头函数不接受任何参数，因此它以一对空圆括号开始作为参数列表。函数体末尾的闭合大括号后面跟着一个逗号，用来将箭头函数与传递给 setInterval 的下一个参数分隔开，这个参数指定了每次重复之间的暂停时间（1,000 毫秒，即一秒钟）。

当我们执行这段代码时，它首先返回一个用于取消重复的间隔 ID——在本例中为 3。然后，在一秒钟的延迟后，第一个 "Beep" 会被打印出来。之后，控制台输出的左侧应该会显示一个数字，每秒递增，表示 console.log("Beep") 被调用的次数。Chrome 使用这个技巧来避免控制台中重复输出的行数过多。当你准备好停止 Beep 时，只需刷新浏览器页面，或者调用 clearInterval 函数并传入间隔 ID。在我们的例子中，应该是 clearInterval(3)。

### 剩余参数

有时你希望你的函数能够接受可变数量的参数。例如，假设你想编写一个函数，它接受某人的名字和他们最喜欢的颜色，并将它们打印成一个句子。你事先并不知道用户会输入多少个最喜欢的颜色，因此你希望让你的函数足够灵活，以处理传入的任意数量的颜色。在 JavaScript 中，你可以使用 *剩余参数* 来实现这一点，剩余参数是一种特殊类型的参数，它将可变数量的参数收集到一个数组中。

剩余参数可以与任何样式的函数定义一起使用。这里我们使用一个剩余参数来创建一个箭头函数，用于列出用户的最爱颜色：

```
**let myColors = (name, …favoriteColors) => {**
 **let colorString** **= favoriteColors.join(", ");**
 **console.log(`My name is ${name} and my favorite colors are ${colorString}.`);**
**};**
**myColors("Nick", "blue", "green", "orange");**
My name is Nick and my favorite colors are blue, green, orange. 
```

一个 rest 参数看起来像是一个普通参数，前面有三个点，它必须始终是函数定义中列出的最后一个参数。当调用函数时，任何常规参数（按顺序列出）都会与第一个提供的参数匹配。然后，rest 参数将其余的参数捆绑成一个数组。在我们的示例中，name 是一个常规参数，favoriteColors 是 rest 参数。当我们调用函数时，"Nick" 参数会被分配给 name 参数，其余的参数 "blue"、"green" 和 "orange" 会被收集到一个数组中，并分配给 favoriteColors 参数。因为 favoriteColors 是一个数组，我们可以使用 join 方法将其转换为一个字符串，用逗号和空格分隔每个颜色。然后，我们使用模板字面量将颜色字符串融入到一个更大的字符串中，并使用 console.log 打印出来。

由于 favoriteColors 是一个 rest 参数，我们可以根据需要使用任意数量的颜色：

```
**myColors("Boring", "gray");**
My name is Boring and my favorite colors are gray.
**myColors("Indecisive", "red", "orange", "yellow", "green", "blue", "indigo", "violet");**
My name is Indecisive and my favorite colors are red, orange, yellow, green, blue, indigo, violet. 
```

无论我们提供多少个参数，函数仍然能够正常工作。

这是另一个使用 rest 参数的示例，这次是将所有作为参数提供的数字相加：

```
**function sum(…numbers) {**
 **let total = 0;**
 **for (let number of numbers) {**
 **total += number;**
 **}**
 **return total;**
**}**
**sum(1, 2, 3, 4, 5);**
15
**sum(6, 7, 8, 9, 10, 11, 12, 13);**
76 
```

这次我们使用了一个函数声明，而不是箭头函数，且函数的唯一参数是 rest 参数。由于没有其他参数，所有参数都被收集到一个数组中，并分配给 numbers rest 参数。然后，我们使用 for…of 循环将这些数字加起来。

### 高阶函数

*高阶函数* 是一种接受另一个函数作为参数，或者输出另一个函数作为返回值的函数。在本章中，你已经见过了两个高阶函数：setTimeout 和 setInterval，它们都接受一个回调函数作为参数，稍后执行。JavaScript 还有许多其他内置的高阶函数。我们将在这里考虑一些，并讨论如何编写你自己的高阶函数。

#### 接受回调的数组方法

有许多内置的方法用于处理数组，这些方法接受一个回调函数。记住，方法是一种对对象（如数组）操作的函数。在大多数情况下，传递给这些高阶数组方法的回调函数会为数组中的每一项执行一次。让我们看几个示例。

查找数组元素

find 数组方法用于查找数组中第一个符合某些条件的元素。你可以通过回调函数指定条件，回调函数返回一个布尔值 true/false。例如，如果我们想查找购物清单中第一个字符数大于六的项，可以这样做：

```
**let shoppingList** **= ["Milk", "Sugar", "Bananas", "Ice Cream"];**
**shoppingList.find(item => item.length > 6);**
'Bananas' 
```

我们传递给 find 的回调函数是 item => item.length > 6。这个回调函数利用了箭头函数的两个有用的语法特性。首先，由于我们的函数只有一个参数 item，我们可以省略括号。其次，由于函数体仅包含一个语句 item.length > 6，我们可以使用简洁的函数体语法，省略 return 关键字和大括号。这些特性让我们能够尽可能简洁地定义查找元素的逻辑，使得箭头函数非常适合编写简单的回调函数。

find 方法依次对数组中的每个元素执行回调函数。回调函数接受元素并根据该元素是否包含超过六个字符来返回 true 或 false。如果回调函数对某个元素返回 true，find 方法将返回该元素并停止搜索。在这种情况下，方法返回 "Bananas" 而不是 "Ice Cream"，因为 "Bananas" 在数组中出现得更早。

如果没有找到符合条件的项，find 方法将返回 undefined：

```
**shoppingList.find(item => item[0] ===** **"****A****"****);**
undefined 
```

这次我们给 find 传递一个回调函数，该函数检查元素是否以字母 *A* 开头。购物清单中的项目没有任何一个以字母 A 开头，因此方法返回 undefined。

过滤数组中的元素

filter 方法返回一个新数组，其中包含原数组中所有满足某些条件的元素。与 find 方法一样，条件是通过回调函数来指定的。为了说明这一点，我们将更新原来的 find 示例，将方法名改为 filter。这样，我们将得到一个包含所有超过六个字符的项目的列表，而不仅仅是通过测试的第一个项目：

```
**let shoppingList** **= ["Milk", "Sugar", "Bananas", "Ice Cream"];**
**shoppingList.filter(item => item.length > 6);**
`(2) ['Bananas', 'Ice Cream']` 
```

这将过滤掉字符长度太短的数组元素，同时将 "Bananas" 和 "Ice Cream" 保留在结果数组中。

转换数组中的每个元素

有时你可能希望转换数组中的每个元素，并将结果存储在一个新数组中。例如，你可能有一个包含数字的数组，这些数字需要以相同的方式进行操作。你可以使用我们在第四章中讨论的 for…of 循环来实现，但更简洁的方法是使用 map 数组方法。它将相同的回调应用于数组中的每个元素，并返回一个包含结果的新数组。例如，在这里，我们使用 map 来接受一个数字数组并生成一个包含这些数字立方的新数组：

```
**let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];**
**let cubes = numbers.map(x => x * x * x);**
**cubes;**
`(10) [1, 8, 27, 64, 125, 216, 343, 512, 729, 1000]` 
```

我们的回调函数 x => x * x * x 接受数组元素并将其立方。map 方法将这个回调应用到 numbers 数组的每个元素，返回一个包含前 10 个完美立方数的新数组，同时保持原数组不变。将 map 与箭头函数的简洁语法进行比较，看看使用 for…of 循环的等效代码：

```
**let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];**
**let cubes = [];**
**for (let x of numbers) {**
 **cubes.push(x * x * x);**
**}**
**cubes;**
`(10) [1, 8, 27, 64, 125, 216, 343, 512, 729, 1000]` 
```

结果是相同的，但使用 map 时，我们能够在一行代码中声明并填充 cubes 数组，而不需要先声明 cubes 为一个空数组，然后在 for…of 循环体内填充它。

如果你有一个包含相似对象的数组，并且你想从每个对象中提取相同的信息，map 方法也很有用。例如，假设你有一个表示商店商品的对象数组，每个对象都有一个名称和价格属性，而你想要获取一个仅包含价格的数组。你可以传递给 map 一个回调函数，访问每个对象的价格属性，像这样：

```
**let stockList = [**
 **{name: "Cheese", price: 3},**
 **{name: "Bread", price: 1},**
 **{name: "Butter", price: 2}**
**];**
**let prices = stockList.map(item => item.price);**
**prices;**
`(3) [3, 1, 2]` 
```

在这里，回调函数是 item => item.price，它接收一个 item 并返回该 item 的 price 属性的值。map 函数将回调依次应用于原始数组中的每个对象，并创建一个包含所有价格的新数组。

一般来说，尽可能使用 map 而不是等效的循环，因为 map 方法简洁，而且代码具有 *自文档化* 特性（map 方法的名称暗示你正在创建一个新数组，该数组从另一个数组中复制并修改元素，无需进一步注释）。当你的需求更加定制时，使用循环会更合适，例如，当输出数组中的元素数量与原始数组中的元素数量不匹配时。

#### 接受回调函数的自定义函数

要创建一个接受回调作为参数的高阶函数，只需像命名其他参数一样，在函数的参数列表中包含回调的名称。然后，当你想在函数体内调用回调时，在参数名称后添加括号，就像调用任何其他函数一样。让我们通过声明一个接受回调并调用它两次的 doubler 函数来说明这一点：

```
**function doubler(callback) {**
 **callback();**
 **callback();**
**}** 
**doubler(() => console.log("Hi there!"));**
Hi there!
Hi there! 
```

当我们定义 doubler 函数时，我们给它一个回调参数。然后，在函数体内，我们写两次 callback()，以对传递给该参数的函数进行两次调用。当我们调用 doubler 时，我们传递给它一个将 "Hi there!" 打印到控制台的函数，因此该消息会被打印两次。请注意，这个回调函数不需要任何参数，因此我们在箭头符号之前写了一个空的括号。

如我们所讨论的，JavaScript 对函数参数的集合数据类型没有概念，因此没有任何东西阻止我们尝试传递一个不是函数的值作为 doubler 的参数。但是，如果我们这样做，当 JavaScript 尝试调用非函数时，会出现错误：

```
**doubler("hello");**
Uncaught TypeError: callback is not a function
at doubler (<anonymous>:2:5)
at <anonymous>:1:1 
```

在这里，我们传递给 doubler 一个字符串，而不是一个函数，因此我们会得到一个 TypeError 错误。

我们传递给 doubler 的回调函数不需要任何参数，但你也可以设置一个高阶函数，使其回调函数接受参数。例如，在这里，我们创建一个函数，该函数调用另一个函数若干次，将当前调用次数传递给回调函数：

```
**function callMultipleTimes(times, callback) {**
 **for (let i** **= 0; i < times; i++) {**
 **callback(i);**
 **}**
**}** 
```

我们声明 `callMultipleTimes` 函数有两个参数：一个要调用的函数（`callback`）和调用次数（`times`）。（注意，与 `setTimeout` 和 `setInterval` 中回调函数是第一个参数不同，我们这里的函数遵循更常见的 JavaScript 约定，将回调函数放在最后一个参数。）函数体由一个 `for` 循环组成，在该循环中我们调用 `callback(i)`，将循环变量 `i` 作为参数传递给回调函数。

因为回调函数只接收一个参数，我们知道传递给 `callMultipleTimes` 的回调函数应该有一个参数。例如：

```
**callMultipleTimes(3, time => console.log(`This was time: ${time}`));**
This was time: 0
This was time: 1
This was time: 2 
```

这里我们传递了一个箭头函数作为回调函数。它有一个 `time` 参数。这个函数将时间信息整合到一条消息中，并将其记录到控制台。每次执行这个回调时，`time` 都会取循环变量 `i` 的当前值，分别将 0、1 和 2 插入到记录的消息中。

#### 返回函数的函数

到目前为止，我们关注的是将函数作为参数的高阶函数，但高阶函数也可以输出一个函数作为返回值。例如，假设你想创建多个函数，将一个后缀添加到字符串的末尾，比如添加 "!!!" 使字符串看起来更激动人心，或者 "???" 使它看起来更加迷惑。与其为每个可能的后缀手动定义一个单独的函数，或者为文本和后缀定义一个函数，每次调用时都必须提供后缀，不如定义一个高阶函数，它接收一个后缀并返回一个将该后缀附加到字符串的函数：

```
**function makeAppender(suffix) {**
❶ **return function (text) {**
  ❷ **return text + suffix;**
 **};**
**}** 
```

这里有两个 `return` 关键字。第一个 ❶ 被高阶函数 `makeAppender` 使用，用来返回一个匿名函数。后面跟着 `function` 关键字，表示我们正在定义一个将被返回的函数。第二个 `return` 关键字 ❷ 出现在匿名函数内部。当*那个*函数被调用时，它会返回匿名函数的 `text` 参数与 `makeAppender` 函数的 `suffix` 参数连接后的值。

为了能够调用内部函数，我们首先必须通过调用外部函数来访问它：

```
**let exciting = makeAppender("!!!");**
**exciting("Hello");**
'Hello!!!' 
```

调用 `makeAppender("!!!")` 返回一个新的函数，我们将其赋值给 `exciting` 变量。这个变量现在包含了从 `makeAppender` 返回的函数表达式，它接受一个字符串作为参数。当我们调用 `exciting("Hello")` 时，我们得到字符串 "Hello!!!"，这是将 "Hello" 和 "!!!" 两个字符串连接起来的结果。

我们的高阶函数 `makeAppender` 的好处在于，我们可以利用它生成用于附加其他后缀的函数，而不仅仅是 "!!!"。例如：

```
**let puzzling =** **makeAppender("???");**
**puzzling("Hello");**
'Hello???'
**let winking = makeAppender(" ;-)");**
**winking("Hello");**
'Hello ;-)' 
```

在这里，makeAppender 返回了两个额外的函数，我们将它们分别赋值给了 puzzling 和 winking 变量。我们只需要定义一个高阶函数，但现在我们有了三个不同的后缀附加函数可以选择，并且可以随意重复使用它们：

```
**winking("Goodbye");**
'Goodbye ;-)'
**puzzling("Goodbye");**
'Goodbye???'
**exciting("Goodbye");**
'Goodbye!!!' 
```

请注意，我们从 makeAppender 返回的每个函数都记住了我们传入的 suffix 值，这就是它能够不断地附加相同后缀的原因。每个函数都是在 makeAppender 的作用域内定义的，因此即使 makeAppender 的调用已经完成，它返回的内部函数仍然能够保持对该作用域中其他值的访问，包括 suffix。

我们在第四章讨论了作用域，例如，如何定义在 while 或 for 循环中的变量无法在循环外部访问。类似地，在函数内部定义的变量只在该函数内部有效，因此通常在函数调用结束后就会消失。然而，作用域在嵌套函数中变得更加有趣，如当前示例所示。你可能会认为外部的 makeAppender 函数在调用后会“消失”，但内部函数保留了对来自该作用域的变量和参数的访问，只要我们保持对内部函数的引用（我们通过变量 exciting、puzzling 和 winking 来实现）。那些保留对其所在作用域中的变量和参数访问的函数被称为*闭包*，因为它们“封闭”了它们的环境。（可以想象，内部函数有一个罩子，能够保持作用域中的所有变量。）

### 总结

本章中，你学到了如何通过创建和使用自定义函数使你的代码更加易读和简洁。你了解了定义函数的三种主要方式——函数声明、函数表达式和箭头函数，并且实验了块体语法和简洁体语法。你学会了如何通过传递参数值来向函数提供输入，并了解了如何利用函数的工作成果，无论是通过其返回值、其副作用，还是两者兼有。你还看到了如何将函数赋值给变量，以及如何将函数传递给高阶函数或从高阶函数返回。
