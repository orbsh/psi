# Rust 表面语法

**状态**：在用
**一句话**：表面照抄 Rust——`fn`、`let mut`、括号逗号、`match`、`|x|`、`#[attr]`、`::` 路径、`;` 与表达式块——语义仍是值语义的方法链语言：没有 `&`、没有 `*`、没有生命周期、没有借用检查。

**动机**：语法是门面，但门面决定读起来像什么，也决定上手时要付几遍学费。Rust 的皮有一个具体的好处：读者对「内存与错误被认真对待」的预期是现成的，而 ψ 恰好不需要为此付代价（值不可变、无引用、无生命周期）。照抄换到的是零学习成本；离开它换到的是每天的顺手。两者在同一点上打架的地方按 [76](76-fluency-over-terseness.md) 裁决——流畅性第一，零惊异第二，简洁不在判据里。

## 形态

```
struct Circle {
    radius: Float,
}

impl Circle {
    fn area(self) -> Float { 3.14159 * self.radius * self.radius }
    fn scale(self, k: Float) -> Circle { Circle { radius: self.radius * k } }
}

fn label(c: Circle) -> Text { "circle {c.radius}" }   // 顶层 fn = 模块对象的方法

fn main(args: List<Text>) -> Int {
    let mut total = 0.0;
    let shapes = shapes_of(args);
    for s in shapes { total = total + s.area; }
    shapes
        .filter(|s| s.area > 2.0)
        .map(|s| label(s))
        .each(|t| Io::stdout.println(t));
    0
}
```

完整的形状见 [语法总表](../reference/syntax.md)，那里是唯一放示例的地方。

## 照抄的部分

| 构造 | 形状 | 为什么照抄 |
|:--|:--|:--|
| 函数 | `fn name(a: A, b: B) -> R { }` | 读者零成本；返回类型后置让人先看名字与参数 |
| 绑定 | `let x: T = e;`、`let mut x`、`x = e;` | `let` 消掉绑定与赋值的读回；`mut` 让可变性显式 |
| 语句 | `;` 结束语句，块尾表达式为返回值 | 换行不再承载语义，格式化器与解析器都松绑 |
| 泛型 | 声明点 `<T: Ord>`，调用点省略 | 与 Rust 一致；推不出时用 `f::<T>(x)` |
| 路径 | `Shape::Circle`、`Geo::pi`、`Io::stdout()` | `::` 管路径、`.` 管值，两条通道各读一种 |
| 模式 | `match x { P => e, _ => e }`、`if let` | 臂用 `=>`、用逗号分隔 |
| 闭包 | `|x| x * 2`、`|acc, x| { ... }` | 与 Rust 同形；`{}` 里多语句 |
| 容器 | `struct` / `enum` / `trait` / `impl` / `dyn` | 名字与形状全同 Rust |
| 属性 | `#[export]`、`#[deprecated("理由")]`、`#[test]`、`#[target("cli")]` | 取 Rust 的写法；[25](25-decorators-as-tags.md) 的「编译期标签、不生成代码」不变 |
| 注释 | `//`、`///` | 与 Rust 同 |
| 错误传播 | `expr?` | 与 Rust 同 |
| 循环 | `while c { }`、`for x in xs { }` | `for` 是 `.each` 的糖，见下 |

## 偏离 Rust 的部分

四处，每处一条理由、一条代价。

**1. 没有 `&`、`*`、生命周期、借用检查。** 值不可变（[63](63-value-semantics.md), [64](64-borrow-checker.md)），函数按值收、按值还；跨区域赋值等于拷贝（[56a](56a-region-param-passing.md), [56b](56b-depth-check.md)）。理由：Rust 的痛不在 `fn` 和 `match`，在 `&`/`&mut` 的选择与生命周期标记；这两样在值语义里没有作用对象，照抄只会抄来噪音。代价：Rust 读者第一次写 `&x` 会报错，且「零拷贝」不再随手可得——想让大数据不搬，只能靠区域约束让分配落在调用者的区域里。

**2. 没有自由函数，但有顶层 `fn`。** `fn` 只能出现在某个容器里（`struct` / `impl` / `object` / `trait`）；文件本身是一个匿名 `object`（[22](22-three-containers.md), [23](23-modules-as-values.md)），所以文件顶层的 `fn` 在表面上与 Rust 的自由函数无法区分，语义上是那个模块对象的方法：模块内直接写 `label(c)`，模块外写 `Report::label(c)`。理由：表面零惊异，语义保住「一切调用都有接收者」（[11](11-method-chain-primitive.md)）。代价：`label(c)` 与 `x.label()` 是两种入口，读代码时仍需知道名字属于哪一类（[18](18-named-functions.md) 记的就是这笔账）。

