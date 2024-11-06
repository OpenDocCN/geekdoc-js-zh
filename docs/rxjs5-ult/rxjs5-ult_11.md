# 操作符首先看到

# 操作符

操作符是给 Observable 赋予力量的关键。没有它们，Observable 就一无是处。有近 60 个以上的操作符。

让我们看一些：

## of

```
let stream$ = Rx.Observable.of(1,2,3,4,5) 
```

此处我们正在使用一个创建类型的操作符为我们创建一个 Observable。这是同步的，会尽快输出值。基本上，它允许你指定以逗号分隔的列表中发出的值。

## do

```
let stream$ = 
Rx.Observable
  .of(1,2,3)
  .do((value) => {
    console.log('emits every value')
  }); 
```

这是一个非常方便的操作符，因为它用于调试你的 Observable。

## filter

```
let stream$ = 
Rx.Observable
.of(1,2,3,4,5)
.filter((value) => {
  return value % 2 === 0;
})

// 2,4 
```

这样可以阻止某些值被发出。

但请注意，我可以在一个方便的地方添加 `do` 操作符，并且仍然可以调查所有的值。

```
let stream$ = 
Rx.Observable
.of(1,2,3,4,5)
.do((value) => {
  console.log('do',value)
})
.filter((value) => {
  return value % 2 === 0;
})

stream$.subscribe((value) => {
  console.log('value', value)
})

//  do: 1,do : 2, do : 3, do : 4, do: 5 
// value : 2, 4 
```
