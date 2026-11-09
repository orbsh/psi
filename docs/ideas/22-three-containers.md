# struct / object / 模块是同一构造

**状态**：在用（可改）
**一句话**：三种容器只有实例数不同——`struct` 实例任意多个，`object` 声明形状并立即绑定唯一实例，文件顶层是匿名 `object`（也就是模块）。

**动机**：一种构造三种用法，规则只写一遍。于是 `def` 在任何容器里都是同一件事：容器实例上的方法。没有自由函数，也就没有「全局函数 vs 结构体方法」的分裂——`println "hi"` 在模块里就是 `self.println "hi"`。

**形态**：

```
struct Circle {
  let radius: Float
  def area -> Float = 3.14159 * self.radius * self.radius
}

object Geo {
  let unit: Float = 1.0
  def scaled c: Circle k: Float -> Circle =
    Circle { radius: c.radius * k }
}

# 文件顶层自动是匿名 object；main 是它的方法，被导出成 wasm export
def main args: List<Text> -> Int = { ... }
```

**代价**：模块是单例，模块级 `mut` 槽就是全局可变状态——语言里唯一的状态位，且跨实例共享。`self` 在模块里的含义依赖「文件顶层是 object」这条隐式规则，初学者第一次看 `println` 不知道它挂在谁身上。另一个代价是单例与结构体在类型系统里是否同构：`object` 的类型是「只有那一个值」，而 `struct` 的类型是形状，两者若严格区分，就要一套单例规则；若不区分，`Geo` 就能被当成类型标注。

**冲突**：[18](18-named-functions.md) 若成立，顶层不再是 object，这条要被拆开。[24](24-namespace-tree.md) 是模块的另一种形态。

**若改掉会动到**：[23](23-modules-as-values.md)、[25](25-decorators-as-tags.md)（`@export` 挂在模块方法上）、[63](63-value-semantics.md)（模块级 mut 是唯一状态位）。