# 第一部分：结束语

> 原文：[`jsprimer.net/basic/other-parts/`](https://jsprimer.net/basic/other-parts/)

在第一部分的基本语法中，我们看到了 JavaScript 的语法和用法，涵盖了 ECMAScript 规范范围内的内容。在第二部分的用例中，我们将应用学到的基本语法，实现一些小型应用程序，并深入了解 JavaScript。此外，我们还将学习如何使用特定于 Node.js 或浏览器执行环境的 API 和库。

在第一部分中，并没有介绍 ECMAScript 的所有语法或内置对象。还有许多未介绍的语法和内置对象也非常有用。

例如，[Proxy](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 和 [Reflect](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Reflect) 这样的内置对象可以为对象的基本操作（如获取或设置属性等）定义自定义行为。此外，`Object` 的内置对象也有一个名为 [Object.defineProperty](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 的方法，它可以修改对象的描述符（descriptor）。通过修改对象的描述符，可以设置对象的元操作，例如禁止修改对象的属性等。

除此之外，这些 API 更多地用于创建库而不是应用程序。此外，它们不会在第二部分的用例中出现，因此本书中省略了这些内容。当您需要时，查找并记住这些 API 的用法将是明智的选择。

几乎所有 JavaScript API 都在多个地方出现过，其中大部分内容都记录在[MDN Web Docs](https://developer.mozilla.org/ja/ "MDN Web Docs")中。MDN 不仅包含 ECMAScript 的功能，还包括了浏览器特定的 DOM API。因此，可以将 MDN 视为 JavaScript 的实质性官方参考文档。

在第二部分中，我们还将涉及到 Node.js 和浏览器特定的 DOM API。由于存在大量依赖于这些执行环境的 API，因此了解如何查找 API 是至关重要的。

例如，Node.js 有官方的参考文档[Node.js 文档](https://nodejs.org/api/)。如果是在浏览器中，MDN Web Docs 是实质上的官方参考文档，前面已经提到过。

另外，当您想了解正在使用的库或工具的用法时，请务必查看该库等的官方网站或存储库。正如在“ECMAScript”章节中介绍的那样，ECMAScript 和 JavaScript 一直在不断变化。因此，由于某些库可能会在不经意间成为已弃用状态，因此首先查看源头信息至关重要。

没有固定的正确的查找方法。但是，了解如何查找是重要的，这样当你想查找时就能找到所需的信息。
