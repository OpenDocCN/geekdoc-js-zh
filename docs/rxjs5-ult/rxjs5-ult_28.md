# 工具

# 工具

这一部分的想法是列出一些有助于编写 RxJS 代码的好工具

## RxJS 开发工具

在 github 链接中找到 [`github.com/kwintenp/rx-devtools`](https://github.com/kwintenp/rx-devtools)。README 列出了如何安装 npm/yarn 模块以及 Chrome 插件。

一个非常好的工具，可以可视化您的代码执行过程以及将要发出的值。

这里是如何在 Angular 项目中运行它的代码：

```
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/filter';
import 'rxjs/add/operator/map';
import 'rxjs/add/operator/take';

export class AppComponent {
  constructor() {
    const interval$ = Observable.interval(1000)
      .debug('test map')
      .startWith(10)
      .take(10)
      .filter((val: number) => val % 2 > 0)
      .map((val: number) => val * 2)
      .subscribe();
  }
} 
```

## RxFiddle

只需访问页面 [`rxfiddle.net/#type=editor`](http://rxfiddle.net/#type=editor) 并开始编写您的 RxJS 表达式。它会展示给您一个运行的可视化效果。再也没有比这更简单的了。
