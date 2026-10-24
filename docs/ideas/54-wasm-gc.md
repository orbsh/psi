# wasm-gc 作为运行时

**状态**：在用（可改）
**一句话**：一个方向是让宿主管内存——分配、回收、对象布局全由 wasm-gc 承担，语言里没有分配器和 GC 实现，运行时只剩一层固定长度 glue（内存读出、字符串视图、宿主导入转发）。

**动机**：不用 wasm-gc 就要自己实现分配器、GC、对象布局和逃逸分析——那几样正是让「小」破产的源头。走这条路，「规格小到一个人能读完」里内存这一整章被划掉。

**形态**：struct 编译成 wasm-gc struct，enum 编译成 tagged struct，闭包编译成 struct（捕获字段 + 函数索引），文本见 [44](44-text-repr.md)。

## wasm-gc 做不到的事（这一条决定方向选择）

wasm-gc 只提供「由 GC 管理的类型化对象」，它没有：

- **放置控制**：不能在指定的内存位置构造对象，也没有「在这块区域里分配」的概念。
- **显式释放**：没有 free，回收时机由 GC 决定；`struct.new` 之后无法自己擦除。
- **布局控制**：字段顺序、打包、内部指针都不在语言手里，也没有 typed memory view。
- **区域/arena**：分层 arena 要的是「bump 分配 + 整块释放」，这必须自建在 linear memory 之上（i32 基址 + 偏移），wasm-gc 一层都不提供。

所以 [56](56-hierarchical-arena.md) 那套（以及 [56a](56a-region-param-passing.md)/[56b](56b-depth-check.md)）与这条**互斥于对象模型**：走区域就自建 linear-memory 分配器，产物仍是标准 wasm 模块（[51](51-standard-wasm.md) 不受影响，只是从「GC 语言」变成「自带 arena 的语言」，形态上更接近 C 编译到 wasm）。两套堆并存的形态见 [56d](56d-hybrid-gc.md)，规则是零互指。

**代价**：

- **宿主能力成为下限**：老运行时、部分嵌入式 wasm 引擎直接跑不了（GC 已进 Wasm 3.0 标准，2025-09 完成；Chrome/Firefox 2023 默认开启，Safari 晚，Wasmtime 需开 feature）。
- **放弃对回收时机的控制**：没有确定性销毁，短生命周期优化无从谈起。
- **布局由宿主定**：字段顺序、装箱策略不可控。
- **与宿主零拷贝互操作**只在这一侧成立（GC 引用可以直传 JS）；一旦改走区域，跨界回到按字节复制。

**冲突**：[55](55-own-gc.md)、[56](56-hierarchical-arena.md) 系列（对象模型互斥）、[64](64-borrow-checker.md)（借用检查的价值在无 GC 时最大）。

**若改掉会动到**：[51](51-standard-wasm.md)、[62](62-byte-boundary.md)、[63](63-value-semantics.md)、[44](44-text-repr.md)、整个内存章节。