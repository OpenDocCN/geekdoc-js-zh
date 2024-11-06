# 热和冷可观察

# 热和冷可观察

有热和冷可观察。让我们谈谈什么是冷可观察。在冷可观察中，两个订阅者各自获得值的副本，如下所示：

```
// cold observable example
let stream$ = Rx.Observable.of(1,2,3);
//subscriber 1: 1,2,3
stream.subscribe(
   data => console.log(data),
   err => console.error(err),
   () => console.log('completed')
)

//subscriber 2: 1,2,3
stream.subscribe(
   data => console.log(data),
   err => console.error(err),
   () => console.log('completed')
) 
```

在热可观察中，订阅者在开始订阅时接收值，但更像是足球的直播流，如果你在比赛开始后 5 分钟开始订阅，你将错过前 5 分钟的比赛，并且你将从那一刻开始接收数据：

```
let liveStreaming$ = Rx.Observable.interval(1000).take(5);

liveStreaming$.subscribe( 
  data => console.log('subscriber from first minute')
  err => console.log(err),
  () => console.log('completed')
)

setTimeout(() => {
   liveStreaming$.subscribe( 
  data => console.log('subscriber from 2nd minute')
  err => console.log(err),
  () => console.log('completed')
) 
},2000) 
```

## 从冷到热 - 凯蒂·佩里模式

在上面的例子中，它实际上并不是热的，事实上，值的两个订阅者将分别收到`0,1,2,3,4`。由于这是足球比赛的直播，它并不像我们想要的那样行动，那么如何修复呢？

有两个组件需要使某物从冷到热。`publish()`和`connect()`。

```
let publisher$ = Rx.Observable
.interval(1000)
.take(5)
.publish();

publisher$.subscribe( 
  data => console.log('subscriber from first minute',data),
  err => console.log(err),
  () => console.log('completed')
)

setTimeout(() => {
    publisher$.subscribe( 
        data => console.log('subscriber from 2nd minute', data),
        err => console.log(err),
        () => console.log('completed')
    ) 
}, 3000)

publisher$.connect(); 
```

在这种情况下，我们看到第一个立即开始订阅的流的输出是`0,1,2,3,4`，而第二个流正在发出`3,4`。很明显，订阅发生的时间很重要。

## 温暖可观察

还有另一种行为类似于热可观察但在某种程度上*懒惰*的可观察。我的意思是它们实质上不会发出任何值，直到有订阅者到来。让我们比较一下热和温暖可观察

**热可观察**

```
let stream$ = Rx.Observable
.interval(1000)
.take(4)
.publish();

stream$.connect();

setTimeout(() => {
  stream$.subscribe(data => console.log(data))
}, 2000); 
```

在这里我们可以看到，热可观察将丢失第一个被发出的值，因为订阅者晚到了。

让我们将这与我们的温暖可观察进行对比

**温暖可观察**

```
let obs = Rx.Observable.interval(1000).take(3).publish().refCount();

setTimeout(() => {
    obs.subscribe(data => console.log('sub1', data));
},1100)

setTimeout(() => {
    obs.subscribe(data => console.log('sub2', data));
},2100) 
```

`refCount()`运算符确保这个可观察变得温暖，即在`sub1`订阅之前不会发出任何值。另一方面，`sub2`晚到了，也就是说该订阅接收当前所在的值，而不是从一开始的值。

因此，这个输出是

```
sub1 : 0
sub1 : 1
sub2 : 1
sub1 : 2
sub2 : 2 
```

这显示了以下情况，第一个订阅者从 0 开始。如果它是热的，它将从更高的数字开始，也就是说它会晚到一点。当第二个订阅者到来时，它不会得到 0，而是 1，显示第一个数字，表明它确实变热了。

## 自然热可观察

通常情况下，如果值立即发出而无需订阅者在场，那么某些东西被认为是热的。一个自然发生的热可观察的例子是`mousemove`。大多数其他热可观察是通过使用`publish()`和`connect()`或使用`share()`运算符将冷可观察变为热可观察。

# 共享

共享意味着使用一个有用的运算符称为`share()`。想象一下，你有以下正常的冷可观察案例：

```
let stream$ = Rx.Observable.create((observer) => {
    observer.next( 1 );
    observer.next( 2 );
    observer.next( 3 );
    observer.complete();
}).share()

stream$.subscribe(
    (data) => console.log('subscriber 1', data),
    err => console.error(err),
    () => console.log('completed')
);
stream$.subscribe(
    (data) => console.log('subscriber 2', data),
    err => console.error(err),
    () => console.log('completed')
); 
```

如果你在`observer.next(1)`上设置断点，你会注意到它被触发两次，每个订阅者一次。这是我们从冷 Observable 期望的行为。共享操作符是将某物转换为热 Observable 的另一种方式，事实上，它不仅在适当条件下将某物变为热 Observable，而且在某些条件下会回退为冷 Observable。那么这些条件是什么呢？

1) **Created as hot Observable**：当新订阅到来且订阅者数量 > 0 时，Observable 尚未完成

2) **Created as Cold Observable** 在新订阅发生之前，订阅者数量变为 0。即存在一个或多个订阅一段时间，但在新订阅发生之前被取消订阅的情况。

3) **Created as Cold Observable** 当 Observable 在新订阅之前完成

这里的关键是一个*活跃*的 Observable 仍在产生值，并且至少有一个现有的订阅者。我们可以看到，在情况 1) 中，Observable 在第二个订阅者到来之前是休眠的，而在第二个订阅者到来时突然变为热 Observable，并开始共享数据。
