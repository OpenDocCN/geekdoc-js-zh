# 第九章：深入类

# 深入了解类

本章更深入地介绍了 TypeScript 类，包括：

+   更多关于公共和私有方法和属性的内容

+   存取器（Getters 和 Setters）

+   使用接口简化构造函数

+   继承：构建类的层次结构

+   隐藏信息：保护类数据和方法免受外部世界的影响

+   使用接口：隐藏你的实现并启用更高级别的抽象

+   抽象类：定义具有最低功能标准的模板

正如你将看到的，TypeScript 支持类的方式与 C#、Java 和其他面向类的语言基本相同。

## 公开、私有和生成的 JavaScript

上一章的公交车示例显示了公共和私有类成员（分别为 `SayRoute` 和 `myRouteNumber`）。你可以将方法和属性声明为公共或私有。下面是一个稍微复杂一点的示例，显示了一个私有方法和一个公共属性：

```
class Bus {

    private myRouteNumber: number;
    public SeatingCapacity: number; 

    private myRunCost: number;

    constructor(routeNumber: number) {
        this.myRouteNumber = routeNumber;

        // Next line is allowed - objects may invoke their own private functions
        this.myRunCost = this.calculateRunCost(30, 1.95); 
    }

    public SayRoute() {
        console.log(`My route is ${this.myRouteNumber}`);
    }

    private calculateRunCost(forDistance: number, fuelCostPerMile: number): number {
        // Logic goes here to calculate cost for this bus to run this route.
        // This function is invisible to client objects.
    }

}

const myBus = new Bus(999);
myBus.SeatingCapacity = 80;

// Edit-time error since "calculateRunCost" is private
console.log(myBus.calculateRunCost(30, 1.95)); 
```

代码定义了一个新的公共属性，`SeatingCapacity`。由于它是公共的，客户端函数既可以读取（获取）它，也可以写入（设置）它。客户端函数可能不会调用私有方法 `calculateRunCost`。但是构造函数允许调用 calculateRunCost，因为它们都属于同一个对象。

### 编译后的对象及其影响

重要的是要记住 TypeScript 最终会被编译成 JavaScript。让我们纠正 TypeScript 的语法错误并展示生成的 JavaScript。

TypeScript Bus 对象：

```
class Bus3 {

    private myRouteNumber: number;
    public SeatingCapacity: number; 

    private myRunCost: number;

    constructor(routeNumber: number) {
        this.myRouteNumber = routeNumber;
        this.myRunCost = this.calculateRunCost(30, 1.95); // Allowed - objects may invoke their own private functions
    }

    public SayRoute() {
        console.log(`My route is ${this.myRouteNumber}`);
    }

    private calculateRunCost(forDistance: number, fuelCostPerMile: number): number {
        // Logic goes here to calculate cost for this bus to run this route.
        // This function is invisible to client objects.
        return 0; // Method signature requires us to return some numeric value to avoid syntax error.
    }

}

const myBus3: Bus3 = new Bus3(999);
myBus3.SeatingCapacity = 80;
myBus3.SayRoute();
myBus3["myRunCost"] = 999; // Use bracket access to change the value of the "private" class property, myRunCost 
```

编译后的 JavaScript Bus 对象：

```
var Bus3 = (function () {
    function Bus3(routeNumber) {
        this.myRouteNumber = routeNumber;
        this.myRunCost = this.calculateRunCost(30, 1.95); // Allowed - objects may invoke their own private functions
    }
    Bus3.prototype.SayRoute = function () {
        console.log("My route is " + this.myRouteNumber);
    };
    Bus3.prototype.calculateRunCost = function (forDistance, fuelCostPerMile) {
        // Logic goes here to calculate cost for this bus to run this route.
        // This function is invisible to client objects.
        return 0; // Method signature requires us to return some numeric value to avoid syntax error.
    };
    return Bus3;
}());
var myBus3 = new Bus3(999);
myBus3.SeatingCapacity = 80;
myBus3.SayRoute();
myBus3["myRunCost"] = 999; 
```

正如你所见，TypeScript 创建了一个立即调用的函数表达式（IIFE），名为 Bus3，具有以下特征：

+   从 TypeScript 源代码发射到编译后的 JavaScript 的注释

+   名为 "Bus3" 的函数。这对应于 TypeScript 构造函数。

+   两个原型方法，"SayRoute" 和 "calculateRunCost"。

此外，它还表明 TypeScript 无法总是像 C# 和 Java 那样强制执行隐私。归根结底，你是在处理 JavaScript，你可以做 JavaScript 允许你做的任何事情。这意味着使用方括号表示法访问对象属性。

## 存取器（Getters 和 Setters）

