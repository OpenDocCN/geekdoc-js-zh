# 运算符 - 构造

# 运算符构造

## 创建

当你刚开始或者只是想测试一些东西时，你往往会从`create()`运算符开始。这需要一个带有`observer`参数的函数。这在之前的部分中已经提到过，比如 Observable Wrapping。签名看起来像下面这样：

```
Rx.Observable.create([fn]) 
```

一个示例看起来像：

```
Rx.Observable.create(observer => {
    observer.next( 1 );
}) 
```

## 范围

签名

```
Rx.Observable.range([start],[count]) 
```

示例

```
let stream$ = Rx.Observable.range(1,3)

// emits 1,2,3 
```
