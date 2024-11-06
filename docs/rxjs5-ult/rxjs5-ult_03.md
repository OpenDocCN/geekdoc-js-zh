# 安装和设置

# 安装和设置

本章内容取自官方文档，但我试图为您提供更多信息，同时将所有内容放在一个地方也很好。[官方文档](https://github.com/ReactiveX/rxjs)

TH Rxjs 库可以以许多不同的方式使用，即`ES6`，`CommonJS`和`ES5/CDN`。

## ES6

### 安装

```
 npm install rxjs 
```

### 设置

```
 import Rx from 'rxjs/Rx';

Rx.Observable.of(1,2,3) 
```

### 注意事项

这个语句`import Rx from 'rxjs/Rx'`利用了整个库。这对于测试各种功能很好，但一旦进入生产阶段，这是*一个坏主意*，因为 Rxjs 是一个相当庞大的库。在更现实的情况下，您会希望使用下面的替代方法，只导入您实际使用的运算符：

```
 import { Observable } from 'rxjs/Observable';
import 'rxjs/add/observable/of';
import 'rxjs/add/operator/map';

let stream$ = Observable.of(1,2,3).map(x => x + '!!!'); 

stream$.subscribe((val) => {
  console.log(val) // 1!!! 2!!! 3!!!
}) 
```

## CommonJS

与 ES6 相同的安装

### 安装

```
 npm install rxjs 
```

### 设置

设置虽然不同

下面再次展示了贪婪导入，这对于测试很好，但对于生产不好。

```
var Rx = require('rxjs/Rx');

Rx.Observable.of(1,2,3); // etc 
```

这里更好的方法是：

```
 let Observable = require('rxjs/Observable').Observable;
// patch Observable with appropriate methods
require('rxjs/add/observable/of');
require('rxjs/add/operator/map');

Observable.of(1,2,3).map((x) => { return x + '!!!'; }); // etc 
```

正如您所看到的`require('rxjs/Observable')`只给我们提供了 Rx 对象，我们需要深入一级才能找到 Observable。

注意，我们只需`require('path/to/operator')`即可获取我们想要在应用程序中导入的运算符。

## CDN 或 ES5

如果我既不是 ES6 也不是 CommonJS，还有另一种方法，即：

```
 <script src="https://unpkg.com/rxjs/bundles/Rx.min.js"></script> 
```

请注意，这将为您提供完整的库。由于您是外部引用它，因此不会影响捆绑大小。
