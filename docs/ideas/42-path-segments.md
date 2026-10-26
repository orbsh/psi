# Path 是段序列

**状态**：在用（可改）
**一句话**：`Path` 是段序列（名字或整数索引），是普通值；两个消费者是 `Json.at` 和 `Fs.read`，渲染由方法决定。

**动机**：路径能存、能传、能算，不是靠字符串拼。`let key = Path::parse("user:name")?;` 之后可以 `config.at(key)`、`keys.map(|k| k.name)`。同一份路径既能寻址 JSON 也能寻址文件系统，两个宿主共享一个类型。

**形态**：

```
let p = Path::parse("user:name")?;      // 来自外部字符串：段用 : 分，失败是 Result
let q = Path::parse("data:0:items")?;   // 整数段是索引
let r = Path::parse(":usr:local:bin")?; // 前导冒号：绝对路径
json.at("user:name:0")                  // 常见情形：at 收 Into<Path>，字符串直接传

p.os        // 用 / 连接
p.json      // 用 . 与 [i]
Path::of(["a.b"])       // 带点/空格/转义的键
```

没有路径字面量：字符串经 `Into<Path>` 隐式转，需要校验时用 `Path::parse`（返回 `Result`）。这条是表面换皮的结果——`user:name` 这种字面量与 `x: Int` 的类型标注、`Geo::pi` 的路径共用冒号，读代码时三件事要靠上下文分。

**代价**：

- **拼写错误推迟到运行期**。没有字面量就没有编译期检查：`json.at("user:nmae")` 是合法程序，取到 `Option` 的 `None` 才发现。要检查必须在定义处写 `Path::parse(...)?`，而它只能校验形状，不能校验键是否存在。
- **不归一化**：大小写、`..`、Windows 分隔符都不处理，语义留给宿主——`a:..:b` 在各宿主上不一样。
- **与查询的分界要守**：一旦加通配，路径就从「定位」变成「查询」，结果形状随段而变。

**冲突**：[43](43-jsonpath-query.md) 正面相邻（通配要不要进 Path，还是另立 `Query`）。

**若改掉会动到**：[41](41-json-boundary.md)、[62](62-byte-boundary.md)、词法（`:` 的判定）。