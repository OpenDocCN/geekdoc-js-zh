# 显示数据

> 原文：[`jsprimer.net/use-case/ajaxapp/display/`](https://jsprimer.net/use-case/ajaxapp/display/)

在前面的章节中，我们使用 Fetch API 从 GitHub API 获取了用户信息。在本节中，我们将对获取的数据进行 HTML 格式化，并在应用程序中显示用户信息。

## *构建 HTML*

*在生成 HTML 字符串时使用模板字符串。模板字符串允许在字符串中包含换行，因此可以表示 HTML 的缩进，从而提高可读性。此外，它还允许轻松嵌入变量，因此非常适合动态地将数据应用于 HTML 模板。

下面的代码声明了从 GitHub 用户信息构建的 HTML 模板。

```
const view = `
<h4>${userInfo.name} (@${userInfo.login})</h4>
<img src="${userInfo.avatar_url}" alt="${userInfo.login}" height="100">
<dl>
    <dt>Location</dt>
    <dd>${userInfo.location}</dd>
    <dt>Repositories</dt>
    <dd>${userInfo.public_repos}</dd>
</dl>
`; 
```

将`userInfo`对象的值应用到这个模板中，将生成如下 HTML 字符串。

```
<h4>js-primer example (@js-primer-example)</h4>
<img src="https://github.com/js-primer-example.png" alt="js-primer-example" height="100">
<dl>
    <dt>Location</dt>
    <dd>Japan</dd>
    <dt>Repositories</dt>
    <dd>1</dd>
</dl> 
```

## *将 HTML 字符串添加到 DOM 中*

*接下来，将生成的 HTML 字符串添加到 DOM 树中并显示。首先，为了动态设置 HTML，需要在`index.html`中添加一个标记元素。这次，我们添加了一个具有`result` id 的 div 元素（以下称为`div#result`元素）。 

index.html

```
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <title>Ajax Example</title>
  </head>
  <body>
    <h2>GitHub User Info</h2>

    <button onclick="fetchUserInfo('js-primer-example');">Get user info</button>
    <!-- 整形したHTMLの挿入先 -->
    <div id="result"></div>

    <script src="index.js"></script>
  </body>
</html> 
```

从这里开始，将 HTML 字符串作为`div#result`元素的孩子插入。使用`document.getElementById`方法通过 id 属性访问设置了 id 的元素。如果成功获取到`div#result`元素，则将之前生成的 HTML 字符串设置为`innerHTML`属性。

```
const result = document.getElementById("result");
result.innerHTML = view; 
```

使用 JavaScript 将 HTML 元素添加到 DOM 中的方法主要有两种。一种是将 HTML 字符串设置为 Element 的`innerHTML`属性，就像这次这样。另一种是生成 Element 对象，然后[逐步构建树结构](https://developer.mozilla.org/ja/docs/Web/API/Node/appendChild)。后者在安全性方面更安全，但代码会更冗长。这次，我们将使用 Element 的`innerHTML`属性，同时进行安全性的转义处理。

## *转义 HTML 字符串*

*将字符串设置到 Element 的`innerHTML`属性中时，该字符串将被解释为 HTML。例如，如果 GitHub 的用户名中包含`<`或`>`符号，则可能会产生意外的 HTML 结构。为了避免这种情况，在设置字符串之前，需要将特定的符号替换为安全的表示。这个过程通常被称为 HTML 转义。

许多 View 库内部都包含转义机制，在动态构建 HTML 时，默认会进行转义。或者，也经常使用[HTML 转义库](https://github.com/teppeis/htmlspecialchars)。像这样进行独立实现的情况是特殊情况，通常几乎都是使用库提供的功能。

按如下方式，声明名为`escapeSpecialChars`的函数以进行特殊字符的转义处理。

```
function escapeSpecialChars(str) {
    return str
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
} 
```

在 HTML 字符串中，将`userInfo`中的值注入到所有需要调用`escapeSpecialChars`函数的位置。但是，在模板字符串中插入的所有部分都应用函数会比较麻烦，而且维护性也不好。因此，通过使用带标签的模板函数，可以避免显式调用转义函数。带标签的模板函数可以处理在模板值嵌入时调用的函数。

下面的`escapeHTML`函数是**标签函数**（详细信息请参考“字符串”章节中的“带标签的模板函数”）。标签函数接受一个字符串字面量数组作为第一个参数，一个要嵌入的值数组作为第二个参数。在`escapeHTML`函数中，在构建字符串时保持字符串字面量和值的原始顺序，如果值是字符串类型，则进行转义。

```
function escapeHTML(strings, ...values) {
    return strings.reduce((result, str, i) => {
        const value = values[i - 1];
        if (typeof value === "string") {
            return result + escapeSpecialChars(value) + str;
        } else {
            return result + String(value) + str;
        }
    });
} 
```

`escapeHTML`函数是标签函数，因此不是使用通常的`()`调用，而是使用标签对模板字符串进行标记后使用。在模板字符串的转义引号前写上函数，可以将函数作为标签函数调用。

```
const view = escapeHTML`
<h4>${userInfo.name} (@${userInfo.login})</h4>
<img src="${userInfo.avatar_url}" alt="${userInfo.login}" height="100">
<dl>
    <dt>Location</dt>
    <dd>${userInfo.location}</dd>
    <dt>Repositories</dt>
    <dd>${userInfo.public_repos}</dd>
</dl>
`;

const result = document.getElementById("result");
result.innerHTML = view; 
```

通过这种方式，我们已经完成了 HTML 字符串的生成和转义。在这些处理中，将调用前面创建的`fetchUserInfo`函数。到目前为止，`index.js`和`index.html`如下所示。

index.js

```
function fetchUserInfo(userId) {
    fetch(`https://api.github.com/users/${encodeURIComponent(userId)}`)
        .then(response => {
            if (!response.ok) {
                console.error("エラーレスポンス", response);
            } else {
                return response.json().then(userInfo => {
                    // HTMLの組み立て
                    const view = escapeHTML`
                    <h4>${userInfo.name} (@${userInfo.login})</h4>
                    <img src="${userInfo.avatar_url}" alt="${userInfo.login}" height="100">
                    <dl>
                        <dt>Location</dt>
                        <dd>${userInfo.location}</dd>
                        <dt>Repositories</dt>
                        <dd>${userInfo.public_repos}</dd>
                    </dl>
                    `;
                    // HTMLの挿入
                    const result = document.getElementById("result");
                    result.innerHTML = view;
                });
            }
        }).catch(error => {
            console.error(error);
        });
}

function escapeSpecialChars(str) {
    return str
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#039;");
}

function escapeHTML(strings, ...values) {
    return strings.reduce((result, str, i) => {
        const value = values[i - 1];
        if (typeof value === "string") {
            return result + escapeSpecialChars(value) + str;
        } else {
            return result + String(value) + str;
        }
    });
} 
```

index.html

```
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="utf-8" />
    <title>Ajax Example</title>
  </head>
  <body>
    <h2>GitHub User Info</h2>

    <button onclick="fetchUserInfo('js-primer-example');">Get user info</button>
    <!-- 整形したHTMLの挿入先 -->
    <div id="result"></div>

    <script src="index.js"></script>
  </body>
</html> 
```

当应用程序启动并点击按钮时，用户信息将按如下方式显示。

![用户信息显示](img/ce2fbff4fda4c97207376b67af18ae7d.png)

## *本节的检查清单*

**   使用模板字符串来构建 HTML 字符串**

+   使用`innerHTML`属性将 HTML 字符串添加到 DOM 中

+   使用带标签的模板函数来转义 HTML 字符串

+   调用`fetchUserInfo`函数，确认用户信息已显示在 HTML 中

到目前为止的应用程序可以通过以下 URL 进行查看。

+   [`jsprimer.net/use-case/ajaxapp/display/example/`](https://jsprimer.net/use-case/ajaxapp/display/example/)
