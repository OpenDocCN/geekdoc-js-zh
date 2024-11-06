# 第八章：介绍类

# 介绍类

TypeScript 提供了对类的支持。类在许多面向对象的语言中作为基础组件。宽泛地定义，类是一组数据和函数，这些函数（通常）操作这些数据。数据可能在定义它的类之外可访问，也可能不可访问。同样，类函数可能在其包含类之外可用，也可能不可用。您可以做出这些决定。

您可以将类视为定义功能模板。这就是“数据和函数”部分。在运行时，我们创建类的*实例*，通常称为“对象”。我们经常以“传递消息”或“在对象上调用函数”来思考。

开发人员使用类来模拟人员、地点、业务实体和概念 - 各种各样的事物。这里有一个简单的示例，开始模拟可能用于公共交通的公交车：

```
class Bus {
    private myRouteNumber: number;
    constructor(routeNumber: number) {
        this.myRouteNumber = routeNumber;
    }
    public SayRoute() {
        console.log(`My route is ${this.myRouteNumber}`);
    }
} 
```

使用关键字`class`后跟类名。

公共交通管理部门通常为公交车分配路线编号。`Bus`类模拟了一个名为`myRouteNumber`的`private`字段来表示路线编号。

`constructor`是一个函数，当客户端代码实例化 Bus 对象时运行。如您所见，构造函数可以接受参数，在这种情况下，构造函数初始化了 Bus 的路线编号。

我们的业务规则规定公交车必须知道如何“说”它们的名字。一个名为`SayRoute`的函数通过在控制台上列出公交车的路线号来满足要求。

TypeScript 引入了一些新术语来描述类^(2)：

+   我们通常将 myRouteNumber、构造函数和 SayRoute 称为*类成员*。

+   myRouteNumber 是一个*属性*。

+   *constructor*是一个特殊函数，每次代码创建 Bus 的新实例时都会运行。它每个对象实例化只运行一次，但每次创建新的 Bus 对象时都会运行。

+   SayRoute 是一个*方法*。

类本身什么也不做。它们很像饼干模板 - 你可以知道饼干会是什么样子，但在有了饼干面团之前你没有饼干。我们创建新对象如下所示：

```
const myBeloved148 = new Bus(148);
const theDreaded164 = new Bus(164);

myBeloved148.SayRoute();
theDreaded164.SayRoute(); 
```

上面的代码声明了`Bus`对象的两个实例，“myBeloved148”（超级快车）和“theDreaded164”（超级本地）。然后在每个实例上调用了`SayRoute`方法。

当我们使用`new`关键字创建一个新对象时，我们正在*实例化*对象。有些人喜欢说“newing it up”。这里有一个展示基础知识的视频：

## 保护您的类数据和方法

您无疑注意到了一对互补的描述符，`private`和`public`。Bus 类声明了一个*private*成员，myRouteNumber。私有成员（即属性和方法）只能在对象内部引用或调用。公共成员和方法可以在类内部引用，也可以被客户端代码引用。这意味着以下代码将无法编译：

```
class Bus {
    private myRouteNumber: number;
    constructor(routeNumber: number) {
        this.myRouteNumber = routeNumber;
    }
    public SayRoute() {
        console.log(`My route is ${this.myRouteNumber}`);
    }
}
const myBus = new Bus(999);
myBus.myRouteNumber = 1234; // Throws an error, "Property 'myRouteNumber' is private and only accessible within the class 'Bus'" 
```

就像语言的其他部分一样，优秀的 TypeScript IDE 提供智能提示来帮助您正确定位和使用公共和私有方法。这里有一个简短的视频演示了这一点：

[`www.youtube.com/embed/QOvFFz1lRJM`](https://www.youtube.com/embed/QOvFFz1lRJM)

（如果您无法观看视频，请[在此处尝试访问](https://youtu.be/QOvFFz1lRJM)或将此 URL 键入您的网络浏览器中：[`youtu.be/QOvFFz1lRJM`](https://youtu.be/QOvFFz1lRJM)）。

TypeScript 类的深入程度远不止于此。下一章会深入探讨这一点。

* * *

> ¹. 这些短语，“传递消息”或“调用函数”基本上意思相同。有时将对象视为活生生的实体可能会有所帮助。这种范式很适合“传递消息”的概念。↩
> 
> ². 更准确地说，它可能是从其他语言借鉴了术语。↩
