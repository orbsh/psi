# 核心 async

**状态**：候选
**一句话**：语言内建 `async`/`await`，脚本能写并发逻辑，而不是把异步全部推给宿主。

**动机**：宿主导入的异步 API（fetch、文件读、定时器）在脚本里只能靠回调或链式 `then` 拼接，写起来反人类。内建 async 让 `let r = fetch url?` 这样写，错误处理与 `?` 统一，脚本能表达真正的编排逻辑。

**形态**（若采用）：

```
def load url: Text -> Result<Json, Error> = async {
  let text = fetch url?
  Json.parse text
}
```

**代价**：

- **wasm 没有线程、没有栈切换**：async 在 wasm 上只能靠状态机变换（每个 async 函数编译成一个状态机 struct），这是 [52](52-no-optimizer.md) 里最贵的一类降级。
- **传染性**：async 函数调用的所有东西都变 async，或者要两套 API。
- **运行时**：需要一个执行器（microtask 队列），要么宿主提供，要么语言自带——后者又回到「私有运行时」。

**冲突**：[52](52-no-optimizer.md)（状态机变换是新的一层降级）、[54](54-wasm-gc.md)、[62](62-byte-boundary.md)（宿主回调怎么进 wasm）。

**若改掉会动到**：降级阶段（多一个状态机变换）、[61](61-three-hosts.md)（执行器放哪）、prelude 的 IO 方法签名。