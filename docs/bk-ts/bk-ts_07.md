# 第七章：函数

# 函数

TypeScript 支持 JavaScript 函数，就像您已经了解的那样。正如它经常做的那样，TypeScript 添加了新的功能，包括：

+   函数类型

+   类型化参数

+   可选参数

+   Rest 参数^(1)

+   箭头函数（通常也称为 Lambda 或匿名函数）

## 函数类型

在 TypeScript 中，一切都有类型。如果您不指定类型，TypeScript 将推断出一种类型，对于函数，它是 `Function`。您可以声明变量的类型为 `Function`，如下所示：

```
let sillyAdderFunction: Function;

sillyAdderFunction = function(a, b) { return a + b};

console.log(sillyAdderFunction(10, 10); 
```

这并不是一个非常有用的事情，但它显示了有一个`Function`数据类型。您将在下面看到箭头函数更加有用。

## 函数参数

大多数函数至少需要一个参数，并且大多数有用的函数返回一个值^(2)。您可以为每个函数参数以及函数本身的返回类型指定类型。这是一个更健壮的`integerAdder()`：

```
function integerAdder(firstNumber: number, secondNumber: number): number {
    return firstNumber + secondNumber;
}

const TwoPlusTwo = integerAdder(2, 2);

// Error: can't pass string to numeric function argument.
const errorAdder1 = integerAdder("ham", "cheese");

// Error: errorAdder2 is a string but the function returns an integer.
const errorAdder2: string = integerAdder(2, 2); 
```

代码定义了一个函数，`integerAdder`。它接受两个参数，正如您所看到的，它们都是整数值。此外，integerAdder 函数本身返回一个数字。

这是一个演示的 30 秒视频：

