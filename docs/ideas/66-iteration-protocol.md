# 迭代协议与循环

**状态**：未定
**一句话**：现在只有 `while` 一种循环形式，集合遍历靠 prelude 的高阶方法（`map`/`filter`/`each`/`fold`）；`each` 与 `for` 的边界、以及提前退出要不要，还没定。

**动机**：链式方法覆盖了绝大多数遍历需求，`while` 兜底即可。少一套循环语法就少一套作用域与 break/continue 规则。但「遍历中途要跳出来」这件事，高阶方法表达不了——`each` 只跑到底。

**形态**（候选）：

```
# A：只有高阶方法 + while
xs.each {it| if bad { return } }      # 只能靠 return（见 67）

# B：加一个 for
for x in xs { if bad { break } }

# C：迭代协议 + 惰性序列
xs.iter.take_while {it.ok}.each ...
```

**代价**：

- **A** 不能中途停止，除非有「非局部退出」（抛错或 `return`），这两种都会让 `each` 的实现要处理控制流。
- **B** 引入唯一的 `break`/`continue` 与循环变量作用域，是第二条控制流语法。
- **C** 要一套迭代器协议（`next`/`done`），是个新类型族，且惰性与 [33](33-dict-passing.md) 的字典传递要配合。

**冲突**：[37](37-option-result.md)（非局部退出与 `?` 的关系）、[67](67-early-return.md)、[38](38-composition.md)。

**若改掉会动到**：prelude 的集合 API、[59](59-single-expansion-target.md)（`for` 是新的展开目标）。