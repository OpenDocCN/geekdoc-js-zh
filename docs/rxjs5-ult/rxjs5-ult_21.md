# 错误处理

# 错误处理

有两种主要方法来处理流中的错误。你可以重试你的流，看看最终是否能够工作，或者你可以接受错误并将其转换。

## 重试 - 现在怎么样？

当你出于某种原因相信错误是暂时的时，这种方法是有道理的。通常*不稳定的连接*是一个很好的候选对象。使用*不稳定的连接*，终点可能会有答复，比如每第 5 次尝试一次。重点是第一次尝试可能会失败，但重试 x 次，尝试之间有一定的时间，最终会得到终点的答复。

### 重试

`retry()`操作符让我们重试整个流，每个值重试 x 次，具有以下签名：

```
retry([times]) 
```

使用`retry()`操作符的重要事项是它延迟了当错误回调被调用时。给定以下代码，错误回调会立即被调用：

```
let stream$ = Rx.Observable.of(1,2,3)
.map(value => {
   if(value > 2) { throw 'error' }
});

stream$.subscribe(
   data => console.log(data),
   err => console.log(err)
) 
```

当错误回调被调用时，流实际上会死掉，这就是`retry()`操作符的作用。通过像这样附加它：

```
let stream$ = Rx.Observable.of(1,2,3)
.map(value => {
   if(value > 2) { throw 'error' }
})
retry(5) 
```

在最后放弃并触发错误回调之前，这将运行值序列 5 次。然而，在这种情况下，代码的编写方式只会生成`1,2`五次。因此，我们的代码并没有充分利用这个运算符的潜力。你可能想要的是在尝试之间能够改变一些东西。想象一下，你的 observable 看起来是这样的：

```
let urlsToHit$ = Rx.Observable.of(url, url2,url3); 
```

在这种情况下很明显，一个终点在第一次尝试时可能会回答不好，或者根本不会回答，因此有意义的是重试 x 次。

但是在 ajax 调用的情况下，假设我们的业务情景是*不稳定的连接*，立即重试是没有意义的，所以我们必须在其他地方寻找更好的操作符，我们需要寻找`retryWhen()`

### retryWhen

`retryWhen()`操作符给了我们在流上操作和适当处理它的机会

```
retryWhen( stream => {
   // return it in a better condition, hopefully
}) 
```

让我们写一段简单的代码片段

```
let values$ = Rx.Observable
.of( 1,2,3,4 )
.map(val => {
    if(val === 2) { throw 'err'; }
    else return val;
})
.retryWhen( stream => {
    return stream;
} );

values$.subscribe(
    data => console.log('Retry when - data',data),
    err => console.error('Retry when - Err',err)
) 
```

这种写法会返回`1`，直到我们用尽内存为止，因为算法在值`2`上总是崩溃，并且会无限期地重试流，由于我们缺乏结束条件。我们需要做的是以某种方式表明错误已经被修复。如果流尝试击中的是 URL 而不是发出数字，那么一个响应的终点就是解决方案，但在这种情况下，我们必须写出这样的东西：

```
let values$ = Rx.Observable.interval(1000).take(5);
let errorFixed = false;

values$
.map((val) => {
   if(errorFixed) { return val; }
   else if( val > 0 && val % 2 === 0) {
      errorFixed = true;
      throw { error : 'error' };

   } else {
      return val;
   }
})
.retryWhen((err) => {
    console.log('retrying the entire sequence');
    return err;
})
.subscribe((val) => { console.log('value',val) })

// 0 1 'wait 200ms' retrying the whole sequence 0 1 2 3 4 
```

但这与我们使用`retry()`操作符所做的非常相似，上面的代码只会重试一次。真正的好处是能够改变我们在`retryWhen()`内返回的流，即像这样涉及延迟：

```
.retryWhen((err) => {
    console.log('retrying the entire sequence');
    return err.delay(200)
}) 
```

这样确保序列在重试之前有 200ms 的延迟，对于 ajax 场景来说，这可能足够让我们的终点*整理好*并开始响应。

**小心**

`delay()`操作符在`retryWhen()`内部使用，以确保重试在稍后的某个时候发生，以此给网络一个恢复的机会。

#### 带延迟和次数的重试

到目前为止，当我们想要重试序列 x 次时，我们使用了`retry()`操作符，当我们想要延迟尝试之间的时间时，我们使用了`retryWhen()`，但是如果我们两者都想要怎么办。我们可以。我们需要考虑如何记住我们迄今为止尝试的次数。引入一个外部变量并保留计数是非常诱人的，但这不是我们以函数式方式处理事务的方式，记住副作用是被禁止的。那么我们该如何解决呢？有一个名为`scan()`的操作符，它允许我们为每次迭代累积值。因此，如果你在`retryWhen()`内部使用 scan，我们可以通过这种方式跟踪我们的尝试：

