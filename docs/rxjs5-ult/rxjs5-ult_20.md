# 运算符 - 分组

# 运算符分组

## 缓冲

`buffer()`运算符的签名是：

```
buffer([breakObservable]) 
```

Buffer 本身意味着我们等待直到`breakObservable`发生才发出任何值。一个例子是以下内容：

```
let breakWhen$ = Rx.Observable.timer(1000);

let stream$ = Rx.Observable.interval(200)
.buffer( breakWhen$ );

stream$.subscribe((data) => console.log( 'values',data )); 
```

在这种情况下，值 0,1,2,3 一次性发出。

### 业务案例

**自动完成**

处理`buffer()`运算符时最明显的情况是`自动完成`。但是`自动完成`是如何工作的呢？让我们逐步看一下

+   用户输入按键

+   基于这些按键击键进行搜索。然而，重要的是搜索本身是在您输入时进行的，无论是因为您输入了 x 个字符还是更常见的方法是让您完成输入并进行搜索，您可能正在输入时进行编辑。因此，让我们迈出这样一个解决方案的第一步：

```
let input = document.getElementById('example');
let input$  = Rx.Observable.fromEvent( input, 'keyup' )

let breakWhen$ = Rx.Observable.timer(1000);
let debounceBreak$ = input$.debounceTime( 2000 );

let stream$ = input$
.map( ev => ev.key )
.buffer( debounceBreak$ );

stream$.subscribe((data) => console.log( 'values',data )); 
```

我们捕获`keyup`事件。我们还使用了一个`debounce()`运算符，它基本上表示; 一旦您停止输入 x 毫秒，我将发出值。然而，这个解决方案只是迈出的第一步，因为它报告了实际键入的确切键。一个更好的解决方案是捕获输入元素的实际内容，并执行一个 ajax 调用，所以让我们看一个更精细的解决方案：

```
let input = document.getElementById('example');
let input$  = Rx.Observable.fromEvent( input, 'keyup' )

let breakWhen$ = Rx.Observable.timer(1000);
let debounceBreak$ = input$.debounceTime( 2000 );

let stream$ = input$
.map( ev => { 
    return ev.key })
.buffer( debounceBreak$ )
.switchMap((allTypedKeys) => {
    // do ajax
    console.log('Everything that happened during 2 sec', allTypedKeys)
    return Rx.Observable.of('ajax based on ' + input.value);
});

stream$.subscribe((data) => console.log( 'values',data )); 
```

让我们称之为`超级自动完成`。之所以取这个名字是因为我们保存用户最终决定成为 Ajax 调用之前所做的每一个交互。因此，上面的结果可能如下所示：

```
// from switchMap
Everything that happened during 2 sec ["a", "a", "a", "Backspace", "Backspace", "Backspace", "Backspace", "b", "b", "Backspace", "Backspace", "Backspace", "f", "g", "h", "f", "h", "g"]

// in the subscribe(fnValue)
app-buffer.js:31 values ajax based on fghfgh 
```

正如您所看到的，我们可能存储有关用户的更多信息，而不仅仅是他们进行了自动完成搜索的事实，我们可以存储他们的输入方式，这可能有趣也可能无趣..

**双击**

在上面的示例中，我展示了如何捕获键的组可能是有趣的，但另一组可能感兴趣的 UI 事件是鼠标点击，即捕获单击、双击或三击。如果不使用 Rxjs，编写这样的代码会相当不优雅，但使用 Rxjs，就像轻而易举一样：

```
let btn = document.getElementById('btn2');
let btn$  = Rx.Observable.fromEvent( btn, 'click' )

let debounceMouseBreak$ = btn$.debounceTime( 300 );

let btnBuffered$ = btn$
.buffer( debounceMouseBreak$ )
.map( array => array.length )
.filter( count => count >= 2 )
;

btnBuffered$.subscribe((data) => console.log( 'values',data )); 
```

多亏了`debounce()`运算符，我们能够表达在发出任何内容之前等待 300 毫秒。这是相当少的行数，我们很容易决定我们的过滤器应该是什么样子。

## bufferTime

`bufferTime()`的签名是

```
bufferTime([ms]) 
```

这个想法是记录在那个时间片段内发生的一切并输出所有值。下面是在 1 秒时间片段内记录输入上的所有活动的示例。

```
let input = document.getElementById('example');
let input$  = Rx.Observable.fromEvent( input, 'input' )
.bufferTime(1000);

input$.subscribe((data) => console.log('all inputs in 1 sec', data)); 
```

在这种情况下，您将得到一个看起来像这样的输出：

```
all inputs in 1 sec [ Event, Event... ] 
```

可能不太可用，所以我们可能需要使用`filter()`来查看实际输入的内容，就像这样：

```
let input = document.getElementById('example');
let input$  = Rx.Observable.fromEvent( input, 'keyup' )
.map( ev => ev.key)
.bufferTime(1000);

input$.subscribe((data) => console.log('all inputs in 1 sec', data)); 
```

还要注意我将事件更改为`keyup`。现在我们能够看到发生的所有`keyup`事件。

### 业务案例

上面的示例在以下情况下可能非常有用：如果您想记录站点上另一个用户正在做什么，并想重放他们曾经做过的所有交互，或者如果他们开始输入，您想通过套接字发送此信息。最后一种情况是现在的标准功能之一，您可以看到对方正在输入。因此，这绝对有其用途。

## groupBy

TODO
