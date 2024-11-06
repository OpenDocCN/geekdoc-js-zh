# 付録: 参考链接集

> 原文：[`jsprimer.net/appendix/links/`](https://jsprimer.net/appendix/links/)

这里介绍了一些未在正文中涉及的 JavaScript 周边工具和库。 由于这些信息容易受时代影响而变得过时，因此将其作为独立的附录整理在一起。

## [](#developer-tools)*开发辅助工具*

*介绍一些有助于使用 JavaScript 开发的工具。

### [](#transpiler)*转译器*

*转译器是一种工具，用于将源代码转换为源代码。 在这里，我们介绍了用于转换 JavaScript 源代码的转译器，这在应用程序开发中经常使用。

#### [](#babel)*Babel*

*[Babel](https://babeljs.io/)是一个以将 ECMAScript 新语法转换为旧 ECMAScript 语法为主要功能的转译器。 例如，像 Internet Explorer 这样的旧浏览器不支持 ECMAScript 2015，因此无法解析新语法。 通过使用 Babel 将 ES2015 语法转换为 ES5 以模拟的代码，即使在只支持旧语法的执行环境中也可以运行。

此外，正如“ECMAScript”章节中所介绍的，它也被用作尝试最新提案功能的工具。

#### [](#typescript)*TypeScript*

*[TypeScript](https://www.typescriptlang.org/)是一种添加了静态类型注解语法的语言和转译器。 TypeScript 语言提供了类型注解等语法，可以为 JavaScript 代码添加类型注解以进行静态类型检查。 此外，TypeScript 代码可以转换为删除与类型相关的语法等内容的 JavaScript 代码。

### [](#module-bundler)*模块打包工具*

*模块打包工具是一种解决 JavaScript 模块依赖关系并将多个模块合并为一个 JavaScript 文件的工具。 模块打包工具会遍历起始模块依赖的模块，并按适当的顺序合并（打包）它们。

通过 NPM，许多 JavaScript 库都针对 Node.js 进行分发，几乎所有这些库都是 CommonJS 模块。 由于 CommonJS 模块是为 Node.js 执行而设计的，因此不能直接在 Web 浏览器上运行。 因此，出现了一种称为模块打包工具的工具，用于解决 CommonJS 模块在浏览器中运行的问题。

#### [](#webpack)*webpack*

*[webpack](https://webpack.js.org/)是一个适用于创建 JavaScript 应用程序的模块打包工具。 webpack 解决了 CommonJS 模块和 ECMAScript 模块的依赖关系，并在优化应用程序的同时合并模块。 除了 JavaScript 模块外，还提供了加载图像和 CSS 等资源的机制，可以处理各种类型的文件。 此外，webpack 提供了通过插件进行扩展的机制，实现灵活的转换。

#### [](#rollup)*Rollup*

*[Rollup](https://rollupjs.org/)是一个适用于创建 JavaScript 库的模块打包工具。 Rollup 主要处理 ECMAScript 模块，解决模块依赖关系，并在优化库的同时合并模块。 可以灵活配置生成多种格式文件，如 CommonJS 格式和 ECMAScript 模块格式等，非常适合创建库。 Rollup 也可以像 webpack 一样通过插件进行扩展，实现灵活的转换。

### [](#coding-style)*编码风格的统一*

*在团队开发中，统一源代码的格式，如换行位置和缩进宽度。 因为不同风格的代码会增加审查的时间。 此外，防止混入不应该使用的旧习语或容易产生错误的危险代码，也是保持质量的重要因素。

统一这些编码风格非常重要，需要持续保持一致性。 因此，建议使用工具进行自动化。

#### [](#prettier)*Prettier*

*[Prettier](https://prettier.io/)是一个通用的代码格式化工具，适用于 JavaScript 等多种语言。 由于无需配置文件即可使用，因此非常易于引入。

#### [](#eslint)*ESLint*

*[ESLint](https://eslint.org/)是用于 JavaScript 文件的 Lint 工具。 *Lint* 是指通过静态分析源代码文件来检测不合适的代码或不符合编码风格的代码的机制。 使用 Lint 可以机械地统一团队内的编码风格。

### [](#code-editor)*代码编辑器*

*选择适合 JavaScript、HTML、CSS 等编码的编辑器可以提高开发生产力。

#### [](#vscode)*VSCode*

*[VSCode](https://code.visualstudio.com/)是由 Microsoft 开发的免费开源代码编辑器。 可以使用 JavaScript 编写插件，以添加各种功能。

### [](#browser-devtools)*浏览器开发者工具*

*许多浏览器提供了开发者工具，本文介绍的控制台也是其中之一。 此外，不同浏览器提供各种功能，如能够逐步执行 JavaScript 代码的调试器，以及 HTTP 通信日志等。

+   Firefox: [Firefox DevTools User Docs](https://firefox-source-docs.mozilla.org/devtools-user/index.html)

+   Chrome: [Chrome DevTools](https://developer.chrome.com/docs/devtools/)

+   Safari: [Safari Developer Help](https://support.apple.com/ja-jp/guide/safari-developer/welcome/mac)

### [](#performance-improvement)*性能改善*

*ウェブサイトやウェブアプリケーションのパフォーマンスを計測、改善するためのツールを紹介します。

#### [](#pagespeed)*PageSpeed Insights*

*[PageSpeed Insights](https://pagespeed.web.dev/)是 Google 提供的网页性能测量工具。 输入要测量的页面 URL 后，它会显示加载时间以及可以改进的项目。

#### [](#webpagetest)*WebPagetest*

*[WebPagetest](https://www.webpagetest.org/)是一个基于浏览器的网页性能测试工具。 它可以在各种条件下使用浏览器访问网站并测量性能。 它是在 BSD 许可下开源的，您也可以将其安装在任何服务器上运行。

#### [](#lighthouse)*Lighthouse*

*[Lighthouse](https://developer.chrome.com/docs/lighthouse/overview/)是 Google 提供的网页分析工具。 它不仅分析网页性能，还从可访问性和 SEO 等方面进行分析，并显示分数。 它已经集成到 Chrome 浏览器的开发者工具中，但也可以通过 npm 安装为 CLI 运行。

## [](#javascript-platform)*JavaScript 的执行平台*

*JavaScript 不仅仅是用于构建网站的语言。 如今，JavaScript 及其周边生态系统已经发展成为跨越多个平台的通用语言。 我们将介绍一些用于执行 JavaScript 程序的平台。

### [](#publishing-website)*发布网站*

*要将网站或 Web 应用程序发布到互联网上，您需要在某个 Web 服务器上托管（发布）它。 这里我们介绍一些提供托管服务的 *托管服务*，可以轻松地发布网站。

#### [](#github-pages)*GitHub Pages*

*[GitHub Pages](https://pages.github.com/)是 GitHub 提供的免费托管服务。 可以将 GitHub 存储库发布为网页，并传送存储库中的 HTML、CSS、JavaScript 等静态文件。

#### [](#firebase-hosting)*Firebase Hosting*

*[Firebase Hosting](https://firebase.google.com/products/hosting/?hl=ja)是 Google 的 Firebase 平台提供的托管服务。 使用 CLI 进行简单部署，并且对于小规模使用是免费的特点。

#### [](#netlify)*Netlify*

*[Netlify](https://www.netlify.com/)��是一种免费的托管服务。 它与类似 GitHub 和 BitBucket 的 Git 存储库服务集成，只需将代码推送到远程存储库，即可自动部署。

### [](#serverless)*在 Node.js 上无服务器执行*

*有一些被称为[函数即服务](https://en.wikipedia.org/wiki/Function_as_a_service)（FaaS）的执行平台，如 AWS Lambda 和 Google Cloud Functions。 在 FaaS 中，您可以在无需准备 Node.js 服务器的情况下执行 Node.js 脚本。 将 JavaScript 函数部署到 FaaS 后，它们将托管在云上的 Node.js 服务器上，并为每个函数分配一个端点。

#### [](#aws-lambda)*AWS Lambda*

*[AWS Lambda](https://aws.amazon.com/jp/lambda/)是在 Amazon Web Services 上提供的无服务器 Node.js 执行环境。

#### [](#google-cloud-functions)*Google Cloud Functions*

*[Google Cloud Functions](https://cloud.google.com/functions/)是在 Google Cloud Platform 上提供的无服务器 Node.js 执行环境。

### [](#desktop-app)*创建桌面应用程序*

*您也可以使用 JavaScript 在 Windows、macOS、Linux 等桌面环境中创建 GUI 应用程序。

#### [](#electron)*Electron*

*[Electron](https://www.electronjs.org/)是由 GitHub 开发的开源桌面应用程序框架。 您可以使用 HTML、CSS 和 JavaScript 打包 Web 应用程序，并创建可分发的可执行文件。

#### [](#nwjs)*NW.js*

*[NW.js](https://nwjs.io/)是由英特尔公司开发的开源桌面应用程序框架。与 Electron 类似，可以开发基于 Chromium 浏览器的应用程序。 NW.js 的特点是可以直接从浏览器中利用 Node.js 的开发生态系统。
