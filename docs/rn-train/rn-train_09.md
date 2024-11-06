# 10 性能（草稿）

# 10 性能（草稿）

# 10.1 shouldComponentUpdate

# 10.1 shouldComponentUpdate

本章节适用于所有的 React 应用程序。

## shouldComponentUpdate

React 通常很快，但您仍然可以通过优化函数[shouldComponentUpdate](https://facebook.github.io/react/docs/component-specs.html#updating-shouldcomponentupdate)来提高性能。默认情况下，它返回 true，如果返回 false，则会跳过渲染函数。

当状态或属性更改时，此函数频繁调用。因此，**保持简单和快速**非常重要。当您调用`setState`时，即使之前的状态等于当前状态，`render`函数也将始终执行。这就是我们可以进行一些优化的地方。

[演示 1](https://jsbin.com/figuse/edit?html,js,output)

在演示 1 中，点击按钮时，它将设置相同的状态，但渲染次数仍然会增加。

[演示 2](http://jsbin.com/culipes/5/edit?html,js,output)

在演示 2 中，我们检查名字的值是否与之前相等，如果相等，则返回 false，然后我们减少渲染函数的次数。

但是，如果我们的状态结构很复杂，例如 `{ a: { b: { c: [1, 2, 3] }}}`，我们必须进行深层比较。这显然违反了我们上面提到的规则，**保持 shouldComponentUpdate 简单**。

## Immutable-js

[Immutable](https://en.wikipedia.org/wiki/Immutable_object) 是来自[函数式编程](https://en.wikipedia.org/wiki/Functional_programming)的概念，不可变数据的一个特征是创建后无法修改。因此，有一些算法用于为每个数据结构创建哈希（更多[详细信息](https://en.wikipedia.org/wiki/Persistent_data_structure)）。我们可以使用这个特性来防止深度比较，浅比较就足够了。在这里，我们将使用来自 Facebook 的[immutable-js](https://facebook.github.io/immutable-js/)。

[演示 3](http://jsbin.com/vofubiy/8/edit?html,js,output)

在演示 3 中，我们点击第一个按钮几次，次数只会增加一次，点击第二个按钮，次数会增加。
