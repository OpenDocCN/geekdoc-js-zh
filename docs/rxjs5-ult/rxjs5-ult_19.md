# 操作符时间

# 操作符时间

这不是一个简单的话题。这里有许多应用领域，你可能想要同步来自 API 的响应，或者你可能想要处理其他类型的流，比如 UI 中的点击或键盘按键事件。

有许多处理时间的操作符，如`delay` `debounce` `throttle` `interval` 等。

这不是一个简单的话题。这里有许多应用领域，你可能想要同步来自 API 的响应，或者你可能想要处理其他类型的流，比如 UI 中的点击或键盘按键事件。

## 间隔

这个操作符用于构建一个 Observable，它的本质是定期泵送值，签名：

```
Rx.Observable.interval([ms]) 
```

示例用法：

```
Rx.Observable.interval(100)

// generates forever 
```

因为这个操作符会永远生成值，你往往希望将它与 `take()` 操作符结合使用，限制生成值的数量，然后结束调用，看起来像这样：

```
Rx.Observable.interval(1000).take(3)

// generates 1,2,3 
```

## 计时器

计时器是一个有趣的操作符，它可以根据调用方式的不同而以几种方式起作用。它的签名是

```
Rx.Observable.timer([initial delay],[thereafter]) 
```

然而，只有初始参数是必需的，因此根据使用的参数数量，由此产生了不同类型的操作符。

**一次性**

```
let stream$ = Rx.Observable.timer(1000);

stream$.subscribe(data => console.log(data));

// generates 0 after 1 sec 
```

这变成了一个一次性的操作，因为我们没有定义下一个值应该何时发生。

**指定第二个参数**

```
let moreThanOne$ = Rx.Observable.timer(2000, 500).take(3);

moreThanOne$.subscribe(data => console.log('timer with args', data));

// generate 0 after 2 sec and thereafter 1 after 500ms and 2 after additional 500ms 
```

因此，这个更灵活，根据第二个参数继续发射值。

## 延迟

`delay()` 是一个延迟发射每个值的操作符，简单来说它的工作原理如下：

```
var start = new Date();
let stream$ = Rx.Observable.interval(500).take(3);

stream$
.delay(300)
.subscribe((x) => {
    console.log('val',x);
    console.log( new Date() - start );
})

//0 800ms, 1 1300ms,2 1800ms 
```

### 业务案例

延迟可以在多个地方使用，但一个很好的例子是在处理错误时，特别是如果我们正在处理`不稳定的连接`并希望在 x 毫秒后重试整个流程：

在错误处理章节中阅读更多

## 抽样

我通常把这种情况想象成*对手说话*。我的意思是事件只在特定的时间点触发

### 业务案例

因此，忽略 x 毫秒内的事件是非常有用的。想象一下一个保存按钮被反复按下。只有在 x 毫秒后才执行并忽略其他的按下，这不是很好吗？

```
const btn = document.getElementById('btnIgnore');
var start = new Date();

const input$ = Rx.Observable
  .fromEvent(btn, 'click')

  .sampleTime(2000);

input$.subscribe(val => {
  console.log(val, new Date() - start);
}); 
```

上面的代码就是这样做的。

## debounceTime

因此，`debounceTime()` 是一个告诉你：我不会一直发射数据，而是在一定的时间间隔内发射数据的操作符。

### 业务案例

Debounce 是一个众所周知的概念，特别是当你在键盘上输入按键时。这是一种说法，我们不关心每次按键抬起，但是一旦你停止输入一段时间，我们应该关心。这通常是你开始自动完成的方式。比如，如果用户 x 毫秒内没有输入，那可能意味着我们应该进行一个 ajax 调用并获取结果。

```
const input = document.getElementById('input');

const example = Rx.Observable
  .fromEvent(input, 'keyup')
  .map(i => i.currentTarget.value);

//wait 0.5s, between keyups, throw away all other values
const debouncedInput = example.debounceTime(500);

const subscribe = debouncedInput.subscribe(val => {
  console.log(`Debounced Input: ${val}`);
}); 
```

