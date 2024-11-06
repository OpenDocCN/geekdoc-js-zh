# 运算符 - 数学

# 运算符 - 数学

## 最大值

```
let stream$ = Rx.Observable.of(5,4,7,-1)
.max(); 
```

这会输出`7`。这个运算符的作用很明显，它给我们一个值，即最大值。不过调用它的方式有不同。你可以给它一个比较器：

```
function comparer(x,y) {
    if( x > y ) {
        return 1;
    } else if( x < y ) {
        return -1;
    } else return 0;
}

let stream$ = Rx.Observable.of(5,4,7,-1)
.max(comparer); 
```

在这种情况下，我们定义了一个`comparer`，它在幕后运行一个排序算法，我们所要做的就是帮助它确定何时*大于*、*等于*或*小于*。或者对于一个对象，思想是一样的：

```
function comparer(x,y) {
    if( x.age > y.age ) {
        return 1;
    } else if( x.age < y.age ) {
        return -1;
    } else return 0;
}

let stream$ = Rx.Observable.of({ name : 'chris', age : 37 }, { name : 'chross', age : 32 })
.max(comparer); 
```

因为我们在`comparer`中告诉它要比较哪个属性，所以最终结果就是第一个条目。

## 最小值

最小值与`max()`运算符几乎完全相同，但返回相反的值，即最小值。

## 总和

`sum()`作为运算符已经不存在了，但我们可以用`reduce()`来代替，就像这样：

```
let stream$ = Rx.Observable.of(1,2,3,4)
.reduce((accumulated, current) => accumulated + current ) 
```

这也可以应用于对象，只要我们定义`reduce()`函数应该做什么，就像这样：

```
let objectStream$ = Rx.Observable.of( { name : 'chris' }, { age : 11 } )
.reduce( (acc,curr) => Object.assign({}, acc,curr )); 
```

这将把对象部分连接成一个对象。

## 平均值

`average()`运算符在 Rxjs5 中不再存在，但你仍然可以用`reduce()`来实现相同的功能

```
let stream$ = Rx.Observable.of( 3, 6 ,9 )
.map( x => { return { sum : x, counter : 1 } } )
.reduce( (acc,curr) => {
    return Object.assign({}, acc, { sum : acc.sum + curr.sum ,counter : acc.counter + 1  }) 
})
.map( x => x.sum / x.counter ) 
```

我承认，这个让我有点头疼，一旦你破解了最初的`map()`调用，`reduce()`就很简单了，而`Object.assign()`像往常一样是一个不错的伙伴。
