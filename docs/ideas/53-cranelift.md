# Cranelift 当后端

**状态**：候选
**一句话**：用 Cranelift 做代码生成（它自带 wasm32 后端），省掉自写发射器；将来要 native 产物时它就是现成的路。

**动机**：自写发射器是一个持续维护成本——每个新语法构造都要多写一条编码路径，调试只能在 wasm 层面看。Cranelift 已被生产验证（Wasmtime、Firefox），发射器与寄存器分配都是别人的活。

**形态**（若采用）：编译到 Cranelift IR，交给它的 wasm32 / x86 / aarch64 后端；native 产物顺便得到。

**代价**：

- **它是通用优化式 codegen**：一旦走进去，编译时间与产物体积由它的 pass 管线决定，「线性、无优化管线」这条主目标就丢了。要么关掉优化（那就是拿它的 API 当汇编器用，收益打折），要么接受它的账。
- **依赖体量**：Cranelift 是个大 crate 树，编译 ψ 编译器本身变慢，分发体积变大。
- **IR 翻译层**：从类型化 HIR 到 Cranelift IR 仍要写一层，只是比直接编码 wasm 稍高。

**冲突**：[52](52-no-optimizer.md) 正面冲突。[51](51-standard-wasm.md)（若目标只有 wasm，自写编码是可达的）。

**若改掉会动到**：[52](52-no-optimizer.md)、[57](57-linear-compile.md)、编译器仓库的依赖图。