TypeScript 提供了创建存取器的语法，通常称为 "getters" 和 "setters"。这些是看起来感觉很像属性但实际上是函数。客户端代码使用它们就像使用任何其他属性一样，下面是一个非常简单的例子：

```
class Bus4 {

    private _mySeatingCapacity: number;

    public set SeatingCapacity(val: number) { this._mySeatingCapacity = val;}
    public get SeatingCapacity() { return this._mySeatingCapacity;}

    constructor() {
    }
}

const theBus: Bus4 = new Bus4();
theBus.SeatingCapacity = 80;

console.log("Seating capacity:", theBus.SeatingCapacity); 
```

`Bus4` 类定义了一个私有属性 `_mySeatingCapacity`。它定义了一个对应的 getter 和 setter，`SeatingCapacity`。客户端代码然后通过将值设置为 80，然后在将其记录到控制台时读取值，与公共属性交互。

TypeScript 将 getters 和 setters 编译成了普通的 JavaScript 的 "Object.DefineProperty" 调用：

```
var Bus4 = (function () {
    function Bus4() {
    }
    Object.defineProperty(Bus4.prototype, "SeatingCapacity", {
        get: function () { return this._mySeatingCapacity; },
        set: function (val) { this._mySeatingCapacity = val; },
        enumerable: true,
        configurable: true
    });
    return Bus4;
}());
var theBus = new Bus4();
theBus.SeatingCapacity = 80;
console.log("Seating capacity:", theBus.SeatingCapacity); 
```

如果你只想在私有属性周围包装一个公共 getter 和 setter，那么这几乎不值得麻烦^(1) - 实际上可能会误导。这里有一个更全面的示例，展示了 getter 如何执行更有意义的计算。

```
class Bus5 {

    private _myTotalPassengers: number;
    private _myCostPerMile: number;
    private _myTotalRouteDistance: number;

    private _myRouteNumber: number;
    public get myRouteNumber() { return this.myRouteNumber; }

    private _mySeatingCapacity: number;
    public set SeatingCapacity(val: number) { this._mySeatingCapacity = val; } 

    public get CostPerRider() {
        const totalRouteCost = this._myTotalRouteDistance * this._myCostPerMile;
        const costPerRider = totalRouteCost / this._myTotalPassengers;
        return costPerRider;
    }

    constructor(routeNumber, costPerMile, totalPassengers, routeDistance) {
        this._myRouteNumber = routeNumber;
        this._myCostPerMile = costPerMile;
        this._myTotalPassengers =  totalPassengers;
        this._myTotalRouteDistance = routeDistance;
    }
}

const myBus5: Bus5 = new Bus5(148, 12.50, 72, 80);
myBus5.SeatingCapacity = 80;

console.log("My total cost per rider:", myBus5.CostPerRider)
console.log("Cost per rider with 80 riders: ", new Bus5(148, 12.50, 80, 50).CostPerRider) 
```

这个示例展示了一个 getter，`CostPerRider`。当客户端代码引用 CostPerRider 时，它通过考虑距离、每英里成本和总乘客数在运行时计算一个值，然后返回该值。

* * *

*关于访问器和性能的说明*要谨慎使用长时间运行的访问器。一些框架，比如 Angular 1.x，使用双向绑定来保持 UI 与后端数据同步。如果您在 Angular 模板中将一个字段绑定到一个长时间运行的访问器，您的用户将不会祝您生日快乐。

* * *

## 使用接口定义构造函数参数

在上一个示例中，Bus5 对象的构造函数以四个数字参数作为输入：

```
const myBus5: Bus5 = new Bus5(148, 12.50, 72, 80); 
```

很难知道这些参数的含义。当然，智能感知会帮助很多，但您需要将鼠标悬停在代码上才能获得上下文。

我们本可以创建一个空构造函数，然后使用公共访问器或属性。在这种情况下，我们最终会得��这样的代码：

```
const myBus5: Bus5 = new Bus5();
myBus5.RouteNumber = 148;
myBus5.CostPerMile = 12.50;
myBus5.TotalPassengers = 72;
myBus5.RouteDistance = 55; 
```

这种方法至少存在两个问题：

1.  它需要公共属性。这意味着它们在初始化后可以被更改，这可能导致有害的副作用。

1.  如果添加一个新的公共属性，IDE 无法轻松告诉您需要在哪里更改代码以初始化它。

考虑第二点。假设您有一个非平凡的 Bus 管理解决方案，并且在解决方案中的多个模块中实例化 Bus 对象。有一天，您意识到需要建模一个新属性，`StandardRouteTime`来记录特定公交路线从起点到终点应该花费多长时间。更新类定义很容易，同样很容易更新任何创建新公交对象实例的代码。但是，找到需要更改的每个地方可能很困难。接口有助于解决这个问题，如下面的代码所示：

