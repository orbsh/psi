# 表面改成 Scala 形，只取链

**状态**：候选

**一句话**：表面取 Scala 的书写形态——花括号定块、`val`/`var`、`def name(args): R = { body }`、`match` 作表达式、`for` 推导、`impl` 兼作扩展方法——保留声明点方差与显式种类的 HKT，砍掉隐式、`infix` 修饰符与四个容器。

**动机**：链是 ψ 的第一原语（[11](11-method-chain-primitive.md)），而 Scala 是唯一把链式调用做成日常书写形态、而不是库风格的主流语言。Rust 的皮在链上有一处结构性摩擦，[75](75-rust-surface-syntax.md) 自己记过这笔账：括号把「链跨过实参继续」切断。Scala 的 `xs.map(f).filter(p)` 同样有括号，但它有两样 Rust 没有的东西——尾块实参与扩展方法，链因此可以延伸到自己没定义的类型上。

换皮的代价在 [75](75-rust-surface-syntax.md) 里算过一次：照抄 Rust 换来零学习成本。这条要重算：ψ 的目标读者不是「从 Rust 来的人」，是写游戏逻辑的人，他们的来路更可能是 Lua、C#、TypeScript，其中 TypeScript 与 Kotlin 的链式写法离 Scala 更近。

**形态**：取什么、留什么、舍什么。

取（Scala 的书写形态）：

- `val` 不可变绑定、`var` 可重绑槽，替代 `let` 与 `let mut`。
- `def name(args): R = { body }`，替代 `fn name(args) -> R { body }`。返回类型在名字之后、体之前，符合 [80](80-position-order.md) 的注记后缀规则。
- 花括号定块，语句按换行分隔，分号只在同一行写多条时用。`if (c) e else e`、`x match { case P => e }`。
- 尾块实参用花括号：`xs.filter { |s| s.area > 3.0 }`（Scala 2 形态，不用 `:` 引入）。
- `for` 推导，展开成 `map`/`flatMap`/`filter` 的链（[59](59-single-expansion-target.md) 的单一展开目标）。
- `impl T { ... }` 也能写在外来类型上，那就扩展方法；不另设 `extension` 关键字——`impl` 已经是唯一的附着动作（[22](22-three-containers.md)、[32](32-inheritance.md)）。
- 宏用 Python 装饰器记法：`@name` 或 `@name(实参)` 挂在其后的声明上，可叠、自下而上应用；宏本身是标了 `@macro` 的普通函数，签名 `Tokens -> Tokens`，编译期由编译器自带的 VM 执行。没有表达式位的宏（[27](27-macros.md)）。
- 闭包用 `|参数| 体`（Rust 形态）：`|x| x * 2`，两参 `|a, b| a + b`，零参 `|| do_thing`。`=>` 在值位只留给 `match` 的臂。
- 函数类型用 Scala 形态：`A => B`、`(A, B) => C`、`() => C`，类型位不写参数名。
- 跨行链用前置点，接收者单独一行，`.method` 逐行缩进。
- 符号名中缀，优先级由首字符挂档（[13](13-infix-by-spelling.md)）——与 Scala 同一条规则。

留（保留类型系统的两样能力，但都要一次算完）：

- **声明点方差**：类型参数标协变或逆变，声明处检查每个出现位置的符号。一次遍历，不跨声明，不迭代。
- **显式种类的 HKT**：种类写在声明上（记号待语法总表定），`F` 由实参的头部构造子一阶匹配解出，不做种类推断，不做高阶合一。不支持部分应用——需要时写具名别名（`type IntEither<A> = Either<Int, A>`），把类型 lambda 变成一次手写而不是推断。实例走字典传递（[33](33-dict-passing.md)），不走单态化枚举（[34](34-monomorphization.md)）——后者的实例枚举本身就是不动点。

`for` 推导依赖前一条：它只是 `map`/`flatMap`/`filter` 的糖，没有自己的语义。

舍（Scala 的复杂度来源）：

- 没有 `given`/`using`/隐式转换。ψ 没有隐式解析，类型类的实例一律显式传（[33](33-dict-passing.md)）。
- 没有种类推断与高阶合一（[36](36-kind-inference.md) 仍关）。
- 没有 `infix` 修饰符。用「符号名可中缀、字母名必须点」这条现成规则替代。
- 没有缩进块与 `end` 标记，块边界一律是花括号。
- 没有 case class 与 extractor 的开放 DSL，`match` 只认既有的几种形态（[68](68-pattern-coverage.md)）。
- 没有 class 与 companion object 的二分：ψ 的 struct 与 object 是同一构造（[22](22-three-containers.md)），enum 是和类型，trait 是标签集。

```psi
struct Circle {
  radius: Float
  def area(self): Float = { 3.14159 * self.radius * self.radius }
}

val shapes = [Circle { radius: 1.0 }, Circle { radius: 2.0 }]

shapes
  .filter { |s| s.area > 3.0 }
  .map { |s| s.area }

for (s <- shapes if s.area > 3.0) yield s.area

def clamp(x: Float, lo: Float, hi: Float): Float = {
  if (x < lo) lo else if (x > hi) hi else x
}

impl Shape {
  def describe(self): Text = { "area = {self.area}" }
}
```

**代价**：

- **语句边界回到换行**。块有花括号兜底，语句没有——解析器要判断「这一行写完没有」，比纯分号语言多一层规则。这是换掉缩进省下的那笔钱的利息：块免费了，行不免费。
- **方差让可变槽写起来啰嗦**。含 `var` 字段的容器，其参数必须不变；想让它协变，就得把写操作推出类型之外。这是健全性要求，不是实现选择。
- **HKT 的成本落在调用点**。每个泛型实例化要做一次头部匹配；不支持部分应用意味着 `Either<E, _>` 这类写法要落成具名别名，别名一多就会有人要求「干脆支持类型 lambda」——那条一开，高阶合一跟着进来。
- **`for` 推导的错误信息变差**。报错指向展开后的链，不是用户写的 `for`；嵌套生成器展开成 flatMap 链，调试视图与源码对不上。
- **扩展方法增加一条名字解析路径**。同一个点可能落到自己的 `impl` 或外来模块的 `impl`，同名时的优先级要为 [21](21-unified-dot-access.md) 的「点只有一种读法」扩一条规则。
- **隐式砍掉之后没有自动实例**。类型类实例要么手写、要么显式传；这在库作者那侧是负担，在读者那侧是透明。

**冲突**：[75](75-rust-surface-syntax.md) 正面冲突（同一层皮的两个答案）、[12](12-arity-grouping.md)（不分括号的实参分组）、[19](19-block-as-arg.md)（尾块站实参位，本条取花括号）、[13](13-infix-by-spelling.md)（首字符定级，两边一致）、[21](21-unified-dot-access.md)（外来 `impl` 给点访问加了第二个来源）、[35](35-kinds.md) 与 [36](36-kind-inference.md)（HKT 把「种类参数」从派发用扩成真正的构造子参数，但仍拒绝种类推断）、[57](57-linear-compile.md)（一阶匹配是一遍扫描内的匹配，不引入不动点）、[71](71-kdl-surface-syntax.md)（KDL 当表面）。

**若改掉会动到**：语法总表的每一个示例、[75](75-rust-surface-syntax.md)、[80](80-position-order.md) 的示例行、[22](22-three-containers.md)、[35](35-kinds.md)、[36](36-kind-inference.md)、[66](66-iteration-protocol.md) 的 `for`、[56e](56e-drop-under-arena.md) 与 [56f](56f-concurrency-region-tree.md) 的代码片段。