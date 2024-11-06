# 测试

# 测试

测试异步代码通常相当棘手。异步代码可能在毫秒甚至几分钟内完成。因此，您需要一种方法来完全模拟它，就像您在 jasmine 中所做的那样。

```
spyOn(service,'method').and.callFake(() => {
    return {
        then : function(resolve, reject){
            resolve('some data')
        }
    }
}) 
```

或者更简洁的版本：

```
spyOn(service,'method').and.callFake(q.when('some data')) 
```

关键是您尝试避免整个时间问题。Rxjs 在`Rxjs 4`中历史上提供了使用具有自己内部时钟的 TestScheduler 的方法，这使��能够增加时间。这种方法有两种风味：

**方法 1**

```
let testScheduler = new TestScheduler();

// my algorithm
let stream$ = Rx.Observable
.interval(1000, testScheduler)
.take(5);

// setting up the test
let result;

stream$.subscribe(data => result = data);

testScheduler.advanceBy(1000);
assert( result === 1 )

testScheduler.advanceBy(1000);
... assert again, etc.. 
```

这种方法相当容易理解。第二种方法是使用热观察和一个`startSchedule()`方法，看起来像这样：

```
// setup the thing that outputs data
var input = scheduler.createHotObservable(
    onNext(100, 'abc'),
    onNext(200, 'def'),
    onNext(250, 'ghi'),
    onNext(300, 'pqr'),
    onNext(450, 'xyz'),
    onCompleted(500)
  );

// apply operators to it
var results = scheduler.startScheduler(
    function () {
      return input.buffer(function () {
        return input.debounce(100, scheduler);
      })
      .map(function (b) {
        return b.join(',');
      });
    },
    {
      created: 50,
      subscribed: 150,
      disposed: 600
    }
  );

//assert
collectionAssert.assertEqual(results.messages, [
    onNext(400, 'def,ghi,pqr'),
    onNext(500, 'xyz'),
    onCompleted(500)
  ]); 
```

在我看来有点难以阅读，但您仍然可以理解，您控制时间，因为您有一个`TestScheduler`来指导时间流逝的速度。

这都是 Rxjs 4，它在 Rxjs 5 中有所改变。我应该说，我即将写下的内容是一个大致的方向和一个不断变化的目标，所以本章节将会更新，但是让我们开始吧。

在 Rxjs 5 中使用了称为`Marble Testing`的东西。是的，这与 Marble Diagram 有关，即您用图形符号表示预期输入和实际输出。

第一次查看[官方文档页面](https://github.com/ReactiveX/rxjs/blob/master/doc/writing-marble-tests.md)时，我就像*现在怎么办？*。但在自己写了几个测试之后，我得出结论，这是一个相当优雅的方法。

因此，我将通过展示代码来解释它：

```
// setup
const lhsMarble = '-x-y-z';
const expected = '-x-y-z';
const expectedMap = {
    x: 1,
    y: 2,
    z : 3
};

const lhs$ = testScheduler.createHotObservable(lhsMarble, { x: 1, y: 2, z :3 });

const myAlgorithm = ( lhs ) => 
    Rx.Observable
    .from( lhs );

const actual$ = myAlgorithm( lhs$ );

//assert
testScheduler.expectObservable(actual$).toBe(expected, expectedMap);
testScheduler.flush(); 
```

让我们一部分一部分来解释

**设置**

```
const lhsMarble = '-x-y-z';
const expected = '-x-y-z';
const expectedMap = {
    x: 1,
    y: 2,
    z : 3
};

const lhs$ = testScheduler.createHotObservable(lhsMarble, { x: 1, y: 2, z :3 }); 
```

我们基本上创建了一个模式指令`-x-y-z`传递给我们的`TestScheduler`上存在的`createHotObservable()`方法。这是一个为我们做了一些繁重工作的工厂方法。将其与自己编写相比，这对应于类似于以下内容：

```
let stream$ = Rx.Observable.create(observer => {
   observer.next(1);
   observer.next(2);
   observer.next(3);
}) 
```

我们不自己做的原因是我们希望`TestScheduler`来做，因此时间按照其内部时钟流逝。还要注意，我们定义了一个预期模式和一个预期映射：

```
const expected = '-x-y-z';

const expectedMap = {
    x: 1,
    y: 2,
    z : 3
} 
```

这就是我们设置所需的内容，但为了使测试运行，我们需要将其`flush`以便`TestScheduler`在内部可以触发 HotObservable 并运行一个断言。查看`createHotObservable()`方法，我们发现它解析我们提供的 mable 模式并将其推送到列表中：

```
// excerpt from createHotObservable
 var messages = TestScheduler.parseMarbles(marbles, values, error);
var subject = new HotObservable_1.HotObservable(messages, this);
this.hotObservables.push(subject);
return subject; 
```

下一步是断言，它分为两步

1) expectObservable()