```
interface Bus6Args {
    routeNumber: number;
    routeDistance: number;
    costPerMile: number;
    totalPassengers: number;
}

class Bus6 {
    private _myTotalPassengers: number;
    private _myCostPerMile: number;
    private _myTotalRouteDistance: number;

    private _myRouteNumber: number;
    public get myRouteNumber() { return this.myRouteNumber; }

    private _mySeatingCapacity: number;
    public set SeatingCapacity(val: number) { this._mySeatingCapacity = val; } 

    public get CostPerRider() {
        const totalRouteCost = this._myTotalRouteDistance * this._myCostPerMile;
        const costPerRider = totalRouteCost / this._myTotalPassengers;
        return costPerRider;
    }

    constructor(args: Bus6Args) {
        this._myRouteNumber = args.routeNumber;
        this._myCostPerMile = args.costPerMile;
        this._myTotalPassengers =  args.totalPassengers;
        this._myTotalRouteDistance = args.routeDistance;
    }
}

const myBus6: Bus6 = new Bus6(
        {routeDistance: 44,
         costPerMile: 12.50,
         routeNumber: 148, 
         totalPassengers: 72});

myBus6.SeatingCapacity = 80;

console.log("My total cost per rider:", myBus6.CostPerRider)
console.log("Cost per rider with 80 riders: ", 
    new Bus6({routeDistance: 44, routeNumber: 148, costPerMile: 12.50, totalPassengers: 80})
    .CostPerRider) 
```

这段代码至少有三个重要原因更好：

1.  清晰度

1.  长期维护

1.  更好的信息隐藏

### 清晰度

代码定义了一个接口，`Bus6Args`。然后类构造函数接受一个类型为 Bus6Args 的参数。这使我们可以编写这样一行代码：

```
const myBus6: Bus6 = new Bus6(
    {routeDistance: 44,
     costPerMile: 12.50, 
     routeNumber: 148, 
     totalPassengers: 72}); 
```

这比以下内容更容易理解：

```
var myBus6 = new Bus6(44, 12.5, 148, 72); 
```

这四个参数的作用立即显而易见。

### 长期维护

回想一下上面的场景 - 复杂的 Bus 管理系统，有许多模块，成千上万行代码，以及许多次代码实例化新的 Bus6 对象。要建模一个新属性，请按照以下简单步骤进行：

1.  更新类定义以包含新属性

1.  更新构造函数和类业务逻辑以使用该属性

1.  编译所有代码。

第一次这样做时，TypeScript 编译器会在您实例化新的 Bus 对象的每个地方报告错误，因为所有构造函数参数都将缺少新属性。这为您提供了一个全面的检查列表，列出了您需要考虑这个新属性的每个地方^(2)。

## 类和接口

许多常见的软件设计模式在接口中找到了它们最佳的实现。在像 TypeScript、C＃ 和 Java 这样的面向对象语言中，开发人员使用接口来抽象实现细节，并创建通用功能，该功能针对一系列看似不同的类而不是单个命名类。

### 类、接口和数据

到目前为止，我们已经使用接口来定义数据的“形状”。我们也可以使用接口来定义类的形状 - 所需的属性。暂时抛开公交车，想象一下产品推荐引擎。假设您有一个包含裤子、衬衫、夹克、鞋子、运动鞋等服装产品的数据库。您已经创建了一个很好的搜索屏幕，允许用户指定首选颜色和价格范围。您想要遍历所有产品，并显示满足用户偏好的任何产品。

我们可以轻松地将这些产品建模为类，如果我们小心对待，我们可以确保每个类都包含`color`和`price`属性。这样一来，我们就可以遍历这些对象的集合，并根据用户的偏好推荐它们。采用这种方法，我们可能会得到这样的模型：

```
class Shirt {
    public color: string;
    public fabricType: string;
    public price: number;
    public cut: string;
    constructor() { }
}

class Shoe {
    public color: string;
    public size: string;
    public price: number;
    constructor() {}
}

class Pants {
    public color: string;
    public inseam: number;
    public waist: number;
    public price: number;
    constructor();
} 
```

这三个类中的每个都有`color`和`price`，这使我们能够编写一些比较逻辑：

```
const allProducts: any[] = [].concat(
    new Shirt(), 
    new Shirt(), 
    new Pants(), 
    new Shoe(), 
    new Pants(), 
    new Shirt());

const Recommend = function(minPrice, maxPrice, requestedColor) {
    return allProducts.reduce(function(prev, curr) {
        if ((curr["color"] === requestedColor) ||
            (curr["price"] >= minPrice && curr["price"] <= maxPrice)) {
                return prev.concat(curr);
            }
    }, []);
}

console.log("Recommended for min/max price of 10/20 and color = blue:", 
    Recommend(10, 20, "blue")); 
```

