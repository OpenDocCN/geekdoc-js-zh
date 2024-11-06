# 介绍

# 介绍

+   什么是 webpack

+   安装

# 什么是 Webpack

# 什么是 Webpack

Webpack 是 JavaScript 生态系统的模块打包工具。它的主要目标是捆绑你的应用程序代码和依赖项，以在浏览器中使用。

## 它真正做了什么？

假设你正在使用像[Boostrap](http://getbootstrap.com/)这样的库，并且你需要在你的应用程序中使用它。一个天真的方法是：

```
<!DOCTYPE html>
<html>
  <head>
  <link href="css/bootstrap.min.css" rel="stylesheet">
  </head>
  <body>
    <!-- You would need to load jQuery first because bootstrap requires it  -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.3/jquery.min.js"></script>
    <script src="js/bootstrap.min.js"></script>
  </body>
</html> 
```

当你的应用程序开始增长时，你会意识到处理所有依赖项是多么麻烦。

现在，想象一下能够直接将你需要的 Javascript 库`require`到你的`.js`文件中，就像这样：

```
var boostrap = require('boostrap');

// ... your code depending on the lib you just loaded 
```

然后只需这样做：

```
<!DOCTYPE html>
  <head>
  </head>
  <body>
    <!-- That's where the webpack magic happens -->
    <script src="js/bundle  And package them into one or multiple files that you can just load into your html without ever worring about loading order and duplication.").js"></script>
  </body>
</html> 
```

这就是 Webpack 的全部内容。它负责加载和创建最终的`js`文件，以便导入到你的`index.html`中。

# 安装

# 安装

## Webpack

首先全局安装 Webpack

```
npm install -g webpack 
```

开发时，将 Webpack 作为项目的依赖是一个好的做法。

```
npm install --save-dev webpack 
```

## WebpackDevServer

Webpack 带有一个用于开发的包。

它允许你实时构建和提供你的文件，当你更新文件时自动重建，还有一些其他很棒的功能！

继续在项目中安装它。

```
npm install --save-dev webpack-dev-server 
```
