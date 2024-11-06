# Observable 解剖

# Observable 解剖

Observable 的 subscribe 方法具有以下签名

```
stream.subscribe(fnValue, fnError, fnComplete) 
```

第一个在下面演示的是**fnValue**

```
let stream$ = Rx.Observable.create((observer) => {
  observer.next(1)
});

stream$.subscribe((data) => {
  console.log('Data', data);
})

// 1 
```

当调用`observer.next(<value>)`时，将调用`fnValue`。

第二个回调**fnError**是错误回调，并且通过以下代码调用，即`observer.error(<message>)`

```
let stream$ = Rx.Observable.create((observer) => {
   observer.error('error message');
})

stream$.subscribe(
   (data) => console.log('Data', data)),
   (error) => console.log('Error', error) 
```

最后我们有**fnComplete**，当流完成并且没有更多值要发出时应该调用它。它是通过调用`observer.complete()`来触发的，如下所示：

```
let stream$ = Rx.Observable.create((observer) => {
   // x calls to observer.next(<value>)
   observer.complete();
}) 
```

## 取消订阅

到目前为止，我们一直在创建一个不负责任的 Observable，不负责任是指它在完成后没有进行清理。所以让我们看看如何做到这一点：

```
let stream$ = new Rx.Observable.create((observer) => {
  let i = 0;
  let id = setInterval(() => {
    observer.next(i++);
  },1000)

  return function(){
    clearInterval( id );
  }
})

let subscription = stream$.subscribe((value) => {
  console.log('Value', value)
});

setTimeout(() => {
  subscription.unsubscribe() // here we invoke the cleanup function

}, 3000) 
```

因此请确保你

+   定义一个清理函数

+   通过调用`subscription.unsubscribe()`隐式调用该函数
