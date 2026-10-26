# 想法目录：一份文档一个想法

这里没有「方案」，只有想法。每条想法一份文档，可以互相矛盾，也可以和已实现的东西矛盾。

## 规则

- **一条想法一份文档**。不合并、不归并成「结论」。文档之间冲突是常态，冲突在双方文档里各自登记，不设裁判。
- **没有否决态**。状态只有三种：`在用`（当前草案里这么写，随时能改）、`候选`（还没定的另一个做法）、`未定`（要什么还没想清）。
- **没有终态**。写进 `在用` 的意思只是「现在这么写」，不是「已经定了」。改一条想法只需要改那一条文档，不需要全局保持一致。
- **不写否决理由，只写代价**。代价是「这么做的开销在哪」，不是「所以不该这么做」；同一条代价在另一份文档里可以是优点。
- **示例按写作时的表面书写**。当前表面只在 [语法总表](../reference/syntax.md) 里冻结；想法文档里的片段只说明那一条想法，不与总表同步更新。
- **允许空**。想法可以先只有一句话和一句动机，形态以后再补；空着比填一个假的定论好。

## 模板

```
# 一句话标题（想法本身，不是话题）

**状态**：在用 / 候选 / 未定
**一句话**：
**动机**：
**形态**：
**代价**：
**冲突**：指向冲突的那份文档，说明冲突在哪
**若改掉会动到**：哪些文档、哪些语法/机制跟着变
```

## 目录

### 调用与表达式

- [11 调用只有一个原语](11-method-chain-primitive.md)
- [12 实参不加括号，arity 分组](12-arity-grouping.md)（候选：现在是括号加逗号）
- [13 符号名即运算符，首字定级](13-infix-by-spelling.md)（优先级表取 Rust 的，自定义符号按首字符挂档）
- [14 不设管道运算符](14-no-pipe.md) / [15 管道运算符](15-pipe-operator.md)
- [16 方法不是值，函数只有块](16-methods-are-not-values.md) / [17 自动柯里化](17-currying.md)
- [18 独立具名函数](18-named-functions.md)（语义上的自由函数；表面已有顶层 `fn`）
- [19 尾块是普通实参](19-block-as-arg.md)（候选：现在闭包写在括号里）
- [1A 字母名中缀 / 反引号 / 声明式 fixity](1A-named-infix.md)
- [75 Rust 表面语法](75-rust-surface-syntax.md)（用在：`fn`、`let mut`、括号逗号、`match`、`|x|`、`#[attr]`、`::`）
- [76 流畅性优先于简洁](76-fluency-over-terseness.md)（语法取舍的第一判据）

### 声明与模块

- [21 字段即方法，点只有一种读法](21-unified-dot-access.md)
- [22 struct / object / 模块是同一构造](22-three-containers.md)
- [23 模块是值，import 是绑定](23-modules-as-values.md) / [24 命名空间树](24-namespace-tree.md)
- [25 装饰器是编译期标签](25-decorators-as-tags.md) / [26 Python 式装饰器](26-python-decorators.md)
- [27 宏与编译期求值](27-macros.md)
- [28 构造受控：字段默认模块私有](28-construct-controlled.md)
- [29 数据先行的硬杠杆与软杠杆](29-data-first-levers.md)

### 类型

- [31 类型是形状加标签，不是树](31-tags-not-trees.md) / [32 继承与线性化](32-inheritance.md)
- [33 字典传递](33-dict-passing.md) / [34 单态化](34-monomorphization.md)
- [35 种类参数只用来按构造子派发](35-kinds.md) / [36 种类推断与高阶合一](36-kind-inference.md)
- [37 无 null，错误即值](37-option-result.md)
- [38 组合只有三种：链、Fn、合取](38-composition.md)
- [39 arity 上限](39-arity-cap.md)

### 边界数据

- [41 Json 是边界上的普通 enum](41-json-boundary.md)
- [42 Path 是段序列](42-path-segments.md) / [43 JsonPath 式查询](43-jsonpath-query.md)
- [44 Text 的表示](44-text-repr.md)

### 编译与后端

- [51 产物是标准 wasm 模块](51-standard-wasm.md)
- [52 没有优化器，没有 LLVM](52-no-optimizer.md) / [53 Cranelift 当后端](53-cranelift.md)
- [54 依赖 wasm-gc，自己不管内存](54-wasm-gc.md) / [55 自建 GC 与分配器](55-own-gc.md) / [56 层级 arena 与区域回收](56-hierarchical-arena.md)
  - [56a 区域当参数：把提升变成选句柄](56a-region-param-passing.md)
  - [56b 禁止跨层反向引用：只比深度](56b-depth-check.md)
  - [56c 闭包与 async：move 或独立区域树](56c-closure-async-region.md)
  - [56d 小 GC 兜底：两套堆，零互指](56d-hybrid-gc.md)
  - [56e 区域里的 Drop：清理链，不是逐值析构](56e-drop-under-arena.md)
  - [56f 并发：每线程一棵区域树，任务靠 scope 借](56f-concurrency-region-tree.md)
- [57 线性多遍，无不动点](57-linear-compile.md)
  - [57a 没有逃逸分析：分配点与逃逸点都在语法里](57a-no-escape-analysis.md)
- [58 全程序一个模块，没有链接器](58-whole-program.md)
- [59 所有糖展开到同一个目标](59-single-expansion-target.md)

### 运行时与宿主

- [61 一份产物，三个宿主](61-three-hosts.md)
- [62 边界只有字节](62-byte-boundary.md)
- [63 不可变值语义，无引用](63-value-semantics.md) / [64 borrow checker](64-borrow-checker.md)
- [65 核心 async](65-core-async.md)
- [66 迭代协议与循环](66-iteration-protocol.md)
- [67 早退](67-early-return.md)
- [68 模式覆盖](68-pattern-coverage.md)

### 立场

- [71 KDL 当表面语法](71-kdl-surface-syntax.md)
- [72 名字：ψ](72-name-psi.md)
- [73 不是通用语言](73-not-general-purpose.md)
- [74 非对称收益：砍能力面，换每次操作](74-asymmetric-payoff.md)

## 参考件

- [语法总表](../reference/syntax.md)：纯表格与示例，不承载判断
- [拆分前的长文](../design.md)：`docs/design.md`，保留备查，与新文档冲突时以本目录为准