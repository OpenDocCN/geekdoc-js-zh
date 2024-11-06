# イベントとモデル

> 原文：[`jsprimer.net/use-case/todoapp/event-model/`](https://jsprimer.net/use-case/todoapp/event-model/)

Todoアイテムを追加する機能を実装しましたが、イベントを受け取って直接 DOMを更新する方法には柔軟性がないという問題があります。 また「Todoアイテムの更新」という機能を実装するには、追加したTodoアイテム要素を識別する方法が必要です。 具体的には、Todoアイテムごとに`id`属性などのユニークな識別子がないため、特定のアイテムを指定して更新や削除をする機能が実装できません。

このセクションでは、まずどのような点で柔軟性の問題が起きやすいのかを見ていきます。 そして、柔軟性や識別子の問題を解決するために**モデル**という概念を導入し、「Todoアイテムの追加」の機能をリファクタリングしていきます。

## [](#direct-dom-modification-issue)*直接 DOMを更新する問題*

*「Todoアイテムの追加を実装する」では、操作した結果発生したイベントという入力に対して、DOM（表示）を直接更新していました。 そのため、TodoリストにTodoアイテムが何個あるか、どのようなアイテムがあるかという状態がDOM 上にしか存在しないことになります。

この場合にTodoアイテムの状態を更新するには、HTML 要素にTodoアイテムの情報（タイトルや識別子となるidなど）をすべて埋め込む必要があります。 しかし、HTML 要素は文字列しか扱えないため、Todoアイテムのデータを文字列にしないといけないという制限が発生します。

また、1つの操作に対して複数の箇所の表示が更新されることもあります。 今回のTodoアプリでもTodoリスト（`#js-todo-list`）とTodoアイテム数（`#js-todo-count`）の2 箇所を更新する必要があります。

次の表に**操作**に対して更新する**表示**をまとめてみます。

| 機能 | 操作 | 表示 |
| --- | --- | --- |
| Todoアイテムの追加 | フォームを入力して送信 | Todoリスト（`#js-todo-list`）にTodoアイテム要素を作成して子要素として追加。合わせてTodoアイテム数（`#js-todo-count`）を更新 |
| Todoアイテムの更新 | チェックボックスをクリック | Todoリスト（`#js-todo-list`）にある指定したTodoアイテム要素のチェック状態を更新 |
| Todoアイテムの削除 | 削除ボタンをクリック | Todoリスト（`#js-todo-list`）にある指定したTodoアイテム要素を削除。合わせてTodoアイテム数（`#js-todo-count`）を更新 |

1つの操作に対する表示の更新箇所が増えるほど、操作に対する処理（リスナーの処理）が複雑化していくことが予想できます。

ここでは、次の2つの問題が見つかりました。

+   Todoリストの状態がDOM 上にしか存在しないため、状態をすべてDOM 上に文字列で埋め込まないといけない

+   操作に対して更新する表示箇所が増えてくると、表示の処理が複雑化する

## [](#introduce-model)*モデルを導入する*

*この問題を避けるために、Todoアイテムという情報をJavaScriptクラスとしてモデル化します。 ここでのモデルとはTodoアイテムやTodoリストなどの**モノの状態や操作方法**を定義したオブジェクトという意味です。 クラスでは操作方法はメソッドとして実装し、状態はインスタンスのプロパティで管理できるため、今回はクラスでモデルを表現します。

たとえば、Todoリストを表現するモデルとして`TodoListModel`クラスを考えます。 TodoリストにはTodoアイテムを追加できるので、TodoListModelに`addItem`というメソッドがあると良さそうです。 また、Todoリストからアイテムの一覧を取得できる必要もあるので、TodoListModelに`getAllItems`というメソッドも必要そうです。 このようにTodoリストをクラスで表現する際に、オブジェクトがどのような処理や状態を持つかを考えて実装します。

このようにモデルを考えた後、先ほどの操作と表示の間にモデルを入れることを考えてみます。 「フォームを入力して送信」という**操作**をした場合には、`TodoListModel`（Todoリスト）に対して`TodoItemModel`（Todoアイテム）を追加します。 そして、`TodoListModel`からTodoアイテムの一覧を取得し、それを元にDOMを組み立て、**表示**を更新します。

先ほどの表にモデルを入れてみます。 **操作**に対する**モデルの処理**はさまざまですが、**操作**に対する**表示**の処理はどの場合も同じになります。 これは表示箇所が増えた場合でも**表示**の処理の複雑さが一定に保てることを意味しています。

| 機能 | 操作 | モデルの処理 | 表示 |
| --- | --- | --- | --- |
| Todoアイテムの追加 | フォームを入力して送信 | `TodoListModel`へ新しい`TodoItemModel`を追加 | `TodoListModel`を元に表示を更新 |
| Todoアイテムの更新 | チェックボックスをクリック | `TodoListModel`の指定した`TodoItemModel`の状態を更新 | `TodoListModel`を元に表示を更新 |
| Todoアイテムの削除 | 削除ボタンをクリック | `TodoListModel`から指定の`TodoItemModel`を削除 | `TodoListModel`を元に表示を更新 |

この表を元に改めて先ほどの問題点を見ていきましょう。

> Todoリストの状態がDOM 上にしか存在しないため、状態をすべてDOM 上に文字列で埋め込まないといけない

モデルであるクラスのインスタンスを参照すれば、Todoアイテムの情報が手に入ります。 またモデルはただのJavaScriptクラスであるため、文字列ではない情報も保持できます。 そのため、DOMにすべての情報を埋め込む必要はありません。

> 操作に対して更新する表示箇所が増えてくると、表示の処理が複雑化する

表示はモデルの状態を元にしてHTML 要素を作成し、表示を更新します。 モデルの状態が変化していなければ、表示は変わらなくても問題ありません。

因此，您应该在模型状态发生变化时更新显示，而不是在操作发生时更新显示。 具体来说，应该是“当`TodoListModel`的状态发生变化时”更新显示，而不是“输入表单并提交”后更新显示。

因此，显示端需要知道`TodoListModel`的状态何时发生变化。 这里再次出现的是事件。

## [](#model-and-event)*传递模型变化的事件*

*当提交表单时，会触发来自 form 元素的`submit`事件。 类似地，当`TodoListModel`的状态发生变化时，会向自身分发`change`事件。 显示端只需监听该事件，当事件发生时更新显示即可。

`TodoListModel`的状态变化包括“向`TodoListModel`添加新的`TodoItemModel`”等情况。 由于上表中的“模型处理”中的某些状态发生了变化，因此需要更新显示。

如果模型中也可以使用 DOM API 的事件机制，那么似乎可以创建一个机制，当模型更新时更新显示。 在浏览器的 DOM API 中，可以使用称为`EventTarget`的事件机制。 在 Node.js 中，可以使用名为`events`的内置模块来实现类似的事件机制。

虽然使用执行环境提供的事件机制很容易，但为了理解事件机制，让我们尝试创建一个具有分发和监听事件功能的类。

虽然听起来很困难，但是通过使用之前学到的类和回调函数等内容，可以实现这一点。

## [](#event-emitter)*EventEmitter*

*事件机制是由“分发事件的一方”和“监听事件的一方”两个方面组成。 有时会自己分发事件并自己监听事件。

这种事件机制的另一种说法是“当分发事件时，调用已注册的回调函数（事件监听器）”，这样就可以理解了。

要更新显示以反映模型的更新，只需创建一个“当`TodoListModel`更新时调用指定的回调函数”类即可实现目的。 但是，“当`TodoListModel`更新时”是非常具体的处理，因此每次增加模型时都要在每个模型中实现相同的处理是很困难的。

因此，我们将创建一个名为`EventEmitter`的类，该类具有在分发事件时调用已注册的回调函数（事件监听器）的功能。 

+   父类（`EventEmitter`）：在分发事件时调用已注册的回调函数（事件监听器）的类

+   子类（`TodoListModel`）：在更新值时，调用已注册的回调函数的类

首先，我们将创建父类`EventEmitter`。

`EventEmitter`是一个具有分发和监听功能的事件机制类。

+   分发端：`emit`方法调用已注册的所有回调函数以指定的`事件名称`

+   监听端：`addEventListener`方法允许为指定的`事件名称`注册任意的回调函数

通过这样做，您可以调用`emit`方法来调用与指定事件相关的已注册回调函数。 这种模式也被称为观察者模式，并且在许多执行环境中（如浏览器和 Node.js）存在类似的 API。

下面的代码将`EventEmitter`类定义为`src/EventEmitter.js`。

src/EventEmitter.js

```
export class EventEmitter {
    // 登録する [イベント名, Set(リスナー関数)] を管理するMap
    #listeners = new Map();
    /**
     * 指定したイベントが実行されたときに呼び出されるリスナー関数を登録する
     * @param {string} type イベント名
     * @param {Function} listener イベントリスナー
     */
    addEventListener(type, listener) {
        // 指定したイベントに対応するSetを作成しリスナー関数を登録する
        if (!this.#listeners.has(type)) {
            this.#listeners.set(type, new Set());
        }
        const listenerSet = this.#listeners.get(type);
        listenerSet.add(listener);
    }

    /**
     * 指定したイベントをディスパッチする
     * @param {string} type イベント名
     */
    emit(type) {
        // 指定したイベントに対応するSetを取り出し、すべてのリスナー関数を呼び出す
        const listenerSet = this.#listeners.get(type);
        if (!listenerSet) {
            return;
        }
        listenerSet.forEach(listener => {
            listener.call(this);
        });
    }

    /**
     * 指定したイベントのイベントリスナーを解除する
     * @param {string} type イベント名
     * @param {Function} listener イベントリスナー
     */
    removeEventListener(type, listener) {
        // 指定したイベントに対応するSetを取り出し、該当するリスナー関数を削除する
        const listenerSet = this.#listeners.get(type);
        if (!listenerSet) {
            return;
        }
        listenerSet.forEach(ownListener => {
            if (ownListener === listener) {
                listenerSet.delete(listener);
            }
        });
    }
} 
```

使用这个`EventEmitter`，您可以使用以下方式来监听事件和分发事件。 监听端使用`addEventListener`方法为事件类型（`type`）注册事件监听器（`listener`）。 分发端使用`emit`方法分发事件并调用事件监听器。

在下面的代码中，我们使用`addEventListener`方法为`test-event`事件注册了两个事件监听器。 因此，当使用`emit`方法分发`test-event`事件时，已注册的事件监听器被调用。

EventEmitter 的执行示例

```
import { EventEmitter } from "./EventEmitter.js";
const event = new EventEmitter();
// イベントリスナー（コールバック関数）を登録
event.addEventListener("test-event", () => console.log("One!"));
event.addEventListener("test-event", () => console.log("Two!"));
// イベントをディスパッチする
event.emit("test-event");
// コールバック関数がそれぞれ呼びだされ、コンソールには次のように出力される
// "One!"
// "Two!" 
```

## [](#event-emitter-todolist-model)*继承自 EventEmitter 的 TodoList 模型*

*接下来，我们将创建继承自创建的`EventEmitter`类的`TodoListModel`类。 创建一个新的`src/model/`目录，并在该目录中创建实现每个模型类的文件。

要创建的模型是表示 Todo 列表的`TodoListModel`和表示每个 Todo 项目的`TodoItemModel`。 `TodoListModel`通过保存多个`TodoItemModel`来表示 Todo 列表。

+   `TodoListModel`：表示 Todo 列表的模型

+   `TodoItemModel`：表示 Todo 项目的模型

首先，我们将创建`TodoItemModel`，并将其命名为`src/model/TodoItemModel.js`。

`TodoItemModel`类定义了每个 Todo 项目所需的信息。 每个 Todo 项目都有标题（`title`）、完成状态（`completed`）和每个项目的唯一标识符（`id`）。 由于它只是数据集合，因此可以是对象而不是类，但在这种情况下我们选择创建类。

下面的代码将`TodoItemModel`类定义为`src/model/TodoItemModel.js`。

src/model/TodoItemModel.js

```
// ユニークなIDを管理する変数
let todoIdx = 0;

export class TodoItemModel {
    /** @type {number} TodoアイテムのID */
    id;
    /** @type {string} Todoアイテムのタイトル */
    title;
    /** @type {boolean} Todoアイテムが完了済みならばtrue、そうでない場合はfalse */
    completed;

    /**
     * @param {{ title: string, completed: boolean }}
     */
    constructor({ title, completed }) {
        // idは連番となり、それぞれのインスタンス毎に異なるものとする
        this.id = todoIdx++;
        this.title = title;
        this.completed = completed;
    }
} 
```

下面的代码显示了`TodoItemModel`类是可以实例化的，并且每个`id`都自动具有不同的值。这个`id`将在稍后用于指定要更新的特定 Todo 项目时作为区分项目的标识符。

使用 TodoItemModel.js 的示例代码

```
import { TodoItemModel } from "./TodoItemModel.js";
const item = new TodoItemModel({
    title: "未完了のTodoアイテム",
    completed: false
});
const completedItem = new TodoItemModel({
    title: "完了済みのTodoアイテム",
    completed: true
});
// それぞれの`id`は異なる
console.log(item.id !== completedItem.id); // => true 
```

接下来，我们将创建`TodoListModel`并保存为`src/model/TodoListModel.js`。

`TodoListModel`类继承自先前创建的`EventEmitter`类。`TodoListModel`类维护了一个`TodoItemModel`数组，并在添加新 Todo 项目时将其添加到该数组中。在此过程中，为了通知`TodoListModel`的状态已更改，它会向自身分派`change`事件。

src/model/TodoListModel.js

```
import { EventEmitter } from "../EventEmitter.js";

export class TodoListModel extends EventEmitter {
    #items;
    /**
     * @param {TodoItemModel[]} [items] 初期アイテム一覧（デフォルトは空の配列）
     */
    constructor(items = []) {
        super();
        this.#items = items;
    }

    /**
     * TodoItemの合計個数を返す
     * @returns {number}
     */
    getTotalCount() {
        return this.#items.length;
    }

    /**
     * 表示できるTodoItemの配列を返す
     * @returns {TodoItemModel[]}
     */
    getTodoItems() {
        return this.#items;
    }

    /**
     * TodoListの状態が更新されたときに呼び出されるリスナー関数を登録する
     * @param {Function} listener
     */
    onChange(listener) {
        this.addEventListener("change", listener);
    }

    /**
     * 状態が変更されたときに呼ぶ。登録済みのリスナー関数を呼び出す
     */
    emitChange() {
        this.emit("change");
    }

    /**
     * TodoItemを追加する
     * @param {TodoItemModel} todoItem
     */
    addTodo(todoItem) {
        this.#items.push(todoItem);
        this.emitChange();
    }
} 
```

下面的代码是向`TodoListModel`类的实例添加新`TodoItemModel`的示例代码。当使用 TodoListModel 的`addTodo`方法添加新 Todo 项目时，将调用 TodoListModel 的`onChange`方法注册的事件监听器。

使用了 TodoListModel.js 的示例代码

```
import { TodoItemModel } from "./TodoItemModel.js";
import { TodoListModel } from "./TodoListModel.js";
// 新しいTodoリストを作成する
const todoListModel = new TodoListModel();
// 現在のTodoアイテム数は0
console.log(todoListModel.getTotalCount()); // => 0
// Todoリストが変更されたら呼ばれるイベントリスナーを登録する
todoListModel.onChange(() => {
    console.log("TodoListの状態が変わりました");
});
// 新しいTodoアイテムを追加する
// => `onChange`で登録したイベントリスナーが呼び出される
todoListModel.addTodo(new TodoItemModel({
    title: "新しいTodoアイテム",
    completed: false
}));
// Todoリストにアイテムが増える
console.log(todoListModel.getTotalCount()); // => 1 
```

现在每个所需的模型类都已经创建好了。接下来让我们使用这些模型来更新显示。

## [](#model-update-view)*使用模型更新视图*

*使用先前创建的`TodoListModel`和`TodoItemModel`类，重新编写“添加 Todo 项目”的功能。

在上一次的代码中，提交表单会直接向 DOM 添加元素。而在本次代码中，提交表单会向`TodoListModel`添加`TodoItemModel`。当`TodoListModel`中添加新的 Todo 项目时，由于已注册了`onChange`事件监听器，因此该监听器函数会被调用，从而更新 DOM（显示）。

首先看一下修改后的`App.js`。

src/App.js

```
import { TodoListModel } from "./model/TodoListModel.js";
import { TodoItemModel } from "./model/TodoItemModel.js";
import { element, render } from "./view/html-util.js";

export class App {
    // 1\. TodoListModelの初期化
    #todoListModel = new TodoListModel();

    mount() {
        const formElement = document.querySelector("#js-form");
        const inputElement = document.querySelector("#js-form-input");
        const containerElement = document.querySelector("#js-todo-list");
        const todoItemCountElement = document.querySelector("#js-todo-count");
        // 2\. TodoListModelの状態が更新されたら表示を更新する
        this.#todoListModel.onChange(() => {
            // TodoリストをまとめるList 要素
            const todoListElement = element`<ul></ul>`;
            // それぞれのTodoItem 要素をtodoListElement 以下へ追加する
            const todoItems = this.#todoListModel.getTodoItems();
            todoItems.forEach(item => {
                const todoItemElement = element`<li>${item.title}</li>`;
                todoListElement.appendChild(todoItemElement);
            });
            // コンテナ要素の中身をTodoリストをまとめるList 要素で上書きする
            render(todoListElement, containerElement);
            // アイテム数の表示を更新
            todoItemCountElement.textContent = `Todoアイテム数: ${this.#todoListModel.getTotalCount()}`;
        });
        // 3\. フォームを送信したら、新しいTodoItemModelを追加する
        formElement.addEventListener("submit", (event) => {
            event.preventDefault();
            // 新しいTodoItemをTodoListへ追加する
            this.#todoListModel.addTodo(new TodoItemModel({
                title: inputElement.value,
                completed: false
            }));
            inputElement.value = "";
        });
    }
} 
```

修改后的`App.js`主要有三个部分进行了更改，让我们逐个来看。

### [](#app-todolist-initialize)*1\. 初始化 TodoListModel*

*导入了创建的`TodoListModel`和`TodoItemModel`。

```
import { TodoListModel } from "./model/TodoListModel.js";
import { TodoItemModel } from "./model/TodoItemModel.js"; 
```

并且，在`App`类中通过私有类字段初始化了`TodoListModel`。由于`TodoListModel`不需要从外部访问，因此将其定义为私有类字段`#todoListModel`。这是为了使 Todo 列表在开始时（实例化`App`类时）以空状态开始。

