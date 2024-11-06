# ECMAScript

> 原文：[`jsprimer.net/basic/ecmascript/`](https://jsprimer.net/basic/ecmascript/)

ここまでJavaScriptの基本文法について見てきましたが、その文法を定めるECMAScriptという仕様自体がどのように変化していくのかを見ていきましょう。

ECMAScriptは[Ecma International](https://www.ecma-international.org/ "Ecma International")という団体によって標準化されている仕様です。 Ecma InternationalはECMAScript 以外にもC#やDartなどの標準化作業をしています。 Ecma Internationalの中のTechnical Committee 39（TC39）という技術委員会が中心となって、ECMAScript 仕様について議論しています。 この技術委員会はMicrosoft、Mozilla、Google、AppleといったブラウザベンダーやECMAScriptに関心のある企業などによって構成されます。

## [](#history)*ECMAScriptのバージョンの歴史*

*ここで、簡単にECMAScriptのバージョンの歴史を振り返ってみましょう。

| バージョン | リリース時期 |
| --- | --- |
| 1 | 1997 年 6 月 |
| 2 | 1998 年 6 月 |
| 3 | 1999 年 12 月 |
| 4 | 破棄^(1) |
| 5 | 2009 年 12 月 |
| 5.1 | 2011 年 6 月 |
| 2015 | 2015 年 6 月 |
| 2016 | 2016 年 6 月 |
| 2017 | 2017 年 6 月 |
| 以降毎年リリース |

ES5.1からES2015がでるまで4 年もの歳月が��かっているのに対して、ES2015 以降は毎年リリースされています。 毎年安定したリリースを行えるようになったのは、ES2015 以降に仕様策定プロセスの変更が行われたためです。

## [](#living-standard)*Living StandardとなるECMAScript*

*現在、ECMAScriptの仕様書のドラフトはGitHub 上の[tc39/ecma262](https://github.com/tc39/ecma262 "tc39/ecma262: Status, process, and documents for ECMA262")で管理されており、日々更新されています。 そのため、本当の意味での最新のECMAScript 仕様は[`tc39.es/ecma262/`](https://tc39.es/ecma262/)となります。 このように更新ごとにバージョン番号をつけずに、常に最新版を公開する仕様のことを**Living Standard**と呼びます。

ECMAScriptはLiving Standardですが、これに加えてECMAScript 2017のようにバージョン番号をつけたものも公開されています。 このバージョンつきECMAScriptは、毎年決まった時期のドラフトを元にしたスナップショットのようなものです。

ブラウザなどに実際にJavaScriptとして実装される際には、Living StandardのECMAScriptを参照しています。 これは、ブラウザ自体も日々更新されるものであり、決まった時期にしかリリースされないバージョンつきよりもLiving Standardのほうが適切であるためです。

## [](#specification-process)*仕様策定のプロセス*

*ES2015 以前はすべての仕様の合意が取れるまで延々と議論を続け、すべてが決まってからリリースされていました。 そのため、ES2015がリリースされるまでには長い時間がかかり、言語の進化が停滞していました。 この問題を解消するために、TC39は毎年リリースする形へとECMAScriptの策定プロセスを変更しました。

この策定プロセスはES2015のリリース後に適用され、このプロセスで初めてリリースされたのがES2016となります。 ES2016 以降では、次のような仕様策定のプロセスで議論を進めて仕様が決定されています。^(2)

仕様に追加する機能（API、構文など）をそれぞれ個別の**プロポーザル**（提案書）として進めていきます。 現在策定中のプロポーザルはGitHub 上の[tc39/proposals](https://github.com/tc39/proposals "tc39/proposals: Tracking ECMAScript Proposals")に一覧が公開されています。 それぞれのプロポーザルは責任者である**チャンピオン**と**ステージ**（Stage）と呼ばれる`0`から`4`の5 段階の状態を持ちます。

| **ステージ** | **ステージの概要** |
| --- | --- |
| 0 | アイデアの段階 |
| 1 | 機能提案の段階 |
| 2 | 機能の仕様書ドラフトを作成した段階 |
| 3 | 仕様としては完成しており、ブラウザの実装やフィードバックを求める段階 |
| 4 | 仕様策定が完了し、2つ以上の実装が存在している 正式にECMAScriptにマージできる段階 |

2ヶ月に一度行われるTC39のミーティングにおいて、プロポーザルごとにステージを進めるかどうかを議論します。 このミーティングの議事録もGitHub 上の[tc39/tc39-notes](https://github.com/tc39/notes "tc39/tc39-notes: TC39 Meeting Notes")にて公開されていま��。 ステージ4となったプロポーザルはドラフト版である[tc39/ecma262](https://github.com/tc39/ecma262 "tc39/ecma262: Status, process, and documents for ECMA262")へマージされます。 そして毎年の決まった時期にドラフト版を元にして`ECMAScript 20XX`としてリリースします。

この仕様策定プロセスの変更は、ECMAScriptに含まれる機能の形にも影響しています。

たとえば、`class`構文の策定は**最大限に最小のクラス**（maximally minimal classes）と呼ばれる形で提案されています。 これによりES2015で`class`構文が導入されましたが、クラスとして合意が取れる最低限の機能だけの状態で入りました。 その他のクラスの機能は別のプロポーザルとして提案され、ES2015 以降に持ち越された形で議論が進められています。

在推进提案时，达成最低程度的共识，是因为 ES4 的痛苦失败作为背景。在 ES4 中，试图对 ECMAScript 进行了大量更改，但在 TC39 内部意见分歧，最终未能达成一致。这导致投入 ES4 制定的几年时间成为了浪费。^(3)

ES2016 以降的制定过程中，并不是所有提案都会被纳入规范。^(4) 当出现其他替代提案或无法保持向后兼容性时，可能会中断提案的制定。但即便如此，由于提案作为一个单位，制定工作的浪费也将被最小化。这种模块化的提案也易于替换。

## [](#try-proposal)*尝试提案的功能*

*ECMAScript 的制定过程中，阶段 4 有一项标准是“存在两个以上的实现”。因此，浏览器的 JavaScript 引擎可能已经实现了正在制定中的提案。在大多数情况下，这些实现是通过实验性的标志来实现的，可以通过启用该标志来尝试使用。

此外，通过 Transpiler 和 Polyfill 等手段，可能可以模拟提案的功能。

Transpiler 是指一种工具，可以通过使用现有功能来模拟新语法。例如，在 ES2015 中引入了`class`语法，但在 ES5 中，`class`是保留字，因此会导致语法错误并且无法执行。Transpiler 将包含`class`语法的源代码转换为使用`function`关键字来伪造的代码。一些著名的 Transpiler 有[Babel](https://babeljs.io/ "Babel · The compiler for writing next generation JavaScript")和[TypeScript](https://www.typescriptlang.org/ "TypeScript - JavaScript that scales.")。

Polyfill 是一种提供新功能或方法等规范实现的库。例如，ES2016 引入了 Array 的`includes`方法。与语法不同，像`includes`这样的方法可以通过重写内置对象来实现。提供 Polyfill 的一些库包括[core-js](https://github.com/zloirock/core-js "zloirock/core-js: Standard Library")和[polyfill.io](https://polyfill.io/v3/ "Polyfill service")。

请注意，Transpiler 和 Polyfill 只是试图使用现有功能来复制新功能的工具。因此，无法使用 Transpiler 或 Polyfill 来复制无法使用现有功能复制的提案（功能）。此外，由于可能无法完全复制，因此不应该使用 Transpiler 或 Polyfill 来学习新功能。

## [](#meaning-specification-process)*了解规范及其制定过程的意义*

*知道 ECMAScript 这种规范及其制定过程的意义是什么？主要有以下几个原因。

+   为了学习语言

+   因为语言正在不断发展

+   为了查找正确的信息状态

### [](#to-learn)*学习语言*

*最简单的原因是为了学习 JavaScript 这种语言本身。如果想要了解语言的细节，可以参考 ECMAScript 规范。

但是，关于 JavaScript 语言功能，有优秀的参考文档网站，比如[MDN Web Docs](https://developer.mozilla.org/ja/ "MDN Web Docs")。因此，在想要学习如何使用时，可能不太需要参考 ECMAScript 规范本身。

### [](#to-progress)*因为语言正在不断发展*

*ECMAScript 是一种 Living Standard，每天都在更新。这意味着语言规范正在不断添加新功能和修复。

ECMAScript 尊重向后兼容性，因此你所学的东西并不会白费。但是意识到语言本身也在发展是很重要的。

ECMAScript 的提案（功能）是为了解决问题而提出的。当提案被合并到 ECMAScript 中并可用时，了解它解决了什么问题是很重要的。在这种情况下，了解 ECMAScript 的制定过程将是有益的。

当你想要了解这个规范为什么会变成现在这样时，拥有查找功能是很重要的。特别是自 ES2015 以来，制定过程变得更加开放，使用 GitHub，并且历史记录也更容易找到。

### [](#to-search)*为了查找正确的信息状态*

*JavaScript 是一种广泛使用的语言，因此世界上存在大量的信息。然而，通过搜索找到的信息中可能包含正确的和错误的信息。

在其中，关于 ECMAScript 规范及其制定中的提案的信息是明确的。基本上，ECMAScript 规范的内容为了保持向后兼容性，几乎不会有破坏性的变化。提案有着明确的阶段状态，如果阶段低于 4，则说明它尚未稳定。

因此，当发现问题时，重要的是要查看相关规范或提案。

这不仅适用于 ECMAScript，对于 Web 和浏览器的信息也同样适用。关于浏览器，存在着 HTML、DOM API、CSS 等开放规范及其各自的制定过程。

### [](#ecmascript-summary)*总结*

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
