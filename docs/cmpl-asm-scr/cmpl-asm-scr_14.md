# 第十二章：访问者模式

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/12-visitor-pattern`](https://keleshev.com/compiling-to-assembly-from-scratch/12-visitor-pattern)

从头开始编译汇编

由 Vladimir Keleshev 编写

我们即将向我们的编译器添加更多传递：类型检查和动态类型代码生成。我们可以做的是通过新的方法扩展 AST 接口，每个传递一个。它可以看起来像这样：

```js
interface AST {
 emit(env: Environment): void;
 emitDynamic(env: Environment): void;
 typeCheck(env: TypeEnvironment): Type;
 equal(other: AST): boolean;
}
```

这完全没问题。然而，这种方式，每个传递的代码都与每个其他传递的代码交织在一起。换句话说，代码是按 AST 节点而不是按传递分组。

使用*访问者模式*，我们可以将每个传递的代码组合在一起，放在一个单独的类下。访问者模式允许我们通过间接方式解耦我们的传递与 AST。而不是为每个传递、每个 AST 节点有一个方法，我们添加一个名为`visit`的单个方法，它将操作委托给实现*访问者接口*的类。*访问者接口*对每个 AST 节点有一个方法：`visitAssert`、`visitLength`、`visitNumber`等。

```js
interface AST {
 visit<T>(v: Visitor<T>): T;
 equal(other: AST): boolean;
}

interface Visitor<T> {
 visitAssert(node: Assert): T;
 visitLength(node: Length): T;
 visitNumber(node: Number): T;
 visitBoolean(node: Boolean): T;
 visitNot(node: Not): T;
 visitEqual(node: Equal): T;
 …
}
```

每个 AST 节点通过调用相应的访问者方法实现新的`AST`接口。例如，`Assert`调用`visitAssert`，`Length`调用`visitLength`等。

```js
class Assert implements AST {
 constructor(public condition: AST) {}

 visit<T>(v: Visitor<T>) {
 return v.visitAssert(this);
 }

 equals(other: AST): boolean {…}
}

class Length implements AST {
 constructor(public array: AST) {}

 visit<T>(v: Visitor<T>) {
 return v.visitLength(this);
 }

 equals(other: AST): boolean {…}
}
```

访问者接口`Visitor<T>`是泛型的。这意味着它可以用来实现返回不同结果的传递。例如，`Visitor<AST>`生成一个`AST`节点，`Visitor<void>`可以作为副作用生成代码。

让我们将现有的代码生成传递转换为访问者。由于我们现有的`emit`方法返回`void`，我们的新访问者将实现`Visitor<void>`。我们不再有单独的`Environment`类，而是让访问者构造函数接受所有环境参数。从某种意义上说，访问者变成了环境。

```js
class CodeGenerator implements Visitor<void> {
 constructor(public locals: Map<string, number>,
 public nextLocalOffset: number) {}

 visitAssert(node: Assert) {
 node.condition.visit(this);
 emit(`  cmp r0, #1`);
 emit(`  moveq r0, #'.'`);
 emit(`  movne r0, #'F'`);
 emit(`  bl putchar`);
 }

 visitLength(node: Length) {
 node.array.visit(this);
 emit(`  ldr r0, [r0, #0]`);
 }

 …
}
```

我们将每个方法的主体，如`Assert.emit`和`Length.emit`复制到访问者方法中，如`visitAssert`和`visitLength`。

在`emit`方法中，我们曾经为内部节点递归地调用`emit`，如下所示：

```js
 emit(env: Environment): void {
 this.array.emit(env);
 emit(`  ldr r0, [r0, #0]`);
 }
```

现在，相反，我们在它们上调用`visit`方法。

```js
 visitLength(node: Length) {
 node.array.visit(this);
 emit(`  ldr r0, [r0, #0]`);
 }
```

之前`this`指的是 AST 节点，但现在节点是通过名为`node`的参数传递的。现在，`this`指的是访问者本身，我们传递它而不是`env`参数。

在我们创建新环境的一些罕见地方，我们用更新后的环境创建一个新的访问者。以下是从`visitFunction`的一个例子。

之前：

```js
 let env = new Environment(locals, -20);
 this.body.emit(env);
```

之后：

```js
 let visitor = new CodeGenerator(locals, -20);
 node.body.visit(visitor);
```

如您所见，从基于 AST 的传递转换为基于访问者的传递是一个纯粹机械的转换。我们将引入的新传递也将基于访问者模式。

下一章：第十三章 静态类型检查和推断

* * *