以下只在你停止输入 500 毫秒后输出一个值，然后值得报告一下，即发射一个值。

## 节流时间

待办事项

## 缓冲

这个操作符有能力在输出值之前记录 x 个发射的值，这个操作符带有一个或两个输入参数。

```
.buffer( whenToReleaseValuesStartObservable )

or

.buffer( whenToReleaseValuesStartObservable, whenToReleaseValuesEndObservable ) 
```

那么这意味着什么？这意味着假设我们有一个点击流，我们可以将其切割成漂亮的小块，每个块的长度都相同。使用第一个带有一个参数的版本，我们可以给它一个时间参数，比如 500 毫秒。因此，某个东西在 500 毫秒内发出值，然后这些值被发出，另一个 Observable 开始，旧的被抛弃。这很像使用秒表并每次记录 500 毫秒。示例：

```
let scissor$ = Rx.Observable.interval(500)

let emitter$ = Rx.Observable.interval(100).take(10) // output 10 values in total
.buffer( scissor$ )

// [0,1,2,3,4] 500ms [5,6,7,8,9] 
```

弹珠图

```
--- c --- c - c --- >
-------| ------- |- >
Resulting stream is :
------ r ------- r r  -- > 
```

### 业务案例

那么这个案例的业务需求是什么呢？`双击`，显然对 `单击` 做出反应很容易，但如果你只想在 `双击` 或 `三连击` 上执行某个操作，你该如何编写处理代码呢？你可能会从以下代码开始：

```
 $('#btn').bind('click', function(){
  if(!start) { start = timer.start(); }
  timePassedSinceLastClickInMs = now - start;
  if(timePassedSinceLastClickInMs < 250) {
     console.log('double click');

  } else {
     console.log('single click')
  }

  start = timer.start();  
}) 
```

将上述视为伪代码的尝试。关键是你需要跟踪一堆变量，记录点击之间经过了多长时间。这不是看起来很好的代码，缺乏优雅。

#### Rxjs 中的模型

到目前为止，我们知道 Rxjs 是关于流和随时间建模值的。点击也不例外，它们随时间发生。

```
---- c ---- c ----- c ----- > 
```

但是我们只关心点击在时间上接近时的情况，即双击或三连击，如下所示：

```
 --- c - c ------ c -- c -- c ----- c 
```

从上述流中，你应该能够推断出发生了一次 `双击`，一次 `三连击` 和一次 `单击`。

那么如果我这样做了，接下来呢？你希望流能够很好地分组，以便告诉我们这一点，即它需要将这些点击作为一组发出。`filter()` 作为一个操作符让我们可以做到这一点。如果我们定义 300 毫秒足够长的时间来收集事件，我们可以将我们的时间从 0 到永远切割成 300 毫秒的块，代码如下：

```
let clicks$ = Rx.Observable.fromEvent(document.getElementById('btn'), 'click');

let scissor$ = Rx.Observable.interval(300);

clicks$.buffer( scissor$ )
      //.filter( (clicks) => clicks.length >=2 )
      .subscribe((value) => {
          if(value.length === 1) {
            console.log('single click')
          }
          else if(value.length === 2) {
            console.log('double click')
          }
          else if(value.length === 3) {
            console.log('triple click')
          }

      }); 
```

以以下方式阅读代码，缓冲流 `clicks$` 将每隔 300 毫秒发出其值，300 毫秒由 `scissor$` 流决定。因此 `scissor$` 流就像是剪刀，切割我们的点击流，然后我们就有了一个优雅的 `双击` 方法。正如你所看到的，上述代码捕获了所有类型的点击，但通过取消注释 `filter()` 操作，我们只得到 `双击` 和 `三连击`。

`filter()` 操作符也可以用于其他用途，比如记录 UI 中发生的事件并为用户重放，只有你的想象力限制了它的用途。
