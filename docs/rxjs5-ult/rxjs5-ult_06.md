# 观察者

# 观察者

请注意以下代码示例创建一个 `Observable`。

```
let stream$ = Rx.Observables.create((observer) => {
  observer.next(4);
}) 
```

`create` 方法接受一个带有输入参数 `observer` 的函数。

观察者只是一个具有以下方法 `next` `error` `complete` 的对象。

```
observer.next(1);
observer.error('some error')
observer.complete(); 
```
