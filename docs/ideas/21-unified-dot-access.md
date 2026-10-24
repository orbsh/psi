# 字段即方法，点只有一种读法

**状态**：在用（可改）
**一句话**：一个点访问 `x.f` 只有一种含义——取这个成员并求值。字段就是零参方法。

**动机**：让「字段改成计算方法」对调用点透明。没有 getter/setter 分裂，也没有 `.radius` 与 `.radius()` 两种写法要选，改实现不动调用点。

**形态**：`let radius: Float` 在 `Circle` 里生成零参方法 `radius`；`c.radius` 和方法调用走同一条名字解析路径，只是方法有参数时后面的步把参数补齐。

```
struct Circle {
  let radius: Float
  def area -> Float = 3.14159 * self.radius * self.radius
}

c.radius          # 零参调用，取字段
c.area            # 零参调用，算出来
```

**代价**：读 `c.radius` 无法从语法上知道它是一次内存读还是一次计算。对纯函数语言这不成问题（结果一样），但性能敏感的路径上，把字段换成方法不会被调用点发现——这是一个静默的行为改变点。另有一个形态代价：字段和方法共享名字空间，同名需要规则。

**冲突**：[16](16-methods-are-not-values.md) 是它的前提——如果方法可值，点上就有两个含义。[17](17-currying.md) 会把「零参调用」变成更难判的东西（零参是饱和还是又一次半调用？）。

**若改掉会动到**：[11](11-method-chain-primitive.md)（零参调用的读法）、[22](22-three-containers.md)、[28](28-construct-controlled.md)（字段可构造性按同一套名字空间决定）。