在这段代码中，我们建立了一个随机产品数组。代码没有显示，但你可以轻松假设每个对象都初始化了适当的数据。

该代码定义了一个函数，`Recommend`，该函数通过`reduce`迭代（通过）产品集合，提取符合用户标准的产品。

这样做已经足够好了，但整体来说真的相当糟糕。有一个`any`数组。它通过括号表示法引用对象属性。如果我们在`allProducts`数组中意外地放入了一个没有颜色的产品，比如瓶装水，代码会抛出运行时错误或返回一个未定义的值。即使我们添加一个新产品，比如围巾，我们也需要非常小心地遵循预期的命名约定。例如，这会导致运行时错误：

```
class Scarf {
    public Color: string;
    public price: number;
    public length: number;
    constructor();
} 
```

它会失败，因为围巾对象中的颜色是大写的，但`Recommend`函数期望小写名称。

我们可以通过使用接口来定义所需的属性来避免所有这些麻烦：

```
interface IRecommendable {
    color: string;
    price: number;
} 
```

到目前为止，这看起来很像本书中前面讨论的接口。然而，我们也可以将接口应用于类：

```
interface IRecommendable {
    color: string;
    price: number;
}

class Scarf implements IRecommendable{
    public color: string;
    public fabricType: string;
    public price: number;
    public length: string;
    constructor() { }
} 
```

`implements`关键字告诉 TypeScript，`Scarf`对象始终至少定义了`color`和`price`属性。它们可以定义更多属性，正如你所看到的，它们确实这样做了。然而，它们至少必须定义这两个属性。

我们可以创建其他“可推荐”对象，通过这样做，我们现在可以享受一些智能感知支持。考虑这个重构后的代码：

```
interface IRecommendable {
    color: string;
    price: number;
}

class Scarf implements IRecommendable{
    public color: string;
    public fabricType: string;
    public price: number;
    public length: string;
    constructor() { }
}

// Product Displays can't be recommended so doesn't implement the interface.
class ProductDisplay {
    public name: string;
    public location: string;
    constructor() {}
}

class Sneaker implements IRecommendable {
    public color: string;
    public inseam: number;
    public waist: number;
    public price: number;
}

const allRecommendableProducts: IRecommendable[] = 
    [].concat(
        new Sneaker(), 
        new Sneaker(), 
        new Scarf(), 
        new Sneaker(), 
        new Sneaker(), 
        new Scarf());

const GetRecommended = function(minPrice, maxPrice, requestedColor) {

    return <IRecommendable> allProducts.reduce(
        function(prev: IRecommendable[], curr: IRecommendable) {
            if ((curr.color === requestedColor) ||
                (curr.price <= maxPrice && curr.price <= maxPrice)) {
                    return prev.concat(curr);
                }
        }, []);
}

const RecommendedItems = GetRecommended(10, 20, "blue");

console.log("Recommended for min/max price of 10/20 and color = blue:", Recommend(10, 20, "blue")); 
```

这段代码比之前的非接口风格方法有很多优势：

+   `allRecommendableProducts`包含一个对象集合（`IRecommendable[]`），每个对象都保证有一个`price`和`color`属性。

+   如果我们尝试添加另一个对象，比如`ProductDisplay`到该集合中，IDE 会警告我们它不符合集合对象的接口要求。这意味着我们的代码可以安全地假定对象属性是存在的。

+   我们可以在 reduce 函数内部使用点符号表示颜色和价格属性。事实上，IDE 甚至会给出有用的智能感知提示。

这里有一个展示整个过程的视频：

