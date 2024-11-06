# Observable 与 Promise

# Observable 与 Promise

让我们直接深入。我们创建了一个称为 Observable 的东西。这是一个类似于 promise 的异步构造，一旦数据到达我们就可以监听它。

```
let stream$ = Rx.Observable.from([1,2,3])

stream$.subscribe( (value) => {
   console.log('Value',value);
})

// 1,2,3 
```

处理 promise 时相应的做法是编写

```
let promise = new Promise((resolve, reject) => {
   setTimeout(()=> {
      resolve( [1,2,3] )
   })

})

promise.then((value) => {
  console.log('Value',data)
}) 
```

Promises 缺乏生成多个值的能力，重试的能力，而且与其他异步概念并不完全兼容。
