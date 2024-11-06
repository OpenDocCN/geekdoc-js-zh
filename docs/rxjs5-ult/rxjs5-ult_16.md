# 操作符 - 转换

# 操作符转换

这个类别的整个重点是展示如何从某些东西创建 Observables，以便它们可以与操作符很好地配合，以及无论你来自哪种构造都可以实现丰富的组合。

## from

在 Rxjs4 中存在一堆类似名称的操作符，比如 `fromArray()`、`from()`、`fromPromise()` 等。所有这些 `from()` 操作符现在都只是 `from()`。让我们看一些例子：

**旧 fromArray**

```
Rx.Observable.from([2,3,4,5]) 
```

**旧 fromPromise**

```
Rx.Observable.from(new Promise(resolve, reject) => {
  // do async work
  resolve( data )
}) 
```

## of

`of` 操作符接受 x 个参数，因此你可以用一个参数调用它，也可以用 10 个参数调用，如下所示：

```
Rx.Observable.of(1,2);
Rx.Observable.of(1,2,3,4); 
```

## to

还有一堆操作符，允许你走另一条路，即离开观察者的美好世界，回到更原始的状态，比如：

```
let promise = Rx.Observable.of(1,2).toPromise();
promise.then(data => console.log('Promise', data)); 
```