[`www.youtube.com/embed/97u6yaGJ1T4`](https://www.youtube.com/embed/97u6yaGJ1T4)

(如果您无法查看视频，请[尝试点击这里](https://youtu.be/97u6yaGJ1T4)或将此网址输入到您的网络浏览器中：[`youtu.be/97u6yaGJ1T4`](https://youtu.be/97u6yaGJ1T4)。)

### 类、接口和方法

除了定义数据要求，您还可以定义必需的方法。让我们在数据导出的背景下探讨这一点。您已经将一组产品建模为对象，并希望允许最终用户将这些产品导出到 Excel 电子表格中。Excel 非常适合与逗号分隔列表一起使用，因此如果您的对象可以创建一个逗号分隔的版本，那么将其导出并让 Excel 发挥其魔力就是小菜一碟。

在纯 JavaScript 中以通用方式实现这一点并不难，所以让我们稍微复杂化一下，引入一点安全性。一些对象包含敏感信息，比如`cost`，您希望根据用户的角色（例如“操作员”，“主管”，“管理员”）限制对该属性的访问。最后，我们不仅担心`cost`属性。一些产品，但不是全部，受到库存控制措施的约束。在这些情况下，我们不需要提供产品的实际库存量，而是需要显示一条消息，“联系销售”。

我们*可以*编写一个大而混乱的 CSV 生成器，通用地迭代对象属性，然后用一堆 if/then/else 语句来填充它。相反，让我们将字段级逻辑委托给产品对象本身。

这里是一个相对复杂的例子：

```
interface StandardProduct {
    name: string;
    description: string;
}

interface SecuredFieldsItem {
    GetAllowedFieldNames: (requestedByRole: string) => string[]; 
    // NOTE: requestedBy would normally be a more complex object.
}

class Fidget implements StandardProduct, SecuredFieldsItem {

    public name: string;
    public description: string;
    public inventory: number;
    public weight: number;
    public recommendedAge: number;
    public cost: number;

    constructor() {};

    public GetAllowedFieldNames(requestedByRole: string) : string[] {
        const minFields = ["name", "weight", "recommendedAge", "description", "inventory"];
        if (requestedByRole === "Price Admin") {
            return minFields.concat("cost");
        }
        return minFields;
    }
}

class HotItem implements StandardProduct, SecuredFieldsItem {
    public name: string;
    public description: string;
    public features: string[];
    public inventory: number;
    public cost: number;

    constructor() {};

    public GetAllowedFieldNames(requestedByRole: string) : string[] {
        const minFields = ["name", "description", "features"];
        let allFields = minFields;
        if (requestedByRole === "Price Admin") {
             allFields = allFields.concat("cost");
        }
        if (requestedByRole === "Inventory Admin") {
             allFields = allFields.concat("inventory");
        }
        return allFields;
    }
}

function getGeneratedCsv(forProducts: SecuredFieldsItem[], forRoleLabel: string) {
    return forProducts.reduce( (prev: string[], curr: SecuredFieldsItem) => {
        const result = getFormattedCsvRow (curr, curr.GetAllowedFieldNames(forRoleLabel));
        return prev.concat(result);
    }, []);

}

function getFormattedCsvRow(sourceItem: SecuredFieldsItem, fieldsToRetrieve: string[]): string {
    return fieldsToRetrieve.reduce( (csvFieldAsBuilt: string, currentField: string) => {
        if (csvFieldAsBuilt.length < 1) {
            return sourceItem[currentField];
        }
        return csvFieldAsBuilt + "," + sourceItem[currentField];
    }, "");
}

// Pretend that these products are initialized with real data.
const allSecurableProducts = [].concat(
    new HotItem(), 
    new Fidget(), 
    new Fidget(), 
    new HotItem());

const csvOutput = getGeneratedCsv(allProducts, "Inventory Admin"); 
```

你会注意到的第一件事是，代码定义了两个接口：`StandardProduct`和`SecuredFieldsItem`。然后，两个类（Fidget 和 HotItem）都实现了这两个接口：

`class Fidget implements StandardProduct, SecuredFieldsItem`

类可以实现多个接口。你所要做的就是像往常一样定义你的接口，然后用逗号分隔每个接口，`implement`每个接口。

看看`SecuredFieldItem`类：

```
interface SecuredFieldsItem {
    GetAllowedFieldNames: (requestedByRole: string) => string[]; 
    // NOTE: requestedBy would normally be a more complex object.
} 
```

实现`SecuredFieldsItem`接口的类必须实现一个方法，`GetAllowedFieldNames`。该方法必须接受一个字符串输入参数，并且必须返回一个字符串数组。在一个更现实的场景中，你可能会传递一些表示用户作为一个整体的对象，包括他/她的角色。这个例子使用硬编码的字符串来简化事情。

正如你所看到的，GetAllowedFieldsNames 在每个类中都有独立的实现。Fidget 只关心角色是“价格管理员”的用户。HotItem 产品对具有“库存管理员”角色的用户执行额外的检查。

`getGeneratedCsv`在每个产品上调用 GetAllowedFieldNames。注意函数签名：

`function getGeneratedCsv(forProducts: SecuredFieldsItem[], forRoleLabel: string)`

它可以遍历具有完全不同属性的不同产品，因为它们都实现了 SecuredFieldsItem 接口。因此，它们将始终有 GetAllowedFieldNames 方法可供调用。

最后，辅助函数`getFormattedCsvRow`基于当前项目和允许的字段生成适当格式的逗号分隔数据行。

## 继承

与 Java 和 C#一样，TypeScript 支持分层类结构。这允许您通过从最小的“基础”类开始然后将其扩展到新类来逐步构建复杂的类。这个新扩展的类被称为*继承*其基类的功能。 “扩展”意味着添加新的类成员（属性和/或方法）。

让我们通过美国居留模型来演示继承。在这种情况下，“居民”意味着永久或临时居住在美国的人。

所有居民都有一个姓名。他们的姓名与他们的居留类型无关。

临时居民是具有两个附加属性的居民：出生国家和他们需要离开国家的日期（即他们的签证到期日期）。

美国公民和临时居民都是一种具有一些额外属性的居民 - 他们出生的城市的名称。

基于此，我们可以推断出类层次结构如下：

```
 Resident 
                 |
    -----------------------------
    |                           |
    v                           v 
```

临时居民 美国公民

让我们展示一些代码。这是一个`Resident`：

```
class Resident {

    private _name: string;
    public get MyName() { return this._name; }

    constructor(name: string) {
        this._name = name;
    }
} 
```

这个简单的类定义了一个单一的私有属性，`_name`。只有在它首次创建时才能设置：

```
const Kelly = new Resident("Kelly"); 
```

它有一个访问器（getter）用于检索居民的姓名：

```
console.log(`Resident's name: ${Kelly.MyName}.`); 
```

这是一个临时居民的模型：

```
class TemporaryResident extends Resident {
    private _countryOfOrigin: string;
    public get MyCountryOfOrigin() { return this._countryOfOrigin; }

    private _requiredExitDate: Date;

    constructor(name: string, countryOfOrigin: string, requiredExitDate: Date) {
        super(name);
        this._countryOfOrigin = countryOfOrigin;
        this._requiredExitDate = requiredExitDate;
    }

} 
```

这个模型引入了新的语法，即`extends`关键字：

```
class TemporaryResident extends Resident { 
```

这意味着`TemporaryResident`共享与`Resident`相同的成员。在这种情况下，它是`_name`属性以及 Resident 的构造函数。

任何扩展其他类的类都必须始终通过调用`super`来调用扩展类的构造函数：

```
 constructor(name: string, countryOfOrigin: string, requiredExitDate: Date) {
        super(name);
        this._countryOfOrigin = countryOfOrigin;
        this._requiredExitDate = requiredExitDate;
    } 
```

如你所见，它不需要与扩展类具有相同的签名。`TemporaryResident`接受三个参数。它通过`super(name)`调用将其中一个参数`name`传递给其扩展类的构造函数。

让我们通过另一个模型来完善示例，即美国公民：

```
class USCitizen extends Resident {

    private _cityOfBirth: string;
    public get MyBirthCity() { return this._cityOfBirth; }

    constructor(name: string, birthCity: string) {
        super(name);

        this._cityOfBirth = birthCity;
    }
} 
```

就像`TemporaryResident`一样，`USCitizen`类共享与`Resident`相同的类成员。它使用`extend`关键字来定义其父类。`USCitizen`的构造函数调用其父类的构造函数，传递名字：`super(birthCity)`。

这是一个功能齐全的视频，详细演示了这一点：

[`www.youtube.com/embed/-P1uYVlYEc4`](https://www.youtube.com/embed/-P1uYVlYEc4)

如果你看不到视频，[请尝试点击此链接](https://youtu.be/-P1uYVlYEc4)或将此 URL 键入您的网络浏览器中：[`youtu.be/-P1uYVlYEc4`](https://youtu.be/-P1uYVlYEc4))。

## 隐藏和暴露类成员

我们已经看到了`public`关键字和`private`关键字如何保护或授予对类成员的访问权限，包括属性和方法。继承增加了一点复杂性，并且使您能够通过 public/private 以及一个新的数据控制关键字`protected`来控制对类成员的访问：

+   `public` 成员始终可以在层次结构中上下访问，并且可以从客户端（即客户端代码）外部访问。

+   `private` 成员只能从类本身访问。这意味着扩展类不能访问其父类的私有成员。

+   TypeScript 提供了一个新关键字，`protected`。受保护的成员既像公有成员又像私有成员。它们对任何外部客户端代码都是私有的。从其定义点和所有扩展的子类来看，它们是公共的。

以下代码片段应有助于澄清问题：

```
class BaseClass {
    private _myPrivateProperty: string = "No one can see me except BaseClass.";
    protected _myProtectedProperty: string = "Extended classes can see me.";
    public MyPublicProperty: string = "Anyone can see and manipulate me.";
}

class ExtendedBaseClass extends BaseClass {

    constructor() {
        super();

        // Next line would be an error since myPrivateProperty is private in BaseClass
        //this._myPrivateProperty = "xyzzy";

        // ExtendedBaseClass can access _myProtectedProperty.
        this._myProtectedProperty = "I can change this value.";

        // Public property values can always be accessed within and outside of the class.
        this.MyPublicProperty = "I can also change this value.";
    }
}

const myExtendedClass = new ExtendedBaseClass();
myExtendedClass.MyPublicProperty = "Set directly on the class via client code.";

// Error:
// myExtendedClass._myPrivateProperty = 
//      "This is not allowed since private properties cannot be read or written.";

// Error:
// myExtendedClass._myProtectedProperty = 
//      "This is also not allowed since it's protected."; 
```

代码示例显示：

+   一个类，`BaseClass`。

+   BaseClass 定义了三个成员：`_myPrivateProperty`、`_myProtectedProperty`和`MyPublicProperty`。

+   它定义了另一个类，`ExtendedBaseClass`。这扩展了 BaseClass。

+   ExtendedBaseClass 允许访问 _myProtectedProperty 和 MyPublicProperty。

+   ExtendedBaseClass *可能不* 访问 _myPrivateProperty。

+   一些客户端代码定义了一个新的 const 变量，`myExtendedBaseClass`。它保存对 ExtendedBaseClass 实例的引用。

+   客户端代码能够访问实例的`MyPublicProperty`，但无法访问私有或受保护的属性。

## 抽象类

抽象类完善了 TypeScript 对这种类型层次结构的支持。抽象类看起来和感觉像标准类，但有一个关键的例外：抽象类永远不能被实例化。如果 JavaScript 是您的第一种和主要的编程语言，这可能看起来很奇怪。然而，抽象类以及接口使开发人员能够自然而优雅地表达许多常见的软件设计模式。让我们考虑一个例子。

想象一下你正在编写一个游戏。玩家在二维地图上放置不同类型的军事基地（例如“陆军”，“海军”）。基地共享一些共同特征，比如“名称”，但在重要细节上有所不同。陆军基地由士兵组成，而海军基地由船只组成。最后，在运行时，玩家可以“激活”一个基地。这会触发基地在游戏中执行有意义的操作。以下是一种简单的建模方式：

```
interface Activatable {
    ActivateSelf: () => void;
}

class NaiveBase {
    private _myName: string;
    public get Name() { return this._myName; }
    constructor (name: string) {
        this._myName = name;
    }
}

class NaiveArmyBase extends NaiveBase implements Activatable{
    private _totalSolders: number;
    public get TotalSolders() { return this._totalSolders; }

    constructor(name: string, totalSolders: number) {
        super(name);
        this._totalSolders = totalSolders;
    }

    public ActivateSelf() {
        throw "Not yet implemented";
    }
}

class NaiveNavyBase extends NaiveBase implements Activatable {
    private _totalShips: number;
    public get TotalShips() { return this._totalShips; }

    constructor(name: string, totalShips: number) {
        super(name);
        this._totalShips = totalShips;
    }

    public ActivateSelf() {
        throw "Not yet implemented";
    }
}

const naiveArmyBase = new NaiveArmyBase("First army base", 100);
const naiveNavyBase = new NaiveNavyBase("First navy base", 3);

// This is allowed but makes no sense:
const someOtherBase = new NaiveBase("what kind of base is this?"); 
```

到目前为止，这是相当简单明了的。一个`NaiveBase`类持有一个私有属性`_myName`，并提供一个获取器来检索值。另外两个类扩展它并添加自己的属性：`NaiveArmyBase`和`NaiveNavyBase`。

陆军和海军基地类都实现了`Activatable`接口，尽管在这个例子中，每个类的`ActiveSelf()`方法只是简单地抛出异常。

这种建模方法存在一个问题：没有纯粹的香草 NaiveBase。玩家永远不会创建普通的基地，他们总是创建特定类型的基地。然而，代码没有阻止这样做。

这里还有另一个问题。这种方法迫使我们在每个类上实现`Activatable`接口。我们可以在基类上实现它，但这只会加剧第一个问题 - 现在我们在一个不应该实例化的类上实现了一个接口。

抽象类为我们解决了这个问题。以下是使用抽象类重新编写的代码：

```
interface Activatable {
    ActivateSelf: () => void;
}

abstract class AbstractBase implements Activatable{
    private _myName: string;
    public get Name() { return this._myName; }

    constructor (name: string) {
        this._myName = name;
    }

    abstract ActivateSelf(): void;
}

class ArmyBase extends AbstractBase {
    private _totalSolders: number;
    public get TotalSolders() { return this._totalSolders; }

    constructor(name: string, totalSolders: number) {
        super(name);
        this._totalSolders = totalSolders;
    }

    public ActivateSelf() {
        throw "Not yet implemented";
    }
}

class NavyBase extends AbstractBase {
    private _totalShips: number;
    public get TotalShips() { return this._totalShips; }

    constructor(name: string, totalShips: number) {
        super(name);
        this._totalShips = totalShips;
    }

    public ActivateSelf() {
        throw "Not yet implemented";
    }
}

const armyBase = new ArmyBase("First army base", 100);
const navyBase = new NavyBase("First navy base", 3);
const anotherArmyBase: Activatable = new ArmyBase("Second army base", 250);

// Compiler throws an error - abstract classes can not be instantiated:
const someOtherKindOfBase = new AbstractBase("what kind of base is this?"); 
```

这个例子介绍了`abstract`关键字。我们现在有一个抽象类`Base`。这个抽象类实现了`Activatable`接口。通过这样做，你可以看到 TypeScript 抽象功能的另一个特点：你可以将类和*类成员*标记为抽象。（事实上，如果包含任何抽象成员，必须将类标记为抽象）。`Activatable`接口需要一个`ActiveSelf`方法。然而，这个方法只对“真实”的基地 - 陆军和海军基地有意义。因此，我们��`ActivateSelf`方法本身标记为抽象：

```
 abstract ActivateSelf(): void; 
```

这个抽象的`ActivateSelf`方法满足了`Activatable`接口的要求。这很完美，因为一个普通的“基地”不能有意义地激活自己 - 只有陆军和海军基地才能这样做。同时，它强制子类实现这个方法。这有两个好处：

1.  你不能忘记做这件事，因为 IDE 和编译器不会让你这样做。

1.  由于子类实现了接口，我们可以编写代码，在需要的时候将它们的类型作为`Activatable`来利用。

抽象基类展示了另一个特性：抽象类可以定义非抽象类成员。由于每个基类都有一个名称，无论基类类型如何，定义一个具体的`_myName`属性和相关的 getter 是有意义的。子类继承这些具体类成员（属性和方法）就像它们继承具体类一样。

陆军和海军基地扩展了抽象类，就像它是一个具体类一样，使用相同的`extends`关键字。

包装示例，你会发现新建的陆军和海军基地的工作方式与简单示例中的工作方式相同：

```
const armyBase = new ArmyBase("First army base", 100);
const navyBase = new NavyBase("First navy base", 3); 
```

由于两种类型的基类都实现了 Activatable，你可以这样做：

```
const anotherArmyBase: Activatable = new ArmyBase("Second army base", 250);
const activatableNavyBase = <Activatable> navyBase; 
```

让我们在视频中把所有内容放在一起：

[`www.youtube.com/embed/ska4WEeG3pM`](https://www.youtube.com/embed/ska4WEeG3pM)

(如果你无法观看该视频，[尝试点击此链接](https://youtu.be/ska4WEeG3pM)或将此网址输入到你的网络浏览器中：[`youtu.be/ska4WEeG3pM`](https://youtu.be/ska4WEeG3pM).)

# 进一步阅读

+   我写了一篇博文，结合了联合、剩余参数和接口（封装在类中），实现了一个通用的记录器函数：[`blog.hellojs.org/simple-javascript-logger-in-typescript-demonstrating-interfaces-union-types-and-rest-parameters-6efc5aee2c97`](https://blog.hellojs.org/simple-javascript-logger-in-typescript-demonstrating-interfaces-union-types-and-rest-parameters-6efc5aee2c97)

# 摘要

上一章让你了解了一点，而这一章则打开了消防龙头。

使用接口来定义数据的形状和类的形状。在这种情况下，“形状”意味着必需的类成员（方法和属性）。

类*实现*接口。类可以实现多个接口。

TypeScript 允许你创建层次结构。一个类可以*扩展*另一个类，而且它本身也可以被扩展。给定的类只能扩展另一个类。

一种特殊类型的类，*抽象类*，永远不能被实例化，但在其他方面看起来和非抽象类一样。抽象类可以（并且经常会）实现接口，它们甚至可以定义具体成员（属性和方法）。

我们已经快要结束了类的学习。下一章将提供类的最终解释，以及引入 TypeScript 提供的最终的类型 - 泛型。

* * *

> ¹. 当你想要将属性提供给客户端代码但不想让该客户端代码编辑值时，Getter 访问器就很有用。在这种情况下，你会定义一个私有变量以及它自己的 Getter 访问器，但没有 Setter 访问器。不要创建一个私有变量，然后将其与公共 Getter *和* Setter 配对。在这种情况下，你可以直接将其保持为公共的。↩
> 
> ². 既然你仍然在这个地方阅读，可以安全地说你对 TypeScript 非常有用。如果你仍然犹豫不决，考虑一下你将如何用纯 JavaScript 解决这个问题。如果你需要做这样的变化，要实现这样的效果就会更加困难，因为你无法得到同样优秀的工具支持。你无法强制出现语法错误。你必须依赖全局搜索和/或查找/替换。不是很有趣。↩
