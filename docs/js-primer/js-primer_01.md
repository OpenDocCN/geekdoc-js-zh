# 引言

> 原文：[`jsprimer.net/intro/`](https://jsprimer.net/intro/)

## [](#do)*本书的目的*

*本书的目的是学习 JavaScript 这门编程语言。 通过从头开始阅读，您可以学习 JavaScript 的语法和功能。

学习 JavaScript 的语法等写法也很重要，但了解实际使用方法也是目的。 这是因为，要阅读或编写 JavaScript 代码，仅仅了解语法是不够的。 因此，在第一部分：基本语法中，我们不仅涉及语法，还提到了实际用法；在第二部分：用例中，我们通过小型应用程序示例介绍了实际用法。

JavaScript 是一种不断变化的语言，语言本身以及周围的开发环境也在不断变化。 本书的目的是为了掌握能够适应围绕 JavaScript 的变化的基础知识。 因此，我们将重点放在为什么不起作用以及如何解决问题上，而不仅仅是学习语法。

## [](#do-not)*本书的目的不在于*

*无法通过一本书学会 JavaScript 的所有内容。 这是因为 JavaScript 的应用范围非常广泛。 因此，我们将明确列出本书不涉及的内容（非目的）。

+   本书不旨在与其他编程语言进行比较

+   本书不旨在学习 Web 浏览器

+   不是为了学习 Node.js

+   JavaScript 的所有语法和功能并非本书的目的

+   本书不旨在成为 JavaScript 的参考资料

+   本书的目标不是学习 JavaScript 库或框架的使用方法

+   阅读本书并不意味着您可以立即创造出什么。

本书的目的不是涵盖所有语法和功能，就像参考[MDN Web Docs](https://developer.mozilla.org/ja/)（MDN）一样，JavaScript 和浏览器的 API 已经有了很好的参考资料。

本书不旨在教授库的使用方法或特定应用程序的创建方法。 我们建议您从库的文档或现有应用程序中学习这些内容。 当然，您也可以阅读有关库或应用程序的其他书籍。

本书将帮助您理解那些库和应用程序运行所需的机制。 精心设计的库和应用程序看起来可能像魔法一样。 实际上，它们是在某种机制的基础上构建的，然后作为库或应用程序运行。

虽然不会详细解释具体机制，但会帮助您意识到这些机制的存在并理解。

## [](#who-read)*谁应该阅读本书*

*本书旨在让有编程经验的人学习 JavaScript 这门语言。 因此，对于第一次学习编程语言的人来说，可能会有一些困难。 但是，由于书中实际上是通过运行程序来学习的，因此即使是编程新手也可以尝试挑战。

即使您已经写过 JavaScript，但对最近的 JavaScript 不太了解的人也是本书的读者对象。 2015 年，JavaScript 经历了名为 ECMAScript 2015 的规范的重大变化。 本书是基于 ECMAScript 2015 的 JavaScript 入门书籍，并在必要时介绍了与以前写法的不同之处。 因此，即使对新的写法或不了解与以前有何不同的情况，本书也会有所帮助。

本书认真对待 JavaScript 的规范。 尽管这是一本入门书，但我们避免极端简化和不准确的内容。 因此，即使是 JavaScript 专家，阅读本书也应该有所发现。

## [](#features)*本书的特点*

*简要介绍本书的特点*

当 ECMAScript 2015 规范进行了重大更新时，JavaScript 的写法和功能大大增加。 这使得 JavaScript 看起来与以往的语言有很大不同。

本书是基于更新后的 ECMAScript 2015 编写的。 如果要学习 JavaScript，最好以更新后的 ECMAScript 2015 为前提，因为这样更容易学习。 本书基于 ECMAScript 2015，同时覆盖了截至目前的最新版本 ECMAScript 2023。

此外，现代 Web 浏览器支持 ECMAScript 2015。 因此，在学习过程中，本书可能不会介绍旧的写法，因为这些写法可能已经不再需要。 但是，对于阅读现有代码时，了解旧的写法仍然是必要的，因此我们会介绍常见情况。

但是，本书没有涉及即将出现的 JavaScript 新功能。 这是因为这些是未来的事情，因此有很多不确定性，而且无法预测实际的使用方式。 本书的目的是在学习基础知识的同时不要偏离实际用例。

本书的文本和源代码作为开源项目在 GitHub 的[asciidwango/js-primer](https://github.com/asciidwango/js-primer)上公开。 此外，由于书籍内容在[jsprimer.net](https://jsprimer.net/)上公开，因此可以在 Web 浏览器中阅读。 在 Web 版本中，您可以即时运行示例代码并��习 JavaScript。

由于书籍内容在 Web 上公开，因此在需要共享书籍内容时可以提供 URL。 此外，书籍内容和示例代码在以下许可范围内可自由使用。

## [](#license)*许可*

*本书中的所有源代码均以[MIT 许可证](https://github.com/asciidwango/js-primer/blob/master/LICENSE-MIT)提供为基础的开源软件。 此外，本书的文本以[知识共享署名 4.0](https://github.com/asciidwango/js-primer/blob/master/LICENSE-CC-BY)（CC BY 4.0）许可证提供。 只要遵循版权声明，您就可以在一定程度上自由使用这两种许可证。

有关详细信息，请参阅[许可证信息](https://github.com/asciidwango/js-primer/blob/master/README.md#license)。 

## [](#print-version)*关于书籍版*

*这本书由[AsciiDwango](https://asciidwango.jp/)出版。 您可以从以下页面购买书籍版。

+   物理书籍和 Kindle: [JavaScript Primer 修订 2 版 迷你入门书 | azu, Suguru Inatomi |书 | 网购 | 亚马逊](https://www.amazon.co.jp/dp/4048931105/)

+   PDF 和 epub: [JavaScript Primer 迷你入门书【委托】 - 達人出版会](https://tatsu-zine.com/books/javascript-primer) （截至 2023 年 06 月 03 日仍为初版）

## [](#diff-with-print-version)*网页版和书籍版的区别*

*书籍版的内容与[JavaScript Primer v4.0.0](https://github.com/asciidwango/js-primer/releases/tag/v4.0.0-publish)相同。

网页版和书籍版有以下不同之处。

+   网页版和书籍版的布局不同

+   书籍版包含索引

+   网页版会持续每天略微更新，以保持最新的规范

书籍版的内容与网页版相同，但内容和布局经过优化，以便作为书籍阅读。 书籍版在出版时基本内容相同，但网页版始终保持更新。

这本书是按顺序阅读的。因此，作为阅读材料，书籍版更易于阅读。由于网页版内置了搜索功能和可执行示例代码的功能，因此请根据需要同时使用。

## [](#sponsors)*关于对书籍的支持*

*JavaScript Primer 会持续更新以跟进 ECMAScript 的更新，反映实际使用情况。 为了持续更新，我们随时欢迎对书籍的支持。

您可以在 GitHub Sponsors 上支持作者。

+   [在 GitHub Sponsors 上赞助@azu](https://github.com/sponsors/azu)

您可以通过 Open Collective 支持 jsprimer 项目。 Open Collective 的支持包括企业特权，例如在网站上显示徽标。

+   JavaScript Primer 赞助商

+   [JavaScript Primer - Open Collective](https://opencollective.com/jsprimer)

此外，撰写书籍评论也是一种支持。

+   [JavaScript Primer 修订 2 版 迷你入门书 | azu, Suguru Inatomi |书 | 网购 | 亚马逊](https://www.amazon.co.jp/dp/4048931105/)

回答 GitHub Discussions（论坛）中其他人的问题，或者写下对 JSPrimer 的感想也是一种支持。

+   [讨论](https://github.com/asciidwango/js-primer/discussions)

Discussions 的指南已整理在以下主题中。

+   [👋 欢迎来到 JavaScript Primer！ · 讨论 #1304](https://github.com/asciidwango/js-primer/discussions/1304)

您可以提���问题或发送 Pull Request 直接支持书籍。 有关 Issue 和 Pull Request，请参阅下一页。

+   [发现错误？请告诉我们 · JavaScript Primer #jsprimer](https://jsprimer.net/intro/feedback/)

## [](#thanks)*致谢*

*在初版中，我们征求了以下人员的审阅意见。

+   mizchi（竹马光太郎）

+   中西优介@better_than_i_w

+   @tsin1rou

+   sakito

+   川上和义

+   尾上洋介

在第 2 版中，我们征求了以下人员的审阅意见。

+   haruguchi（池奥 悠马）

+   2nofa11（ツノ）

+   staticWagomU（林 永远）

+   kakts（阿久津 恵太）

+   keisuke kudo（工藤佳祐）

+   r-shirasu

+   藤野慎也（morinokami）

+   kobakazu0429（小畠 一泰）

+   滝谷修

这本书能够变得更好，完全归功于大家的合作。

此外，这本书是在 GitHub 上公开的状态下编写的。因此，感谢所有为这本书做出贡献的人。

## [](#changelog)*变更点*

*网页版会持续更新以支持最新的 ECMAScript。 在支持每个新的 ECMAScript 版本时，我们会制作变更摘要的发布说明。

+   [v1.0.0: 初版发布](https://github.com/asciidwango/js-primer/releases/tag/1.0.0)

+   [v2.0.0: 支持 ECMAScript 2020](https://github.com/asciidwango/js-primer/releases/tag/v2.0.0)

+   [v3.0.0: ECMAScript 2021 兼容](https://github.com/asciidwango/js-primer/releases/tag/v3.0.0)

+   [v4.0.0: ECMAScript 2022 兼容](https://github.com/asciidwango/js-primer/releases/tag/v4.0.0)

+   [v5.0.0: ECMAScript 2023 兼容/更改为 CC BY 许可证/Open Collective](https://github.com/asciidwango/js-primer/releases/tag/v5.0.0)

当新版本发布时，希望收到通知的人，请[关注](https://github.com/asciidwango/js-primer/watchers)[GitHub 仓库](https://github.com/asciidwango/js-primer)。

![关注按钮](https://github.com/asciidwango/js-primer/watchers)

此外，您也可以通过下面的表单预先注册邮箱地址，以便通过邮件接收更新信息。

+   [注册接收更新通知的邮箱地址表单](https://us13.list-manage.com/subscribe?u=fc41e11a2b9dc6f05350e0de0&id=7ab1594ae8)**********
