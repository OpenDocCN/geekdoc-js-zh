# 基础知识

# 基础知识

![开始吧](img/5724c886.jpg)

在本节中，我将尝试为你介绍一个基本的 Webpack 工作流程的初始化和配置。

+   入门

# 入门

# 入门

你可以在 [Github](https://github.com/alexandrebodin/webpack-book-get-started) 上找到此示例的源代码。

## 安装

首先全局安装 Webpack。

```
npm install -g webpack 
```

接下来创建一个文件夹并初始化你的 npm 项目

```
mkdir project_dir && cd project_dir
npm init 
```

回答问题（如果你不打算发布你的代码，可以一直按回车直到完成）

## 设置

现在让我们创建一些文件。

```
project_dir/
-- index.html
-- index.js
-- webpack.config.js 
```

+   `webpack.config.js` 是 Webpack 配置文件的默认名称。你可以只使用此文件来设置完整的项目工作流程。

在你的 `index.html` 中，我们加载 Webpack 将为你创建的文件。

```
<script src="bundle  And package them into one or multiple files that you can just load into your html without ever worring about loading order and duplication.").js"></script> 
```

在你的 `index.js` 中说 "Hi"。

```
console.log('Hello World!'); 
```

在你的 `webpack.config.js` 中，我们将初始化应用程序配置。

```
module.exports = {
  entry: './index.js',
  output: {
    filename: 'bundle  And package them into one or multiple files that you can just load into your html without ever worring about loading order and duplication.").js'
  }
}; 
```

配置告诉 Webpack 它需要处理的文件是 `index.js`，并且它必须输出一个 `build` 目录中的文件 `bundle.js`。

让我们试着运行 Webpack 吧！

```
webpack 
```

这里，Webpack 会自动查找 `webpack.config.js` 文件。你可以使用 `'--config=PATH_TO_CONFIG'` 指定另一个文件。

现在你的项目应该是这样的（忽略 npm 的东西）。

```
project_dir/
-- bundle.js
-- index.js
-- index.html
-- webpack.config.js 
```

在一些浏览器中，你无法加载本地 Javascript 文件，这就是 `webpack-dev-server` 的作用所在。

除了其他一些令人惊叹的功能外，`webpack-dev-server` 还可以处理你的文件并创建一个 HTTP 服务器来为它们提供服务。

让我们在你的项目中安装它

```
npm install --save-dev webpack-dev-server 
```

现在运行

```
./node_modules/.bin/webpack-dev-server 
```

转到 `http://localhost:8080`。你应该在你的 `console` 中看到 `Hello World!`。

你会在你的 `terminal` 中注意到 Webpack 仍在运行。如果你改变你的 `index.js` 文件，它将立即重新处理。然后你只需重新加载你的浏览器。

Webpack 提供了自动页面重新加载等功能，多亏了插件和加载器。

## 一些代码

这很好，但现在你有一个空的 Javascript 文件。让我们添加一些代码。

创建一个名为 `app` 的文件夹，并在其中创建一个名为 `my_module.js` 的文件。

### `my_module.js`

```
module.exports = {
  sayHi: function(firstname) {
    console.log('Hi ' + firstname);
  }
}; 
```

### `index.js`

```
var myModule = require('./app/my_module');

myModule.sayHi('Webpack'); 
```

如果你保持 `webpack-dev-server` 运行，请刷新你的浏览器或重新启动 `webpack-dev-server`。

![魔法](img/e5257492)

你可以在 `console` 中看到 'Hi Webpack'。

总结一下，Webpack 可以像[NodeJs](https://nodejs.org/docs/latest/api/modules.html)一样加载模块，并将它们编译成一个文件。现在你可以使用开源库并更好地组织你的代码。

## Npm 糖

之前，你需要使用 `./node_modules/.bin/webpack-dev-server` 而不是 `webpack-dev-server`，因为它没有全局安装。

让我们设置一个 `npm` 脚本来简化这个过程。将这个添加到你的 `package.json` 中的 `"scripts"`。

```
{
  // ...
  "scripts": {
    "dev": "webpack-dev-server"
  }
} 
```

现在你只需要运行

```
npm run dev 
```

当你在 `package.json` 脚本中使用模块时，Npm 首先会在项目的 `./node_modules` 文件夹中查找，因此你不需要手动指定文件夹。

# 配置

# 配置

# 使用加载器

# 使用加载器

# 使用插件

# 使用插件

# 设置一个 ReactJS 应用程序

# 设置一个 ReactJS 应用程序
