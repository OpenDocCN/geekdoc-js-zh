# 运算符 - 组合

# 运算符组合

有许多运算符可以让你合并来自 2 个或更多源的值，它们的作用有些不同，因此了解区别很重要。

## combineLatest

这个的签名是：

```
Rx.Observable.combineLatest([ source_1, ...  source_n]) 
```

```
let source1 = Rx.Observable.interval(100)
.map( val => "source1 " + val ).take(5);

let source2 = Rx.Observable.interval(50)
.map( val => "source2 " + val ).take(2);

let stream$ = Rx.Observable.combineLatest(
    source1,
    source2
);

stream$.subscribe(data => console.log(data));

// emits source1: 0, source2 : 0 |  source1 : 0, source2 : 1 | source1 : 1, source2 : 1, etc 
```

它的作用是从每个`source`中本质上获取最新的响应，并将其作为包含 x 个元素的数组返回。每个源一个元素。

正如你所看到的，source2 在发出 2 个值后停止发出，但能够继续发送最新发出的值。

### 业务案例

业务案例是当你对每个源的最新值感兴趣，而过去的值不那么重要时，当然你有多个想要合并的源。

## concat

签名是：

```
Rx.Observable([ source_1,... sournce_n ]) 
```

查看以下数据，很容易认为在数据发出时应该关心：

```
let source1 = Rx.Observable.interval(100)
.map( val => "source1 " + val ).take(5);

let source2 = Rx.Observable.interval(50)
.map( val => "source2 " + val ).take(2);

let stream$ = Rx.Observable.concat(
    source1, 
    source2
);

stream$.subscribe( data => console.log('Concat ' + data));

// source1 : 0, source1 : 1, source1 : 2, source1 : 3, source1 : 4
// source2 : 0, source2 : 1 
```

然而，结果是，生成的 observable 会先获取第一个源的所有值并首先发出这些值，然后获取源 2 的所有值，因此源进入`concat()`运算符的顺序很重要。

因此，如果你有一个情况，其中某个源应该优先考虑，那么这就是适合你的运算符。

## merge

这个运算符使你能够将多个流合并为一个。

```
let merged$ = Rx.Observable.merge(
    Rx.Observable.of(1).delay(500),
    Rx.Observable.of(3,2,5)
)

let observer = {
    next : data => console.log(data)
}

merged$.subscribe(observer); 
```

这个运算符的重点是将多个流合并，正如你在上面看到的，任何时间运算符，比如`delay()`都会被尊重。

## zip

```
let stream$ = Rx.Observable.zip(
    Promise.resolve(1),
    Rx.Observable.of(2,3,4),
    Rx.Observable.of(7)
);

stream$.subscribe(observer); 
```

给我们`1,2,7`

让我们看另一个例子

```
let stream$ = Rx.Observable.zip(
    Rx.Observable.of(1,5),
    Rx.Observable.of(2,3,4),
    Rx.Observable.of(7,9)
);

stream$.subscribe(observer); 
```

给我们`1,2,7`和`5,3,9`，所以它基于列连接值。它将根据最小公共分母进行操作，本例中为 2。`2,3,4`序列中的`4`被忽略。正如你从第一个例子中看到的，也可以混合不同的异步概念，比如将 Promises 与 Observables 结合，因为间隔转换会发生。

### 业务案例

如果你真的关心不同源在某个位置发出的内容。比如说所有源的第二个响应，那么`zip()`就是你的运算符。