2) flush()

expect 调用基本上设置了对我们的 HotObservable 的订阅

```
// excerpt from expectObservable()
this.schedule(function () {
    subscription = observable.subscribe(function (x) {
        var value = x;
        // Support Observable-of-Observables
        if (x instanceof Observable_1.Observable) {
            value = _this.materializeInnerObservable(value, _this.frame);
        }
        actual.push({ frame: _this.frame, notification: Notification_1.Notification.createNext(value) });
    }, function (err) {
        actual.push({ frame: _this.frame, notification: Notification_1.Notification.createError(err) });
    }, function () {
        actual.push({ frame: _this.frame, notification: Notification_1.Notification.createComplete() });
    });
}, 0); 
```

通过定义一个内部的`schedule()`方法并调用它。

断言的第二部分是断言本身：

```
// excerpt from flush()
 while (readyFlushTests.length > 0) {
    var test = readyFlushTests.shift();
    this.assertDeepEqual(test.actual, test.expected);
} 
```

它最终将两个列表相互比较，`actual`和`expect`列表。

它进行了深度比较并验证了两件事，即数据发生在正确的时间`frame`上以及该帧上的值是否正确。因此，这两个列表都包含类似于这样的对象：

```
{ 
  frame : [some number],
  notification : { value : [your value] }
} 
```

这两个属性必须相等才能使断言为真。

看起来不太对吧？

## 符号

我还没有真正解释我们查看的内容：

```
-a-b-c 
```

但它实际上是有意义的。`-` 表示经过的时间段。`a` 只是一个符号。因此，实际和预期中写入的 `-` 数量很重要，因为它们需要匹配。让我们看另一个测试，这样你就能掌握它，并介绍更多符号：

```
const lhsMarble = '-x-y-z';
const expected = '---y-';
const expectedMap = {
    x: 1,
    y: 2,
    z : 3
};

const lhs$ = testScheduler.createHotObservable(lhsMarble, { x: 1, y: 2, z :3 });

const myAlgorithm = ( lhs ) => 
    Rx.Observable
    .from( lhs )
    .filter(x => x % 2 === 0 );

const actual$ = myAlgorithm( lhs$ );

//assert
testScheduler.expectObservable(actual$).toBe(expected, expectedMap);
testScheduler.flush(); 
```

在这种情况下，我们的算法包括一个 `filter()` 操作。这意味着 1、2、3 不会被发出，只有 2。看看传入模式，我们有：

```
'-x-y-z' 
```

以及预期模式

```
`---y-` 
```

这就是你清楚地看到 `-` 的数量很重要的地方。你写的每个符号，无论是 `-` 还是 `x` 等等，都发生在某个特定的时间，因此在这种情况下，当 `x` 和 `z` 由于 `filter()` 方法不会发生时，意味着我们只需在结果输出中用 `-` 替换它们，所以

```
-x-y 
```

变成

```
---y 
```

因为 `x` 不会发生。

当然还有其他有趣的符号，让我们定义诸如错误之类的事物。错误用 `#` 表示，下面是这样一个测试的示例：

```
const lhsMarble = '-#';
const expected = '#';
const expectedMap = {
};

//const lhs$ = testScheduler.createHotObservable(lhsMarble, { x: 1, y: 2, z :3 });

const myAlgorithm = ( lhs ) => 
    Rx.Observable
    .from( lhs );

const actual$ = myAlgorithm( Rx.Observable.throw('error') );

//assert
testScheduler.expectObservable(actual$).toBe(expected, expectedMap);
testScheduler.flush(); 
```

这里是另一个符号 `|` 代表完成的流：

```
const lhsMarble = '-a-b-c-|';
const expected = '-a-b-c-|';
const expectedMap = {
    a : 1,
    b : 2,
    c : 3
};

const myAlgorithm = ( lhs ) => 
    Rx.Observable
    .from( lhs );

const lhs$ = testScheduler.createHotObservable(lhsMarble, { a: 1, b: 2, c :3 });    
const actual$ = lhs$;

testScheduler.expectObservable(actual$).toBe(expected, expectedMap);
testScheduler.flush(); 
```

还有更多的符号，比如 `(ab)` 本质上表示这两个值在同一时间段被发出，等等。现在，希望你已经理解了符号如何工作的基础知识，我敦促你编写自己的测试来完全掌握它，并学习本章开头提到的官方文档页面中介绍的其他符号。

祝你测试愉快
