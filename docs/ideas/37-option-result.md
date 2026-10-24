# 无 null，错误即值

**状态**：在用（可改）
**一句话**：没有 `null`；`Option<T>` / `Result<T, E>` 是 prelude 里的普通 enum，`match` 必须穷尽，`x?` 是 `Result` 传播记法。

**动机**：把「可能没有」和「可能失败」变成类型里的形状而不是运行时的空指针。穷尽匹配让漏掉分支成为编译错误，错误处理不再是可遗忘的旁路。

**形态**：

```
enum Option<T> { Some v: T, None }
enum Result<T, E> { Ok v: T, Err e: E }

let n = maybe?              # ? 展开成 match；不是异常
match shape {
  Shape.Circle r -> r * r
  _ -> 0.0
}
```

**代价**：

- **组合成本**：一串可能失败的操作要么写成 `?` 链（要求函数返回 `Result`），要么嵌套 `match`。后者是缩进地狱，前者要求错误类型一致（或要有转换机制）。
- **没有异常 = 没有跨层跳转**：深层回调里失败只能一层层返回，宿主边界上的错误要显式编解码。
- **错误类型设计的负担**：`Result<T, E>` 的 `E` 选谁（一个 enum？`Text`？分层的错误类型？）是每个库作者都要重复回答的问题。

**冲突**：[32](32-inheritance.md) 若成立，`null` 与可空类型会重新出现（`Shape?`）。与 [66](66-iteration-protocol.md)/[67](67-early-return.md) 相邻：`?` 是目前唯一的早退手段。

**若改掉会动到**：prelude、[67](67-early-return.md)、[62](62-byte-boundary.md)（宿主错误的编解码）。