[`www.youtube.com/embed/zUI2Ygfdhj0`](https://www.youtube.com/embed/zUI2Ygfdhj0)

（如果您看不到视频，请尝试[通过此链接访问](https://youtu.be/zUI2Ygfdhj0)或将此网址键入到您的网络浏览器中：[`youtu.be/zUI2Ygfdhj0`](https://youtu.be/zUI2Ygfdhj0)）。

请注意，integerAdder 指定了其参数的类型以及其返回值的类型：`integerAdder(firstNumber: number, secondNumber: number): number`。这告诉 TypeScript 编译器足够的信息，以防止您在纯 JavaScript 世界中犯的两个常见错误：

+   您不能将非数字值传递给 integerAdder 函数。

+   结果是数字。您不能调用该函数并将其结果接受到非数字变量中。

函数不需要返回一个值。如果您想要明确（而您可能确实想要！），请指定返回类型为 `void`：

```
function noOperation(): void { 
    return; 
} 
```

### 可选参数

我们可以定义带有可选参数的函数。想象一下，您的应用程序中有一个名为 `InitializeDataSet` 的函数。有时，您希望将数据集初始化为一组硬编码的值（即默认值）。在其他情况下，您希望提供一些“种子”数据。

```
function InitializeDataSet(seedData?: any) {
    if (seedData) {
        // use the seed data to initialize the data set
    }
    else {
        // initialize using some default hard coded values
    }
} 
```

使用问号（?）来表示可选参数。

在运行时，客户端代码像往常一样调用该函数。如果该代码没有为可选参数提供值，其值为 `undefined`。

### 默认参数值

您可以为函数的参数指定默认值。以下是先前示例的重写版本，以显示此语法并讨论使用它的影响：

```
function InitializeDataSetWithDefaultValues(seedData = { seedValue1: "a", seedValue2: "b"}) {
    // seedData will have valid data so no need to check it.

    /*
    if (seedData) {
        // use the seed data to initialize the data set
    }
    else {
        // initialize using some hard coded values
    }
    */
}

InitializeDataSetWithDefaultValues();
InitializeDataSetWithDefaultValues(undefined);
InitializeDataSetWithDefaultValues({seedValue1: "x", seedValue2: "y"}); 
```

请注意：

+   函数接受一个输入参数 `seedData`。TypeScript 推断参数的类型为具有两个字符串属性的对象。

+   如果客户端代码没有传递 seedData 的值，它将使用特定的初始值。

+   如果你向函数传递值`undefined`，它也会返回指定的初始值。

## 剩余参数

有时候你想要向函数传递未知数量的参数。这种情况在 React 开发、数据序列化和反序列化以及映射过程中经常出现。

让我们考虑一个日志记录的例子。我们总是可以使用`console.log()`将消息记录到控制台。然而，在运行时调试应用程序是一个真正的挑战，特别是最终用户报告的错误。当有人告诉你有错误时，你已经无能为力了。让我们在工具包中添加一些持久性错误记录：

```
function myLogger(msgType: "INFO" | "ERROR", ...messages: any[]) {
    if (msgType === "INFO") {
        console.log(messages);
    }
    else  {
        // Save the details to local storage for future analysis/debugging
        localStorage.setItem("lastErrorMessage", JSON.stringify(messages));
        console.error(messages);
    }
}

myLogger("INFO","Greetings!");
myLogger("INFO", "Successfully saved the data, results:", {someResult: "", databaseResultCode: 1});
myLogger("ERROR", "ERROR: Failed to save the data, error details follow.", {errorDetails: "[some error details object goes here]"}, "Error occurred at `${new Date()}`"); 
```

代码执行以下操作：

+   定义了一个函数，`myLogger`。

+   myLogger 接受两个参数：`msgType` 和 `messages`。

    +   msgType 是一个联合数据类型 - 客户端代码必须传递"INFO"或"ERROR"。

    +   messages 是一个`any`类型的数组。注意变量名前面的`...`。这表示所有剩余的参数都将被放入数组中。

+   你可以看到代码如何调用 myLogger 函数，在其三次调用中传递不同数量的参数。

## 箭头函数

箭头函数的命名来源于它们的语法。

许多 TypeScript 开发者更喜欢称这些*lambda*函数，有时候也称为*匿名*函数。它们几乎总是指同一件事情。

箭头函数非常有用。它们提供了一个很好的语法缩写，并且更重要的是，通过将`this`关键字重新定义为更直观的内容，有助于减少对 JavaScript 作用域的混淆。

这里有一个非常简单的示例来帮助我们开始学习语法：

```
const myHelloFunction = () => { return "Hello!"};

myHelloFunction(); 
```

这个例子定义然后调用了一个名为`myHelloFunction`的函数。由于 myHelloFunction 是`const`，我们需要在定义时初始化它。这就是这部分内容：`{return "Hello!"};` myHelloFunction 现在是一个普通函数，我们可以像调用其他 JavaScript 函数一样使用函数调用运算符`()`来调用它：`myHelloFunction()`;

我们不需要定义函数体。例如，这是完全正确的语法：

```
let myGoodbyeFunction: () => string;
myGoodbyeFunction = () => {return "Good Bye!"}
console.log(myGoodbyeFunction()); 
```

第一行定义了一个变量，`myGoodbyeFunction`。myGoodbyeFunction 的类型是一个不带参数并返回字符串的函数。

第二行给 myGoodbyeFunction 赋值。在这种情况下，它是函数体本身。

最后，代码记录执行 myGoodbyeFunction 的结果。

在继续之前，让我们看一下转译后的 JavaScript。这是 myHelloFunction 的转译后的 JavaScript：

```
var myGoodbyeFunction;
myGoodbyeFunction = function () { return "Good Bye!"; };
console.log(myGoodbyeFunction()); 
```

这是逐行转换：

| **TypeScript** | **Transpiled JavaScript** |
| --- | --- |
| 1\. let myGoodbyeFunction: () => string; | var myGoodbyeFunction; |
| 2\. myGoodbyeFunction = () => {return "Good Bye!"} | myGoodbyeFunction = function () { return "Good Bye!"; }; |
| 3\. console.log(myGoodbyeFunction()); | console.log(myGoodbyeFunction()); |

`let`转译为`var`。箭头函数转译为直接的`function`定义。

### 在箭头函数中指定参数

我们通过将参数插入到函数定义的 `()` 部分来指定参数。以下是一个接受两个整数输入并返回数字结果的加法器函数：

```
let myAdderArrowFunction: (arg1: number, arg2: number) => number;
myAdderArrowFunction = (firstNumber: number, secondNumber: number) => {
        return firstNumber + secondNumber;
    }
console.log(myAdderArrowFunction(2, 2)); 
```

该代码使用箭头语法定义了一个新函数，`myAdderArrowFunction`。myAdderFunction 接受两个数值参数并返回数字。

重要的是要记住，`let` 语句只是定义了一个名为 myAdderFunction 的类型化变量。它的类型恰好是带有类型签名和类型结果的 `Function`。

第二行初始化了 myAdderArrowFunction。请注意，当指定输入参数时，我没有使用相同的名称。同样，`let` 语句正在定义一个类型 - 一个接受两个参数并返回数字的函数。只要满足类型的要求，参数名称就无关紧要。

### 箭头函数作为接口组件

考虑到箭头函数可以独立于其实现定义类型，您可以在任何需要类型的地方使用它们，包括接口。出于许多原因，这是一种非常有用的能力。其中一个原因是当您使用未使用 TypeScript 编写的外部库时。这在现实世界中显然经常发生^(3)。让我们假设我们正在使用这样一个库，该库负责创建详细、高度交互式的可视化效果。我们不知道它是如何工作的，也不在乎。我们查阅了库的文档，看到它提供了一个强大的 JavaScript API，其中包括以下函数：

+   `Render`：我们提供页面上一个 `<div>` 标签的 ID，它会在那里呈现可视化效果。

+   `SetDimensions`：我们可以使用此调用设置高度和宽度。

+   `SaveSettings`：我们要求引擎保存当前屏幕设置以供下次使用。

这里有一些 TypeScript，尽管供应商的 API 没有提供，但让我们为我们的代码注入了一些强类型：

```
interface IVisualizationEngine {
    Render: (htmlDivName: string) => boolean;
    SetDimensions: (width: number, height: number) => void;
    SaveSettings: () => boolean;
}

// Assume that the visualization engine was already loaded
// and that we can get a handle to its API set via the global window object.
const myVisualizationEngine: IVisualizationEngine = <IVisualizationEngine>window["VisualizationEngine"];

if (myVisualizationEngine.Render("myDiv")) {
    myVisualizationEngine.SetDimensions(1024, 800);
    if (myVisualizationEngine.SaveSettings()) {
        console.log("Successfully saved the visualization.");
    }
    else {
        console.error("ERROR: Failed to save the visualization.");
    }
}
else {
    console.error("Failed to render the visualization!");
} 
```

这是一个复杂的示例。让我们拆开它：

+   该代码定义了一个接口，`IVisualizationEngine`。

+   该接口定义了三个不同的函数，每个函数对应我们关心的 API 调用^(4)。

+   我们通过全局 window 对象获取了引擎的句柄。为了使这个工作，我们必须通过 HTML 中的 script 标签引用引擎，并且引擎必须保存在全局 window 中。

到目前为止，我们为自己做了一件非常好的事情。我们现在可以强类型访问这个第三方的 API！这允许 IDE 为我们提供我们所钟爱的节省时间和减少错误的智能提示。

这里是另一个视频展示了这些概念。在这种情况下，我们将模拟一个已经生产和稳定了很长时间的现有联系人管理库。该视频创建了一个接口到该普通的 JS 库，然后使用它：

[`www.youtube.com/embed/e1BGcBO8E6U`](https://www.youtube.com/embed/e1BGcBO8E6U)

（如果您无法看到视频，请尝试[点击此链接](https://youtu.be/e1BGcBO8E6U)直接在 YouTube 上观看。您也可以尝试在您喜欢的网络浏览器中键入此链接：[`youtu.be/e1BGcBO8E6U`](https://youtu.be/e1BGcBO8E6U)。）

### 箭头函数作为 IIFEs

正如我们上面看到的，箭头函数转译为标准的 JavaScript 函数。我们可以在定义时调用函数。这些被称为立即调用函数表达式（IIFEs）。箭头函数也可以随时调用。这里有一个刻意的例子：

```
console.log(`Hello, ${(() => {return "Paul";})()}`); 
```

这段代码定义了一个函数：`() => { return "Paul"}`。它将该行代码封装在自己的括号中，然后使用调用函数运算符`()`立即调用该函数。这本身被包裹在一个模板字符串中，最后，结果被记录在控制台中。

这里是转译后的 JavaScript：

```
console.log("Hello, " + (function () { return "Paul"; })()); 
```

这个（可能过度制作的 :)) 视频展示了实时演示：

[`www.youtube.com/embed/DTBpHxWPf_w`](https://www.youtube.com/embed/DTBpHxWPf_w)

（如果您无法看到视频，请[尝试点击这里](https://youtu.be/DTBpHxWPf_w)或将此 URL 键入到您的网络浏览器中：[`youtu.be/DTBpHxWPf_w`](https://youtu.be/DTBpHxWPf_w)。）

### 用箭头函数编写更干净的代码

前面的示例没有很好地展示出来，但箭头函数可以帮助你做更多事情，而不仅仅是教你的 IDE 有关函数类型。它还可以帮助你编写更干净的代码，尽管这可能是主观的。这里有一个例子：

```
const numbers: number[] = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const evenNumbers = numbers.filter( (item) => !(item % 2));
const sumOfNumbers = numbers.reduce((prev: number, curr: number) => {
    return prev + curr;
}, 0); 
```

在这个示例中，我们定义了一个包含 1 到 10 的数字的数组。

然后我们定义一个变量`evenNumbers`。evenNumbers 是对数字数组进行过滤的结果，仅返回模二等于零的项目。这对我们来说是一种新形式：

`(item) => !(item % 2)`

我们完全可以将其写成：

`(item) => { return !(item % 2); }`

当您可以用单个语句返回一个值时，我们被允许使用这种简写。

第二个示例使用标准的`reduce`函数将所有数字相加。Reduce 针对数组运行，并将函数和初始值作为参数传递。该函数传递了前一个值和当前值，在示例中定义为`(prev: number, curr: number)`^(5)。请注意，`reduce`*也*传递第三个参数，即数组中项目的索引。我们不关心这一点，所以我们不会去捕捉它，可以这么说。

大多数 TypeScript 开发人员发现箭头函数非常有帮助，通常比一直拼写单词"function"更清晰。

# 进一步阅读

+   这篇博文描述了这里涵盖的大部分内容，还有两个叙述视频：[`executeautomation.com/blog/working-with-functions-anonymous-and-arrow-functions-in-typescript/`](http://executeautomation.com/blog/working-with-functions-anonymous-and-arrow-functions-in-typescript/)

+   这篇博文提供了对箭头函数的详细视角，特别是它解释了在这种情境下 `this` 的工作原理。[`piotrwalat.net/arrow-function-expressions-in-typescript/`](http://piotrwalat.net/arrow-function-expressions-in-typescript/)

# 函数总结

TypeScript 通过...增强了普通的 JavaScript 函数。

+   定义函数参数的类型（字符串、布尔值，甚至是接口类型）

+   定义函数的返回类型（不仅仅是参数）

+   为参数指定默认值

+   通过 `rest` 操作符（"..."）指定可变数量的参数

像其他 TypeScript 的产物一样，函数本身也是有类型的，它们的类型是 `Function`。

TypeScript 引入了一种不同类型的函数，箭头函数。我们还没有听到箭头函数的最后一句话。我们将在第九章“深入类”中重新讨论它们，并探索它们如何帮助简化 JavaScript 闭包。简而言之，它们与 `this` 关键字一起以更直观的方式工作。

函数就讲到这里了。下一章将温柔地介绍基本的 TypeScript 类，并为更高级的主题奠定基础，比如抽象类以及接口与其配合的工作原理。

* * *

> ¹. 不要与表述性状态转移混淆。如你所见，rest 参数是完全不同的东西。↩
> 
> ². 如果你的函数不返回任何值，很可能会影响到其范围之外的一些数据。这通常是一个坏事，因为它引入了副作用的风险。如果你想更有说服力，你可以[从这里开始](https://softwareengineering.stackexchange.com/questions/15269/why-are-side-effects-considered-evil-in-functional-programming)。↩
> 
> ³. 在现实世界中这种情况经常发生，TypeScript 通过使用“类型文件”提供了一个重要的功能集来处理这个挑战。本书的初稿没有详细讨论类型文件，但“继续学习”章节会指引你朝着正确的方向前进。↩
> 
> ⁴. 在这个上下文中，“关心”意味着库可能提供其他有用的函数，但出于某种原因我们不打算使用它们。我们不需要将每一个函数都映射到我们的接口定义中，只需我们计划使用的函数。↩
> 
> ⁵. `Reduce`，以及 `filter` 和 `map` 倾向于出现在遵循函数式编程风格的代码中。我在我的博客上写了一系列关于这个主题的小系列文章，链接在这里：[`hackernoon.com/functional-programming-the-examples-series-851421e7ae5b`](https://hackernoon.com/functional-programming-the-examples-series-851421e7ae5b)。↩
