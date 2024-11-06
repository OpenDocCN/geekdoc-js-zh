# 级联调用

# 级联调用

级联调用意味着基于正在发生的调用，应该进行另一个调用，可能还有一个基于那个调用的调用。

## 依赖调用

依赖调用意味着调用必须按顺序进行。调用 2 必须在调用 1 返回后发生。甚至可能需要使用调用 1 的数据来指定调用 2。

想象一下你有以下场景：

+   用户需要先登录

+   然后我们可以获取他们的用户详细信息

+   然后我们可以获取他们的订单

### 承诺方法

```
login()
     .then(getUserDetails)
     .then(getOrdersByUser) 
```

### Rxjs 方法

```
let stream$ = Rx.Observable.of({ message : 'Logged in' })
      .switchMap( result => {
         return Rx.Observable.of({ id: 1, name : 'user' })
      })
      .switchMap((user) => {
        return Rx.Observable.from(
           [ { id: 114, userId : 1 },
             { id: 117, userId : 1 }  ])
      })

stream$.subscribe((orders) => {
  console.log('Orders', orders);
})

// Array of orders 
```

在 Rxjs 示例中，我简化了这个例子，但请想象

```
Rx.Observable.of() 
```

它执行适当的 `ajax()` 调用，就像在 运算符和 Ajax 中一样

## 半依赖

+   我们可以获取用户的详细信息

+   然后我们可以并行获取订单和消息。

### 承诺方法

```
getUser()
   .then((user) => {
      return Promise.all(
        getOrders(),
        getMessages()
      )
   }) 
```

### Rxjs 方法

```
let stream$ = Rx.Observable.of({ id : 1, name : 'User' })
stream.switchMap((user) => {
  return Rx.Observable.forkJoin(
     Rx.Observable.from([{ id : 114, user: 1}, { id : 115, user: 1}],
     Rx.Observable.from([{ id : 200, user: 1}, { id : 201, user: 1}])
  )
})

stream$.subscribe((result) => {
  console.log('Orders', result[0]);
  console.log('Messages', result[1]);

}) 
```

## 坑

我们使用 `switchMap()` 而不是 `flatMap()`，这样我们就可以在必要时放弃一个 ajax 调用，这在 自动完成配方 中会更有意义