src/App.jsより抜粋

```
// ...省略...
export class App {
    // 1\. TodoListModelの初期化
    #todoListModel = new TodoListModel();
    // ...省略...
} 
```

### [](#app-todolist-onchange)*2\. 当 TodoListModel 的状态更新时更新显示*

*在`mount`方法中实现了当`TodoListModel`更新时更新显示的逻辑。通过`TodoListModel`的`onChange`方法注册的监听器函数会在`TodoListModel`的状态更新时被调用。

这个监听器函数内部使用了 TodoListModel 的`getTodoItems`方法来获取 Todo 项目。然后，从项目列表中创建类似以下的列表元素（`todoListElement`）。

```
<!-- todoListElementの実質的な中身 -->
<ul>
    <li>Todoアイテム1のタイトル</li>
    <li>Todoアイテム2のタイトル</li>
</ul> 
```

创建的`todoListElement`元素使用了之前创建的`html-util.js`的`render`函数来覆盖容器元素的内容。此外，由于可以通过 TodoListModel 的`getTotalCount`方法获取项目数，因此可以删除管理项目数的`todoItemCount`变量。

src/App.jsより抜粋

```
import { TodoListModel } from "./model/TodoListModel.js";
// render 関数をimportに追加する
import { element, render } from "./view/html-util.js";
export class App {
    #todoListModel = new TodoListModel();

    mount() {
        // ...省略...
        this.#todoListModel.onChange(() => {
            // ...省略...
            // コンテナ要素の中身をTodoリストをまとめるList 要素で上書きする
            render(todoListElement, containerElement);
            // アイテム数の表示を更新
            todoItemCountElement.textContent = `Todoアイテム数: ${this.#todoListModel.getTotalCount()}`;
        });
        // ...省略...
    }
} 
```

### [](#app-add-new-todoitem)*3\. 提交表单后，添加新的 TodoItem*

*在上一次的代码中，提交表单（`submit`）时会直接向 DOM 添加元素。而在本次代码中，`TodoListModel`的状态更新后会更新显示的机制已经建立好了。

因此，在`submit`事件的监听器函数中，只需向`TodoListModel`添加新的`TodoItemModel`，显示就会更新。只需将直接向 DOM 添加`appendChild`的部分替换为使用`TodoListModel`的`addTodo`方法来更新模型即可。

## [](#conclusion)*总结*

*在本节中，我们使用模型和事件机制对上一节的“实现添加 Todo 项目”进行了重构。虽然代码量增加了，但下一步要实现的“更新 Todo 项目”和“删除 Todo 项目”也可以使用类似的机制来实现。与上一节直接更新 DOM 的操作不同，虽然添加很容易，但更新或删除现有元素需要指定元素，这就变得困难了。

下一节将实现剩余功能，如“更新 Todo 项目”和“删除 Todo 项目”。

## [](#section-checklist)*本节的检查清单*

**   理解了直接更新 DOM 的问题

+   使用`EventEmitter`类实现了事件机制

+   实现了 Todo 列表和 Todo 项目作为模型

+   继承了`TodoListModel`的`EventEmitter`类

+   使用模型来重构了添加 Todo 项目的功能

您可以在以下 URL 查看到目前的 Todo 应用程序。

+   [`jsprimer.net/use-case/todoapp/event-model/event-emitter/`](https://jsprimer.net/use-case/todoapp/event-model/event-emitter/)***********