**3. 字符串插值。** `"circle {c.radius}"`，不写 `format!`。理由：宏被成本模型否决（[27](27-macros.md)），而没有插值的字符串拼接会毁掉日志与错误信息这类最常见的写法。代价：`{}` 在 Rust 里交给 `format!` 的实参，这里直接捕获作用域内的名字与表达式——`{` 需要转义，且插值里的表达式不能跨行。

**4. 自定义符号运算符。** `a <+> b` 合法，优先级由名字的首字符挂到 Rust 的档位上（[13](13-infix-by-spelling.md)）。理由：解析要自足，不能读声明来定 fixity；首字符定级是唯一不破坏这一点的办法。代价：DSL 运算符的位置被拼写绑定，`+>>` 落在加减级，`<*>` 落在比较级。

**5. 零参不写括号。** `c.radius` 与 `c.area` 同形，`x.trim` 不加 `()`。理由：字段与零参方法在语义上是同一件事（[21](21-unified-dot-access.md), [16](16-methods-are-not-values.md)），如果写法上要区分，把字段改成计算方法就要改所有调用点——那正是 [21](21-unified-dot-access.md) 要消除的东西。两套写法还会违反 [76](76-fluency-over-terseness.md) 第 2 条。代价：Rust 读者会把 `c.area` 读成字段访问；`()` 在 ψ 里因此只表示「这次调用有实参」，读代码时要靠这一点重新建立手感。

另外四处是 Rust 没有的东西，谈不上偏离：`object`（单例容器；`mod` 是文件而不是值，[22](22-three-containers.md)）、`import` 是表达式而不是声明（模块是值，[23](23-modules-as-values.md)）、`dyn` 之外没有动态派发的细节（[33](33-dict-passing.md)）、没有宏与 `unsafe`（[27](27-macros.md)）。

## 表面与区域的关系

Rust 的皮会让人期待「编译器替我管生命周期」。这里没有那件事，但也不是「不再检查」——检查的对象从「每个引用的生存期」缩小成「每个逃逸点的区域归属」。

逃逸点在语法上可枚举，只有三类：返回值、写全局、被 spawn 捕获。编译器在这三处各做一次决定：这次分配落在调用者的区域里，还是拷贝一次搬过去。零拷贝要求区域句柄一路可见（[56a](56a-region-param-passing.md)）；拷贝的成本就是这次拷贝的字节数。不需要第三种机制——跨区域赋值要么是拷贝进目标区域，要么是编译期改了分配点。

所以准确的说法是：省下的是精度，不是检查。检查退化成每边比一次深度（`depth(指向) ≤ depth(容器)`，[56b](56b-depth-check.md)）。

## 代价

- **相似的表皮推迟了失望**。Rust 读者会预期 `&mut`、关联类型、宏、`unsafe`、精细的 trait 对象能力，这些一个都没有；撞墙的时刻被推后，但一定到来。
- **括号打断了「链跨过实参继续」**。`xs.map(f).filter(p)` 里括号把视线切断，这是 [76](76-fluency-over-terseness.md) 第 1 条与第 3 条的直接冲突，本条选第 3 条。省括号那种写法在 [12](12-arity-grouping.md) 里作为候补留着。
- **两套访问要教**：`Shape::Circle` 是路径，`c.area` 是求值；写错不会静默，但需要一次说明。
- **零参调用与名字同形**：`f` 是调用还是值，靠「函数只有闭包一种形态」这条语义分开（[16](16-methods-are-not-values.md), [18](18-named-functions.md)）——名字在值位置被读成一次零参调用，所以零参函数的返回值不能与同名绑定共存。
- **`for` 是糖**：它展开成 `.each`，所以循环体内不能构造控制流之外的返回值（[59](59-single-expansion-target.md) 只认一个展开目标）。

**冲突**：[12](12-arity-grouping.md)（不加括号 arity 分组）、[19](19-block-as-arg.md)（尾块站实参位）、[13](13-infix-by-spelling.md)（首字符挂档 vs Rust 的优先级表）、[18](18-named-functions.md)（函数不挂容器）、[21](21-unified-dot-access.md)（点只有一种读法，与 `::` 分工）、[71](71-kdl-surface-syntax.md)（KDL 当表面）。

**若改掉会动到**：语法总表的每一个示例、[11](11-method-chain-primitive.md) 的调用形态、[12](12-arity-grouping.md)、[13](13-infix-by-spelling.md) 的表、[19](19-block-as-arg.md)、[21](21-unified-dot-access.md) 的示例、[25](25-decorators-as-tags.md) 的记法、[42](42-path-segments.md) 的字面量形态、[56e](56e-drop-under-arena.md) 与 [56f](56f-concurrency-region-tree.md) 的示例、[66](66-iteration-protocol.md) 的 `for`。