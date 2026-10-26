# 一份产物，三个宿主

**状态**：在用（可改）
**一句话**：同一份 `.wasm` 落进三个宿主——Rust 引擎扩展、浏览器、CLI；差异只走编译期 `#[target]`，不走运行时分支。

**动机**：一门给宿主用的脚本语言，最怕的是「三方各有一套字节码」。标准 wasm 产物让「写一次，三处跑」在产物层面成立，宿主只需要提供导入。CLI 自身也是 wasm host，不是另一条编译路径。

**形态**：

- **Rust 扩展**：宿主进程内嵌 wasm runtime，实例化脚本模块，调用导出的 `main` 或命名方法；参数序列化进线性内存，结果按同一格式回写。
- **浏览器**：`.wasm` + 几十行 JS glue：`WebAssembly.instantiateStreaming` + 导入表（`log`、`fetch`、`now`）。
- **CLI**：`psi run script.psi`，Rust 二进制 + runtime，编译与实例化在同一次进程内；stdin/stdout 是 `Text` 或 `Bytes`，参数是 `List<Text>`；没有 shell 语义（没有内建 `cd`/`ls`/环境变量，这些走宿主导入）。

**代价**：

- **三个宿主各要实现一套导入**（文件、时钟、网络、日志），language-level 的标准库其实分散在三处 glue 里。
- **CLI 没有 shell 身份**：不能 `cd`、不能管道连其他命令——这是刻意的放弃，但对从 nushell 来的人是能力倒退。
- **宿主能力即语言能力下限**：浏览器里没有文件系统，`#[target]` 让相关代码在浏览器编译时消失，同一份源码因此有「编译不过的组合」。

**冲突**：[51](51-standard-wasm.md) 是前提。[62](62-byte-boundary.md) 决定边界形态。[25](25-decorators-as-tags.md)（`#[target]` 是差异机制）。

**若改掉会动到**：[51](51-standard-wasm.md)、[62](62-byte-boundary.md)、[58](58-whole-program.md)。