```
let ATTEMPT_COUNT = 3;
let DELAY = 1000;
let delayWithTimes$ = Rx.Observable.of(1,2,3)
.map( val => {
  if(val === 2) throw 'err'
  else return val;
})
.retryWhen(e => e.scan((errorCount, err) => {
    if (errorCount >= ATTEMPT_COUNT) {
        throw err;
    }
    return errorCount + 1;
}, 0).delay(DELAY));

delayWithTimes$.subscribe(
    val => console.log('delay and times - val',val),
    err => console.error('delay and times - err',err)
) 
```

## 转换 - 这里没有什么可看的。

当你遇到错误并选择将其重新制作成有效的 Observable 时，就采用这种方法。

所以让我们通过创建一个使命在于惨败的 Observable 来举例说明这一点。

```
let error$ = Rx.Observable.throw('crash');

error$.subscribe( 
  data => console.log( data ),
  err => console.log( err ),
  () => console.log('complete')
) 
```

这段代码只会执行错误回调，而不会触发完成回调。

### 修补它

我们可以通过引入`catch()`操作符来修补这个问题。用法如下：

```
let errorPatched$ = error$.catch(err => { return Rx.Observable.of('Patched' + err) });
errorPatched$.subscribe((data) => console.log(data) ); 
```

正如你所看到的，通过使用`.catch()`并返回一个新的 Observable 来“修补它”，修复了流。问题是这是否是你想要的。当然，流会存活并达到完成，并且可以在崩溃点之后发出任何值。

如果这不是你想要的，也许上面的重试方法更适合你，你来判断。

### 多个流怎么办？

你没想到会这么容易吧？通常在编写 Rxjs 代码时，你会处理多个流，并且使用`catch()`操作符的方法很棒，只要你知道在哪里放置你的操作符。

```
let badStream$ = Rx.Observable.throw('crash');
let goodStream$ = Rx.Observable.of(1,2,3,);

let merged$ = Rx.Observable.merge(
  badStream$,
  goodStream$
);

merged$.subscribe(
   data => console.log(data),
   err => console.error(err),
   () => console.log('merge completed') 
) 
```

猜猜发生了什么？1) crash + 值被发出 + 完成 2) crash + 值被发出 3) 仅发出 crash

遗憾的是发生了第 3 种情况。这意味着我们几乎没有处理错误。

**让我们修补它** 所以我们需要修补错误。我们使用`catch()`操作符进行修补。问题是在哪里？

让我们试试这个？

```
let mergedPatched$ = Rx.Observable.merge(
    badStream$,
    goodStream$
).catch(err => Rx.Observable.of(err));

mergedPatched$.subscribe(
    data => console.log(data),
    err => console.error(err),
    () => console.log('patchedMerged completed')
) 
```

在这种情况下，我们得到了'crash'和'patchedMerged completed'。好吧，我们达到了完成，但它仍然没有给我们`goodStream$`的值。所以这是一个更好的方法，但仍然不够好。

**更好地修补它** 因此，在`merge()`之后添加`catch()`操作符确保了流完成，但这还不够好。让我们尝试改变`catch()`的放置位置，放在合并之前。

```
let preMergedPatched$ = Rx.Observable.merge(
    badStream$.catch(err => Rx.Observable.of(err)),
    goodStream$
).catch(err => Rx.Observable.of(err));

preMergedPatched$.subscribe(
    data => console.log(data),
    err => console.error(err),
    () => console.log('pre patched merge completed')
) 
```

然后，我们得到了值，我们的错误发出其错误消息作为一个新的很好的 Observable，并且我们得到了完成。

**注意** 放置`catch()`的位置很重要。

#### 适者生存

还有另一种可能感兴趣的情况。上面的情况假设你希望一切都被发出，错误消息、值，一切。

如果不是这种情况，如果你只关心行为正常的流中的值呢？假设这是你的情况，有一个操作符可以做到这一点`onErrorResumeNext()`

```
let secondBadStream$ = Rx.Observable.throw('bam');
let gloriaGaynorStream$ = Rx.Observable.of('I will survive');

let emitSurviving = Rx.Observable.onErrorResumeNext(
    badStream$,
    secondBadStream$,
    gloriaGaynorStream$
);

emitSurviving.subscribe(
    data => console.log(data),
    err => console.error(err),
    () => console.log('Survival of the fittest, completed')
) 
```

这里唯一发出的是'I will survive'和'Survival of the fittest, completed'。
