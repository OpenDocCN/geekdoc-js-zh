# Todo 应用程序的重构

> 原文：[`jsprimer.net/use-case/todoapp/final/`](https://jsprimer.net/use-case/todoapp/final/)

在前一节中，我们已经实现了计划中的所有 Todo 应用程序功能。 但是，如果查看`App.js`，几乎所有内容都是关于 HTML 元素的处理。 随着显示内容的增加，HTML 元素的创建处理会呈线性增长。 如果继续扩展 Todo 应用程序，`App.js`会变得臃肿，代码难以阅读，维护性会降低。

现在让我们回顾一下`App.js`的角色。 拥有一个名为`App`的类，这个类负责初始化 Model 以及在 HTML 元素和 Model 之间传递事件。 可以说，它是一个管理者，负责将来自显示的事件传递给 Model，并将来自 Model 的变更事件传递给显示。

在这个部分，为了将`App`类专注于作为事件管理者的角色，我们将`App`类中负责创建 HTML 元素的过程提取到另一个类中进行重构。

## [](#component)*View 组件*

*`App`类中占据大部分的是创建与`TodoItemModel`数组对应的 Todo 列表的 HTML 元素的过程。 我们将这样的显示处理分解为各个模块，并将`App`类中创建的模块用作重构的形式。 在这里，我们将处理显示的类称为 View 组件，并通过在文件名末尾添加`View`来区分。

Todo 列表的显示由以下两个部件（View 组件）组成。

+   Todo 项目视图组件

+   将 Todo 项目作为列表整合到 Todo 列表视图组件中

创建与此部件对应的下一个 View 模块。 这些 View 模块将在`src/view/`目录中创建。

+   `TodoItemView`: Todo 项目视图组件

+   `TodoListView`: Todo 列表视图组件

### [](#TodoItemView)*创建 TodoItemView*

*首先，从与 Todo 项目对应的`TodoItemView`开始创建。

创建`src/view/TodoItemView.js`文件，并`export`如下`TodoItemView`类。 这个`TodoItemView`具有返回与 Todo 项目对应的 HTML 元素的`createElement`方法。

src/view/TodoItemView.js

```
import { element } from "./html-util.js";

export class TodoItemView {
    /**
     * `todoItem`に対応するTodoアイテムのHTML 要素を作成して返す
     * @param {TodoItemModel} todoItem
     * @param {function({id:number, completed: boolean})} onUpdateTodo チェックボックスの更新イベントリスナー
     * @param {function({id:number})} onDeleteTodo 削除ボタンのクリックイベントリスナー
     * @returns {Element}
     */
    createElement(todoItem, { onUpdateTodo, onDeleteTodo }) {
        const todoItemElement = todoItem.completed
            ? element`<li><input type="checkbox" class="checkbox" checked>
                                    <s>${todoItem.title}</s>
                                    <button class="delete">x</button>
                                </li>`
            : element`<li><input type="checkbox" class="checkbox">
                                    ${todoItem.title}
                                    <button class="delete">x</button>
                                </li>`;
        const inputCheckboxElement = todoItemElement.querySelector(".checkbox");
        inputCheckboxElement.addEventListener("change", () => {
            // コールバック関数に変更
            onUpdateTodo({
                id: todoItem.id,
                completed: !todoItem.completed
            });
        });
        const deleteButtonElement = todoItemElement.querySelector(".delete");
        deleteButtonElement.addEventListener("click", () => {
            // コールバック関数に変更
            onDeleteTodo({
                id: todoItem.id
            });
        });
        // 作成したTodoアイテムのHTML 要素を返す
        return todoItemElement;
    }
} 
```

TodoItemView 的`createElement`方法基于`App`类中创建 HTML 元素的部分。 `createElement`方法接收的不仅仅是`TodoItemModel`实例，还有`onUpdateTodo`和`onDeleteTodo`这两个监听器函数。 当 View 中发生相应事件时，这些接收到的监听器函数将被调用。

通过从外部接收监听器函数作为参数，可以在事件发生时在 View 类的外部定义具体处理。

例如，这个`TodoItemView`类可以这样使用。 接收`TodoItemModel`实例和事件监听器对象，并返回 Todo 项目的 HTML 元素。

使用 TodoItemView 的示例代码

```
import { render } from "./html-util.js";
import { TodoItemModel } from "../model/TodoItemModel.js";
import { TodoItemView } from "./TodoItemView.js";

// TodoItemViewをインスタンス化
const todoItemView = new TodoItemView();
// 対応するTodoItemModelを作成する
const todoItemModel = new TodoItemModel({
    title: "あたらしいTodo",
    completed: false
});
// TodoItemModelからHTML 要素を作成する
const todoItemElement = todoItemView.createElement(todoItemModel, {
    onUpdateTodo: () => {
        console.log("チェックボックスが更新されたときに呼ばれるリスナー関数");
    },
    onDeleteTodo: () => {
        console.log("削除ボタンがクリックされたときに呼ばれるリスナー関数");
    }
});
render(todoItemElement, document.body); // <li>要素をdocument.bodyへレンダリング 
```

### [](#TodoListView)*创建 TodoListView*

*接下来创建与 Todo 列表对应的`TodoListView`。

创建`src/view/TodoListView.js`文件，并`export`如下`TodoListView`类。 这个`TodoListView`具有从`TodoItemModel`数组中创建 Todo 列表的 HTML 元素并返回的`createElement`方法。

src/view/TodoListView.js

```
import { element } from "./html-util.js";
import { TodoItemView } from "./TodoItemView.js";

export class TodoListView {
    /**
     * `todoItems`に対応するTodoリストのHTML 要素を作成して返す
     * @param {TodoItemModel[]} todoItems TodoItemModelの配列
     * @param {function({id:number, completed: boolean})} onUpdateTodo チェックボックスの更新イベントリスナー
     * @param {function({id:number})} onDeleteTodo 削除ボタンのクリックイベントリスナー
     * @returns {Element} TodoItemModelの配列に対応したリストのHTML 要素
     */
    createElement(todoItems, { onUpdateTodo, onDeleteTodo }) {
        const todoListElement = element`<ul></ul>`;
        // 各 TodoItemモデルに対応したHTML 要素を作成し、リスト要素へ追加する
        todoItems.forEach(todoItem => {
            const todoItemView = new TodoItemView();
            const todoItemElement = todoItemView.createElement(todoItem, {
                onDeleteTodo,
                onUpdateTodo
            });
            todoListElement.appendChild(todoItemElement);
        });
        return todoListElement;
    }
} 
```

TodoListView 的`createElement`方法使用`TodoItemView`创建 Todo 项目的 HTML 元素，并将其添加到`todoListElement`中。 这个 TodoListView 的`createElement`方法也接收`onUpdateTodo`和`onDeleteTodo`的监听器函数。 但是，`TodoListView`直接将这些监听器函数传递给`TodoItemView`。 这是因为具体触发 DOM 事件的元素是在`TodoItemView`内部创建的。

## [](#app-refactoring)*重构 App*

*最后，使用最近创建的`TodoItemView`类和`TodoListView`类来重构`App`类。

将`App.js`重写为使用`TodoListView`类。 在`onChange`的监听器函数中，将`App`类更改为使用`TodoListView`类创建 Todo 列表的 HTML 元素。 在这种情况下，将相应的回调函数传递给 TodoListView 的`createElement`方法。

+   `onUpdateTodo`的回调函数中调用 TodoListModel 的`updateTodo`方法

+   在`onDeleteTodo`的回调函数中调用 TodoListModel 的`deleteTodo`方法

src/App.js

```
import { TodoListModel } from "./model/TodoListModel.js";
import { TodoItemModel } from "./model/TodoItemModel.js";
import { TodoListView } from "./view/TodoListView.js";
import { render } from "./view/html-util.js";

export class App {
    #todoListModel = new TodoListModel();

    mount() {
        const formElement = document.querySelector("#js-form");
        const inputElement = document.querySelector("#js-form-input");
        const containerElement = document.querySelector("#js-todo-list");
        const todoItemCountElement = document.querySelector("#js-todo-count");
        this.#todoListModel.onChange(() => {
            const todoItems = this.#todoListModel.getTodoItems();
            const todoListView = new TodoListView();
            // todoItemsに対応するTodoListViewを作成する
            const todoListElement = todoListView.createElement(todoItems, {
                // Todoアイテムが更新イベントを発生したときに呼ばれるリスナー関数
                onUpdateTodo: ({ id, completed }) => {
                    this.#todoListModel.updateTodo({ id, completed });
                },
                // Todoアイテムが削除イベントを発生したときに呼ばれるリスナー関数
                onDeleteTodo: ({ id }) => {
                    this.#todoListModel.deleteTodo({ id });
                }
            });
            render(todoListElement, containerElement);
            todoItemCountElement.textContent = `Todoアイテム数: ${this.#todoListModel.getTotalCount()}`;
        });
        formElement.addEventListener("submit", (event) => {
            event.preventDefault();
            this.#todoListModel.addTodo(new TodoItemModel({
                title: inputElement.value,
                completed: false
            }));
            inputElement.value = "";
        });
    }
} 
```

现在`App`类中的 HTML 元素创建处理已经移动到 View 类中，`App`类只负责管理 Model 和 View 之间的事件。 

### [](#app-event-listener)*整理 App 的事件监听器*

*查看`App`类中注册的事件监听器函数，共有以下 4 种类型。

| 事件流 | 监听器函数 | 角色 |
| --- | --- | --- |
| `Model` → `View` | `this.#todoListModel.onChange(listener)` | `TodoListModel`接收变更事件 |
| `查看` → `模型` | `formElement.addEventListener("submit", listener)` | 接收表单提交事件 |
| `查看` → `模型` | `onUpdateTodo: listener` | 接收待办事项复选框更新事件 |
| `查看` → `模型` | `onDeleteTodo: listener` | 接收待办事项删除事件 |

