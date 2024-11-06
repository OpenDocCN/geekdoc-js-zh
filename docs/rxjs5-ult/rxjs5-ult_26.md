# Subject

# Subject

Subject 具有双重性质。它既具有 Observer 的行为，也具有 Observable 的行为。因此，以下是可能的：

发出值

```
subject.next( 1 )
subject.next( 2 ) 
```

订阅值

```
const subscription = subject.subscribe( (value) => console.log(value) ) 
```

总结一下，以下操作在其上存在：

```
next([value])
error([error message])
complete()
subscribe()
unsubscribe() 
```

## 充当代理

`Subject`可以充当代理，即从另一个流接收值，订阅者可以监听`Subject`的值。

```
let source$ = Rx.Observable.interval(500).take(3);
const proxySubject = new Rx.Subject();
let subscriber = source$.subscribe( proxySubject );

proxySubject.subscribe( (value) => console.log('proxy subscriber', value ) );

proxySubject.next( 3 ); 
```

因此，proxySubject `监听` `source$`

但它也可以做出自己的贡献

```
proxySubject.next( 3 )  // emits 3 and then 0 1 2 ( async ) 
```

**注意**

任何在创建订阅之前发生的`next()`都会丢失。还有其他可以满足此要求的 Subject 类型如下。

### 业务案例

那么这有什么有趣之处？它可以在某些数据到达时监听某个源，同时具有发出自己数据的能力，所有数据都到达同一订阅者。在组件之间以总线方式通信的能力是我能想到的最明显的用例。组件 1 可以通过`next()`传递其值，组件 2 可以订阅，反之亦然，组件 2 可以反过来发出值，组件 1 可以订阅。

```
sharedService.getDispatcher = function(){
   return subject;
}

sharedService.dispatch = function(value){
  subject.next(value)
} 
```

## ReplaySubject

原型：

```
new Rx.ReplaySubject([bufferSize], [windowSize], [scheduler]) 
```

示例：

```
let replaySubject = new Rx.ReplaySubject( 2 );

replaySubject.next( 0 );
replaySubject.next( 1 );
replaySubject.next( 2 );

//  1, 2
let replaySubscription = replaySubject.subscribe((value) => {
    console.log('replay subscription', value);
}); 
```

哇，这里发生了什么，第一个数字怎么了？

因此，在创建订阅之前发生的`.next()`通常会丢失。但是在`ReplaySubject`的情况下，我们有机会将发出的值保存在缓存中。在创建时，决定了缓存要保存两个值。

让我们说明一下这是如何工作的：

```
replaySubject.next( 3 )
let secondSubscriber( (value) => console.log(value) ) // 2,3 
```

**注意**

`.next()`操作发生的时间、缓存的大小以及订阅创建的时间都很重要。

在上面的示例中，演示了如何在构造函数中使用`bufferSize`参数。但是还存在一个`windowSize`参数，您可以指定值在缓存中保留的时间。将其设置为`null`，它将无限期保留在缓存中。

### 业务案例

在这里很容易想象到业务案例。您获取了一些数据，并希望应用程序记住最新获取的内容，您获取的内容可能只在一定时间内有效，当足够的时间过去后，您会清除缓存。

## AsyncSubject

```
let asyncSubject = new Rx.AsyncSubject();
asyncSubject.subscribe(
    (value) => console.log('async subject', value),
    (error) => console.error('async error', error),
    () => console.log('async completed')
);

asyncSubject.next( 1 );
asyncSubject.next( 2 ); 
```

看到这个我们期望 1,2 会被发出对吗？错误。

除非发生`complete()`，否则不会发出任何内容

```
asyncSubject.next( 3 )
asyncSubject.complete()

// emit 3 
```

`complete()`需要发生，无论之前的操作成功还是失败，所以

```
asyncSubject( 3 )
asyncSubject.error('err')
asyncSubject.complete()

// will emit 'err' as the last action 
```

### 业务案例

当您关心在流结束之前保留最后状态时，无论是值还是错误。因此，通常不是最后发出的状态，而是在关闭时间之前的最后状态。状态指的是值或错误。

## BehaviourSubject

这个 Subject 发出以下内容：

+   初始值

+   通常发出的值

+   最后发出的值。

方法：

```
next()
complete()
constructor([start value])
getValue() 
```

```
let behaviorSubject = new Rx.BehaviorSubject(42);

behaviorSubject.subscribe((value) => console.log('behaviour subject',value) );
console.log('Behaviour current value',behaviorSubject.getValue());
behaviorSubject.next(1);
console.log('Behaviour current value',behaviorSubject.getValue());
behaviorSubject.next(2);
console.log('Behaviour current value',behaviorSubject.getValue());
behaviorSubject.next(3);
console.log('Behaviour current value',behaviorSubject.getValue());

// emits 42
// current value 42
// emits 1
// current value 1
// emits 2
// current value 2
// emits 3
// current value 3 
```

### 业务案例

这与`ReplaySubject`非常相似。不过有一个区别，我们可以利用一个默认/初始值，在第一个值开始到达之前，我们可以首先显示它。我们可以检查最新发出的值，当然也可以监听所有已发出的内容。因此，将`ReplaySubject`视为更长期的记忆，而将`BehaviourSubject`视为具有默认行为的短期记忆。
