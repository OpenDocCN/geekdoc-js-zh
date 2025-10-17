# ECMAScript

> 原文：[`jsprimer.net/basic/ecmascript/`](https://jsprimer.net/basic/ecmascript/)

到目前为止，我们已经看到了 JavaScript 基本语法的内容，但让我们看看定义这些语法的 ECMAScript 规范是如何变化的。

ECMAScript 是由 [Ecma International](https://www.ecma-international.org/ "Ecma International") 这个组织标准化的一种规范。Ecma International 还在标准化 ECMAScript 以外的 C# 或 Dart 等语言。其中的技术委员会 39（TC39）是核心，负责讨论 ECMAScript 规范。这个技术委员会由 Microsoft、Mozilla、Google、Apple 等浏览器供应商和 ECMAScript 相关的企业组成。

## *ECMAScript 的版本历史*

*在这里，简单回顾一下 ECMAScript 的版本历史。

| 版本 | 发布日期 |
| --- | --- |
| 1 | 1997 年 6 月 |
| 2 | 1998 年 6 月 |
| 3 | 1999 年 12 月 |
| 4 | 废弃¹ |
| 5 | 2009 年 12 月 |
| 5.1 | 2011 年 6 月 |
| 2015 | 2015 年 6 月 |
| 2016 | 2016 年 6 月 |
| 2017 | 2017 年 6 月 |
| 以後毎年リリース |

从 ES5.1 到 ES2015 需要 4 年的时间，而 ES2015 以后每年都有发布。能够实现每年稳定发布，是因为 ES2015 以后进行了规范制定过程的变化。

## *成为 Living Standard 的 ECMAScript*

*目前，ECMAScript 规范草案由 GitHub 上的 [tc39/ecma262](https://github.com/tc39/ecma262 "tc39/ecma262: Status, process, and documents for ECMA262") 管理，并持续更新。因此，真正的最新 ECMAScript 规范是位于 `tc39.es/ecma262/`。这种不按版本号更新，而是始终发布最新版本的规范被称为 **Living Standard**。

ECMAScript 是 Living Standard，但除此之外，还公开了像 ECMAScript 2017 这样的带有版本号的规范。这种带版本号的 ECMAScript 是基于每年固定时间点的草案的快照。

在将 JavaScript 实际实现到浏览器等时，会参考 Living Standard 的 ECMAScript。这是因为浏览器本身也在不断更新，与只在固定时间点发布的带版本号的版本相比，Living Standard 更为合适。

## *规范制定过程*

*ES2015 以前，所有规范都需要经过长时间的讨论，直到达成一致，然后才能发布。因此，ES2015 的发布需要很长时间，语言的进化也停滞不前。为了解决这个问题，TC39 改变了 ECMAScript 的制定过程，改为每年发布。

这个制定过程是在 ES2015 发布后应用的，第一次应用这个过程的版本是 ES2016。从 ES2016 开始，通过以下规范制定过程进行讨论，以确定规范。²

仕様に追加する機能（API、構文など）をそれぞれ個別の**提案書**（提案书）として進めていきます。现在正在制定中的提案在 GitHub 上的 [tc39/proposals](https://github.com/tc39/proposals "tc39/proposals: Tracking ECMAScript Proposals") 中有列表公开。每个提案都有一个负责人称为 **冠军** 和一个称为 **阶段**（Stage）的 `0` 到 `4` 的 5 个阶段的状态。

| **阶段** | **阶段概述** |
| --- | --- |
| 0 | 概念阶段 |
| 1 | 功能提案阶段 |
| 2 | 创建了功能规格草案的阶段 |
| 3 | 规范已完成，需要寻求浏览器实现和反馈的阶段 |
| 4 | 规范制定完成，存在两个以上实现，可以正式合并到 ECMAScript 的阶段 |

每隔两个月举行的 TC39 会议中，会讨论每个提案是否应该进入下一个阶段。这次会议的会议记录也公开在 GitHub 上的 [tc39/tc39-notes](https://github.com/tc39/notes "tc39/tc39-notes: TC39 Meeting Notes")。达到阶段 4 的提案将被合并到 [tc39/ecma262](https://github.com/tc39/ecma262 "tc39/ecma262: Status, process, and documents for ECMA262") 的草案版中。然后，在每年的固定时间，基于草案版发布 `ECMAScript 20XX`。

这个制定过程的变化对 ECMAScript 包含的功能形式也有影响。

例如，`class` 结构的制定是以 **最大限地最小化**（maximally minimal classes）的形式提出的。因此，ES2015 中引入了 `class` 结构，但只以能够达成共识的最低限的功能状态出现。其他类功能则以其他提案的形式提出，并在 ES2015 之后以延期讨论的形式进行。

在推进提案时，达成最低程度的共识，是因为 ES4 的痛苦失败作为背景。在 ES4 中，试图对 ECMAScript 进行了大量更改，但在 TC39 内部意见分歧，最终未能达成一致。这导致投入 ES4 制定的几年时间成为了浪费。³

ES2016 以降的制定过程中，并不是所有提案都会被纳入规范。⁴ 当出现其他替代提案或无法保持向后兼容性时，可能会中断提案的制定。但即便如此，由于提案作为一个单位，制定工作的浪费也将被最小化。这种模块化的提案也易于替换。

## *尝试提案的功能*

*ECMAScript 的制定过程中，阶段 4 有一项标准是“存在两个以上的实现”。因此，浏览器的 JavaScript 引擎可能已经实现了正在制定中的提案。在大多数情况下，这些实现是通过实验性的标志来实现的，可以通过启用该标志来尝试使用。

此外，通过 Transpiler 和 Polyfill 等手段，可能可以模拟提案的功能。

Transpiler 是指一种工具，可以通过使用现有功能来模拟新语法。例如，在 ES2015 中引入了`class`语法，但在 ES5 中，`class`是保留字，因此会导致语法错误并且无法执行。Transpiler 将包含`class`语法的源代码转换为使用`function`关键字来伪造的代码。一些著名的 Transpiler 有[Babel](https://babeljs.io/ "Babel · The compiler for writing next generation JavaScript")和[TypeScript](https://www.typescriptlang.org/ "TypeScript - JavaScript that scales.")。

Polyfill 是一种提供新功能或方法等规范实现的库。例如，ES2016 引入了 Array 的`includes`方法。与语法不同，像`includes`这样的方法可以通过重写内置对象来实现。提供 Polyfill 的一些库包括[core-js](https://github.com/zloirock/core-js "zloirock/core-js: Standard Library")和[polyfill.io](https://polyfill.io/v3/ "Polyfill service")。

请注意，Transpiler 和 Polyfill 只是试图使用现有功能来复制新功能的工具。因此，无法使用 Transpiler 或 Polyfill 来复制无法使用现有功能复制的提案（功能）。此外，由于可能无法完全复制，因此不应该使用 Transpiler 或 Polyfill 来学习新功能。

## *了解规范及其制定过程的意义*

*知道 ECMAScript 这种规范及其制定过程的意义是什么？主要有以下几个原因。

+   为了学习语言

+   因为语言正在不断发展

+   为了查找正确的信息状态

### *学习语言*

*最简单的原因是为了学习 JavaScript 这种语言本身。如果想要了解语言的细节，可以参考 ECMAScript 规范。

但是，关于 JavaScript 语言功能，有优秀的参考文档网站，比如[MDN Web Docs](https://developer.mozilla.org/ja/ "MDN Web Docs")。因此，在想要学习如何使用时，可能不太需要参考 ECMAScript 规范本身。

### *因为语言正在不断发展*

*ECMAScript 是一种 Living Standard，每天都在更新。这意味着语言规范正在不断添加新功能和修复。

ECMAScript 尊重向后兼容性，因此你所学的东西并不会白费。但是意识到语言本身也在发展是很重要的。

ECMAScript 的提案（功能）是为了解决问题而提出的。当提案被合并到 ECMAScript 中并可用时，了解它解决了什么问题是很重要的。在这种情况下，了解 ECMAScript 的制定过程将是有益的。

当你想要了解这个规范为什么会变成现在这样时，拥有查找功能是很重要的。特别是自 ES2015 以来，制定过程变得更加开放，使用 GitHub，并且历史记录也更容易找到。

### *为了查找正确的信息状态*

*JavaScript 是一种广泛使用的语言，因此世界上存在大量的信息。然而，通过搜索找到的信息中可能包含正确的和错误的信息。

在其中，关于 ECMAScript 规范及其制定中的提案的信息是明确的。基本上，ECMAScript 规范的内容为了保持向后兼容性，几乎不会有破坏性的变化。提案有着明确的阶段状态，如果阶段低于 4，则说明它尚未稳定。

因此，当发现问题时，重要的是要查看相关规范或提案。

这不仅适用于 ECMAScript，对于 Web 和浏览器的信息也同样适用。关于浏览器，存在着 HTML、DOM API、CSS 等开放规范及其各自的制定过程。

### *总结*

*JavaScript 涵盖了 ECMAScript、Web 浏览器、Node.js、WebAssembly、WebGL、WebRTC 等广泛领域。 没有必要了解所有内容，甚至可能没有人了解所有内容。 在这种情况下，比起知识本身，更重要的是拥有在想了解某事时如何查找的方法。

任何事物都不会突然出现全新的概念，事物都有一个过程。 在 ECMAScript 中，制定过程以制定提案的形式公开。 换句话说，并非直接向规范中添加新功能，而是经历了提案阶段。

在不断变化的软件领域，拥有适当的查找方法至关重要。

> ¹. ECMAScript 4 包含了复杂且重大的更改，由于无法达成一致，规范被废弃。 ↩
> 
> ². 该制定过程的详细信息可在[`tc39.es/process-document/`](https://tc39.es/process-document/)找到。 ↩
> 
> ³. Allen Wirfs-Brock 先生撰写的[编程语言标准化](http://wirfs-brock.com/allen/files/papers/standpats-asianplop2016.pdf)详细介绍了 ES2015 的规范制定者。 ↩
> 
> ⁴. [已废弃的提案](https://github.com/tc39/proposals/blob/main/inactive-proposals.md)中列出了已中止制定的提案。 ↩*********
