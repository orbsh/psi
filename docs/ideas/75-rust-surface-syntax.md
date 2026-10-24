# Rust 表面语法 + 隐式区域

**状态**：候选
**一句话**：用 Rust 的语法皮（`fn`、表达式块、`match`、结构体解构、`impl`）承载隐式层级 arena——不写生命周期标记、不写 allocator 参数、不写 `&`/`*`，运行时按区域批量回收。

**动机**：语法是门面，但它决定「读起来像什么」。Rust 的那套语法让人相信内存是被认真对待的；区域模型让人不必写生命周期。合起来的样子是：写起来像 Rust，回收像 arena。

## 形态

```
fn process(data: String) -> String {
    let words: Vec<String> = data.split(" ")
    let summary = combine(words)
    if summary.is_valid() {
        CACHE.lock().push(summary.clone())   # 跨区域：clone 进 root
    }
    summary                                   # 表达式返回 = 分配在 caller 区域
}
```

## 一处必须纠正的说法

「Rust 会拒绝你返回局部引用，这门语言不拒绝你，而是帮你把它换个长寿的地方放」——这句话跳过了代价，实际发生的是两件事之一，二者不能都要：

- **静态选区域**：返回值、写入全局、被 spawn 捕获——**逃逸点在语法上是可枚举的**（就这三类），编译器在每个逃逸点做一次决定：这次分配去 caller 区域，而不是当前区域。零拷贝，但「能逃」的前提是这个决定在编译期做得出来（区域句柄一路可见，见 [56a](56a-region-param-passing.md)）。
- **运行时搬移**：在逃逸点拷贝一次（`move_to_region` / `push_and_promote`）。可行，成本是这次拷贝的字节数，但「运行时检测到跨层级赋值」是多余的——赋值语句在源码里，编译期就看得到。

所以准确的说法是：**不是「不再检查」，而是检查的对象从「每个引用的生存期」缩小成「每个逃逸点的区域归属」**。前者解析的是引用之间的关系，后者只是比一次深度（[56b](56b-depth-check.md)）。省下的是精度，不是检查。

顺带一个具体的观察：上面那段示例里的 `summary.clone()` 已经把数据搬进 root 区域了，`push_and_promote` 是冗余的。跨区域赋值要么是 clone（拷贝进目标区域），要么是编译期改分配点——不需要第三种机制。

## 代价

- **语法与 ψ 的现有记法冲突**：自由函数（[18](18-named-functions.md)）、括号逗号实参（[12](12-arity-grouping.md)）、`&`/`*` 的缺席、`Vec<T>`/`Mutex` 这类标准库形状——这是一整套不同的表面，不是换几个关键字。
- **「没有生命周期」的代价转移到签名**：区域约束仍要出现（`-> caller.X`），只是比 `&'a mut T<'b>` 少。完全不写 = 全部走默认（当前区域 / root 区域），那时局部视图类 API 写不出来。
- **它与「写起来爽」关系不大**：Rust 的痛不在 `fn` 和 `match`，在错误信息与 `&`/`&mut` 的选择。去掉这两样之后，得到的更像 OCaml/Go；保留这两样，痛就回来。

**冲突**：[11](11-method-chain-primitive.md)、[12](12-arity-grouping.md)、[13](13-infix-by-spelling.md)、[18](18-named-functions.md)、[73](73-not-general-purpose.md)（定位是脚本，不是通用系统语言）。

**若改掉会动到**：整份文法与参考件（[语法总表](../reference/syntax.md)）、[56e](56e-drop-under-arena.md)（`impl Drop` 的承诺）、[56f](56f-concurrency-region-tree.md)（`spawn` 的语法）。