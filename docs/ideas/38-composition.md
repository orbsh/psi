# 组合只有三种：链、Fn、合取

**状态**：在用（可改）
**一句话**：组合的三种拓扑各不相同，不能互相替代——链是先后（复合），`Fn` 是把一段先后物化成值，trait 合取是同时具备（积）。

**动机**：说清「什么不是组合的问题」。若让 trait 层承担串联，就必须引入「携带上一段结果的能力对象」——那个东西就是 `Fn` 换个名字，还得再定义一次 `apply`。所以 trait 层的组合止于合取，串联的职责归链。

**形态**：

```
xs.map f.filter p              # 链：先后，语法层免费
let normalize = {it.trim.lower}   # Fn：把一段变换变成值
normalize.then quote           # 组合是 Fn 上的方法，不是 ∘ 运算符
trait Duplex: Read + Write     # 合取：同时具备两个面
Read + Write                   # 就地写合取
```

```
trait Fn<A, B> {
  def apply arg: A -> B
  def then next: Fn<B, C> -> Fn<A, C> = {x| next.apply (self.apply x)}
}
```

**代价**：没有 `∘` 这样的运算符，点式组合（`f.then g.then h`）比 `f ∘ g ∘ h` 长；合取没有排序语义，想表达「先 Read 后 Write 的流水」必须用链或 `Fn`，不能借 trait 组合。

**冲突**：[17](17-currying.md) 相关（偏应用会让 `Fn` 与多参函数的关系变复杂）；[16](16-methods-are-not-values.md) 决定 `Fn` 只能由块实现。

**若改掉会动到**：prelude 的 `Fn` 族、[35](35-kinds.md)（容器派生方法依赖 `Fn`）、中缀运算符清单（若要加 `∘`）。