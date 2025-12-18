# 第十章：基本标量数据类型

> 原文：[`keleshev.com/compiling-to-assembly-from-scratch/10-primitive-scalar-data-types`](https://keleshev.com/compiling-to-assembly-from-scratch/10-primitive-scalar-data-types)

从头开始编译汇编

由 Vladimir Keleshev

当我们提到 *标量数据类型* 时，我们指的是可以适应单个机器字的数据类型，它不是一个指针，而是自身携带一个值。我们已经有了一个用于整数数字的标量数据类型。

让我们引入一个布尔数据类型。首先，我们需要为它创建一个 AST 节点：

```js
class Boolean implements AST {
 constructor(public value: boolean) {}

 emit(env: Environment) {
 if (this.value) {
 emit(`  mov r0, #1`);
 } else {
 emit(`  mov r0, #0`);
 }
 }

 equals(other: AST): boolean {…}
}
```

我们以与整数 1 和 0 相同的方式发出它（对于 `true` 和 `false`）。

在解析器中，我们引入了新的 `true` 和 `false` 令牌，并将它们组合起来创建一个 `boolean` 解析器：

```js
let TRUE =
 token(/true\b/y).map((_) => new Boolean(true));
let FALSE =
 token(/false\b/y).map((_) => new Boolean(false));

let boolean: Parser<AST> = TRUE.or(FALSE)
```

我们可以通过添加一个 `boolean` 选项来扩展表达式语法的 `atom` 规则。在这个阶段，引入一个额外的 `scalar` 规则是好主意：

```js
scalar <- boolean / ID / NUMBER
atom <- call / scalar / LEFT_PAREN expression RIGHT_PAREN
```

然后，在将此语法作为解析器实现之后，我们将在扩展的基础语言中获得布尔值：

```js
assert(true);
assert(!false);
```

然而，它们的行为与整数 `1` 和 `0` 完全相同，所以 `assert(true == 1)` 将会成功。在静态类型中，这个比较会被编译器拒绝。在动态类型中，这将在运行时评估为 `false`。

* * *

类似地，我们可以添加其他标量，例如 `undefined`，`null`（编译为 0，就像 `false` 一样），或者字符类型，它可能编译为其 ASCII 码的整数值（尽管，JavaScript 和 TypeScript 将字符视为字符串）。

AST 构造函数签名摘要及示例

| AST 构造函数签名 | 示例 |
| --- | --- |
| `Boolean(value: boolean)` `Undefined()` `Null()` | `true` `undefined` `null` |

下一章：第十一章 数组和堆分配

* * *
