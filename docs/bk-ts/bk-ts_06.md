# 第六章：模板字符串

# 模板字符串

模板字符串仅仅勉强值得拥有自己的章节。它们既不是一种类型，也不是 TypeScript 提供的支持面向对象编程的任何构建块之一。然而，它们非常有用。本书中接下来的大部分示例都会使用它们。

模板字符串是一个带有占位符的“常规”JavaScript 字符串。在运行时，您的代码将占位符替换为实际值。

使用`${}`表示占位符。将变量或函数放在大括号内，它将在运行时进行评估。

以下是一个示例：

```
const userName: string = "Paul";
const helloTemplate: string = `Hello, ${userName}!`;
console.log(helloTemplate); 
```

在运行时，代码将`userName`替换为 userName 的实际值。

这里有另一个示例，它将模板字符串作为参数传递给`console.log()`。

```
function getRandomName() {
    const names: string[] = ["Paul","Aidan","Kelly", "Amina"];
    return names[Math.floor((Math.random() * 4));
}
console.log(`Hello, ${getRandomName()}`!); 
```

在上面的示例中，占位符是一个函数。

这是一个很好的快捷方式，特别是当您将其与普通 JS 等效对比时：

```
function getRandomName(): string {
    var names: string[] = ["Paul","Aidan","Kelly", "Amina"];
    return names[Math.floor((Math.random() * 4));
}
console.log("Hello, " + getRandomName() + "!"); 
```

您可以使用任意数量的占位符：

```
const name: string = "Paul";
const graduatedYear: number = 1987;

console.log(`${name} graduated in ${graduatedYear}.`); 
```

我已经训练自己在需要定义字符串时使用反引号字符。您永远不知道何时可能需要在运行时将一些数据引入字符串中。

您将在下一章节中看到展示模板字符串的视频。

模板字符串实际上支持更多功能，称为“标记模板”。这些允许您在发出字符串之前对其进行预处理。我从未在现实世界中看到过这种功能的使用，所以我在这里没有涉及。阅读这里的内容，并自行决定是否认为您会使用它们：[`basarat.gitbooks.io/typescript/docs/template-strings.html`](https://basarat.gitbooks.io/typescript/docs/template-strings.html)

字符串模板就是这样。让我们回到细节中，深入研究 TypeScript 函数。
