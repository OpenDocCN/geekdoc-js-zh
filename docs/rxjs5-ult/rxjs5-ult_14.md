# 操作符 - 一个可观察对象中的可观察对象

# 一个可观察对象中的可观察对象

很多时候，你会遇到这样一种情况，你从一个类型的可观察对象开始，然后希望它变成另一种类型。

## 例子

```
let stream$ = Rx.Observable
.of(1,2,3)
.flatMap((val) => {
  return Rx.Observable
            .of(val)
            .ajax({ url : url + }) )
            .map((e) => e.response ) 
}

stream.subscribe((val) => console.log(val))

// { id : 1, name : 'Darth Vader' }, 
// { id : 2, name : 'Emperor Palpatine' },
// { id : 3, name : 'Luke Skywalker' } 
```

所以在这里，我们有一个情况，从值 1,2,3 开始，希望它们分别对应一个 ajax 调用

--1------2-----3------> --json-- json--json -->

我们不使用`.map()`操作符的原因是

```
let stream$ = Rx.Observable
.of(1,2,3)
.map((val) => {
  return Rx.Observable
            .of(val)
            .ajax({ url : url + }) )
            .map((e) => e.response ) 
} 
```

它不会给出我们想要的结果，而是结果会是：

```
// Observable, Observable, Observable 
```

因为我们已经创建了一个可观察对象列表，所以有三个不同的流。然而，`flatMap()` 操作符能够将这三个流展平成一个称为`metastream`的流。然而，在处理 ajax 时，还有另一个有趣的操作符我们应该使用，它被称为`switchMap()`。在这里阅读更多关于它的信息 级联调用