在三个从视图到模型的事件流中，存在三个监听器函数，它们分别在代码的不同位置编写。 此外，可以看出每个监听器函数都与待办事项应用的功能相对应。 为了使这些监听器函数更容易理解为待办事项应用的功能，让我们重新定义它们为`App`类的方法。

将相应的监听器函数改为`handle`方法，并进行调用。

src/App.js

```
import { render } from "./view/html-util.js";
import { TodoListView } from "./view/TodoListView.js";
import { TodoItemModel } from "./model/TodoItemModel.js";
import { TodoListModel } from "./model/TodoListModel.js";

export class App {
    #todoListView = new TodoListView();
    #todoListModel = new TodoListModel([]);

    /**
     * Todoを追加するときに呼ばれるリスナー関数
     * @param {string} title
     */
    handleAdd(title) {
        this.#todoListModel.addTodo(new TodoItemModel({ title, completed: false }));
    }

    /**
     * Todoの状態を更新したときに呼ばれるリスナー関数
     * @param {{ id:number, completed: boolean }}
     */
    handleUpdate({ id, completed }) {
        this.#todoListModel.updateTodo({ id, completed });
    }

    /**
     * Todoを削除したときに呼ばれるリスナー関数
     * @param {{ id: number }}
     */
    handleDelete({ id }) {
        this.#todoListModel.deleteTodo({ id });
    }

    mount() {
        const formElement = document.querySelector("#js-form");
        const inputElement = document.querySelector("#js-form-input");
        const todoItemCountElement = document.querySelector("#js-todo-count");
        const containerElement = document.querySelector("#js-todo-list");
        this.#todoListModel.onChange(() => {
            const todoItems = this.#todoListModel.getTodoItems();
            const todoListElement = this.#todoListView.createElement(todoItems, {
                // Appに定義したリスナー関数を呼び出す
                onUpdateTodo: ({ id, completed }) => {
                    this.handleUpdate({ id, completed });
                },
                onDeleteTodo: ({ id }) => {
                    this.handleDelete({ id });
                }
            });
            render(todoListElement, containerElement);
            todoItemCountElement.textContent = `Todoアイテム数: ${this.#todoListModel.getTotalCount()}`;
        });

        formElement.addEventListener("submit", (event) => {
            event.preventDefault();
            this.handleAdd(inputElement.value);
            inputElement.value = "";
        });
    }
} 
```

将监听器函数作为`App`类的方法排列，使待办事项应用的功能在代码中更加清晰可见。

## [](#section-conclusion)*本节总结*

*本节内容包括以下：*

+   将与显示相关的处理从 App 中分离到视图组件中

+   将待办事项应用功能与相应的监听器函数移到`App`类的方法中

+   完成了待办事项应用

您可以在以下网址找到已完成的待办事项应用。

+   [`jsprimer.net/use-case/todoapp/final/final/`](https://jsprimer.net/use-case/todoapp/final/final/)

事实上，此待办事项应用尚未完全实现为应用程序。

连续按 Enter 键时会添加空的待办事项是不期望的行为。 另外，在 App 的`mount`方法中，注册了 TodoListModel 的`onChange`方法等事件监听器，但并未解除这些事件监听器。 虽然这在此待办事项应用中并不是问题，但是未解除的事件监听器可能导致内存泄漏。

如果有余力，可以尝试添加以下功能以完成待办事项应用。

+   当标题为空时，禁止提交表单以添加待办事项

+   在 App 的`mount`方法中注册事件监听器后，为 App 实现了`unmount`方法，以便解除事件监听器

为了解决应用程序的生命周期问题，为 App 创建了`mount`方法和对应的`unmount`方法。 这样做可以根据 Web 页面的生命周期进行相应处理。

```
const app = new App();
// ページのロードが完了したときのイベント
window.addEventListener("load", () => {
    app.mount();
});
// ページがアンロードされたときのイベント
window.addEventListener("unload", () => {
    app.unmount();
}); 
```

您可以在以下网址找到实现剩余待办事项的代码。 请务必尝试自己实现并思考网页或应用的运作方式。

+   [`jsprimer.net/use-case/todoapp/final/more/`](https://jsprimer.net/use-case/todoapp/final/more/)

## [](#todo-conclusion)*待办事项应用总结*

*本次我们将待办事项应用划分为模型和视图两个模块。 通过划分模块，我们提高了代码的可读性，并使待办事项应用更容易添加新功能。 关于模块划分等设计方面并没有唯一正确的答案，有各种不同的观点。*

选择待办事项应用作为案例是因为它在 JavaScript 网页应用中经常使用。 使用各种库实现的待办事项应用可以在[TodoMVC](https://todomvc.com/)网站上找到。 此次创建的待办事项应用是在不使用库的情况下实现的，删除了 TodoMVC 中的筛选功能等内容。^(1)

在现实生活中，几乎不可能完全不使用库来实现 Web 应用程序。 使用库可以省去编写像`html-util.js`这样的工具的麻烦，同时也更容易解决最后一个问题：生命周期。 

然而，即使在使用库进行开发时，第一部分的基本语法和第二部分介绍的 JavaScript 基础也很重要。 因为这些库也是建立在这些基础之上的。

此外，适用于构建应用程序的库也因应用程序类型和目的而异。 尽管某些库提供了像魔术般的功能，但请记住它们也是建立在基础技术之上的。

本书主要介绍了 JavaScript 的基础知识，但正如「ECMAScript」章节所述，JavaScript 的基础知识也在不断更新。 随着基础知识的更新，衍生出的库也会更新，而曾经的经典也会逐渐改变。 发现未知内容意味着 JavaScript 本身正在发展。

阅读本书后仍有不理解或不熟悉的内容并不是问题。 在 JavaScript 这种不断发展的语言及其使用环境中，能够查找到未知内容是很重要的。

> ¹. 在不使用库或框架的情况下实现 JavaScript 的代码有时被称为 Vanilla JS。 ↩
