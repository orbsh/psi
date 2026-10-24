# 种类参数只用来按构造子派发

**状态**：在用（可改）
**一句话**：`trait Functor<F<_>>` 里的 `F<_>` 接一个构造子而不是类型；构造子只能写死在 trait 声明与 impl 目标里，不推断。

**动机**：让 trait 的默认实现写一次、所有构造子继承——`as`、`map2`、`zip_with` 这类派生方法不必每个容器重写。收益只有一条，但很实在。

**形态**：

```
trait Functor<F<_>> {
  def map<A, B> self: F<A> f: Fn<A, B> -> F<B>
  def as<A, B> self: F<A> value: B -> F<B> = self.map {x| value}
}

impl Functor for List {     # 目标不带实参 = 按构造子实现
  def map<A, B> f: Fn<A, B> -> List<B> = ...
}
```

四条规则把它关在可控范围：

- **种类不推断**。构造子只写在 trait 声明与 impl 目标里；`def` 上不许出现种类参数，否则调用点得推断构造子，等于高阶合一。
- **impl 目标两种形态**：带实参的类型（`for List<Shape>`）按具体类型实现；不带实参的泛型名（`for List`）按构造子实现，后者只对带种类参数的 trait 合法。
- **构造子来自接收者的静态类型**：`xs.map f` 里 `xs: List<Int>`，先合一外层（F := List），再合一元素（A := Int），最后从 `f` 推 B——一阶合一加一次构造子替换。
- **接收者类型可显式写出**：`self: F<A>`；impl 目标是具体类型时可省略。

**代价**：v1 只支持一元构造子，`Map<K, _>` 这种部分应用没有语法，二元容器要么各自声明 `BiFunctor<F<_,_>>`，要么不用 HKT。种类参数不进推断，所以不能写 `def collapse<F<_>> xs: F<Int>`。整个机制的实现复杂度不低（合一层级 + 记录构造子），而收益只在有多个容器的时候兑现。

**冲突**：[36](36-kind-inference.md) 是「让它更强」的方向（推断构造子、支持部分应用）。

**若改掉会动到**：trait 语法、[33](33-dict-passing.md)（字典键含构造子）、[38](38-composition.md)（`Fn` 与容器派生方法）。