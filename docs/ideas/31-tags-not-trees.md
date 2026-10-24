# 类型是形状加标签，不是树

**状态**：在用（可改）
**一句话**：类型 = 数据形状（struct / enum）+ 一组 trait 标签；`impl` 是唯一附着动作，没有任何继承链。

**动机**：树模型只有一根主轴（谁是父类），一个类型只能占一个位置；标签模型里一个类型可以同时具备任意多个面。表达式问题（加类型 vs 加操作）的两个轴都开着：加类型 = 新增 `impl`，已有 trait 不动；加 trait = 新增 `impl`，已有类型不动。唯一被迫改代码的地方是穷尽匹配——那正是要的。

**形态**：

```
struct Reading {          # product
  let label: Text
  let value: Float
}

enum Shape {              # sum
  Circle radius: Float
  Rect w: Float h: Float
}

trait Area {              # 标签：一组方法签名
  def area -> Float
  def twice -> Float = self.area * 2.0    # 默认实现：唯一的代码复用来源
}

impl Area for Shape {     # 唯一附着动作
  def area -> Float = match self {
    Shape.Circle r -> 3.14159 * r * r
    Shape.Rect w h -> w * h
  }
}
```

| 层面 | 树 | ψ |
|:--|:--|:--|
| 类型关系 | 继承链 | `impl` 附着，无链 |
| 方法解析 | 沿父链查找 | `(trait, type)` 二元组查字典 |
| 数据复用 | 继承字段 | struct 组合 + trait 默认实现 |
| 分类 | 一个类占一个位置 | 一个类型带任意多个 trait |

`trait A: B` 只表示合取约束（实现 A 的类型必须同时实现 B），不是父类型，所以没有 `super`、没有 mixin、没有线性化。`impl` 遵守 orphan rule（trait 或类型至少一个在本地），否则全程序编译也判不出该用哪份实现。

**代价**：默认实现只复用方法体、不带字段，复用粒度被限在「一段行为」，重复的字段只能靠组合再转发一遍。行为冲突只能显式点名（两个 trait 有同名方法且都给了默认实现时，实现类型必须覆盖它），没有「就近/优先」规则可依赖。另外 orphan rule 让库不能给外部类型加外部 trait 的实现，插件式扩展要另想办法。

**冲突**：[32](32-inheritance.md) 是把这条替换掉的另一种做法。与 [33](33-dict-passing.md) 和 [34](34-monomorphization.md) 是正交的两个轴（附着形态 vs 派发实现），可以任意组合，但组合起来各有代价。

**若改掉会动到**：[32](32-inheritance.md)、[33](33-dict-passing.md)、[35](35-kinds.md)（种类参数挂在 trait 上）、[41](41-json-boundary.md)（Json 用 enum 表达形状）。