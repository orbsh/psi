# 产物是标准 wasm 模块

**状态**：在用（可改）
**一句话**：产物是标准 wasm 模块（Wasm 3.0 GC 类型 + 少量 WASI 导入），不是私有字节码，不捆绑私有运行时。

**动机**：这是「演示格式」与「交付格式」的分界。任何符合 Wasm 3.0 的宿主都能直接实例化，不需要装 ψ 的运行时；产物能被其他工具链消费（`wasm-objdump`、bundler、CDN）。211B 的产物如果只能在自家运行时里跑，那是一个 showcase。

**形态**：全程序一个 `.wasm`，导出 `main` 与 `#[export]` 标注的方法；导入是 WASI 的一小组（stdout/stderr、时钟、文件）加宿主自定义的 `externref` 接口。

**代价**：

- **依赖宿主能力**：GC 类型 Safari 跟进晚，Wasmtime 要开 `gc` feature，部分嵌入式引擎跑不了——第二梯队运行时就是第二梯队用户。
- **调试体验**：wasm 的堆栈与源码映射靠 name section，工具链比 native 差一截。
- **放弃 native 产物**：需要 native 二进制时要么另开后端（见 [53](53-cranelift.md)），要么靠宿主嵌入 runtime。

**冲突**：[53](53-cranelift.md)（native 产物）、[55](55-own-gc.md)（私有运行时）。

**若改掉会动到**：[54](54-wasm-gc.md)、[61](61-three-hosts.md)、[58](58-whole-program.md)。