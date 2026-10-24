# Json 是边界上的普通 enum

**状态**：在用（可改）
**一句话**：`Json` 是 prelude 里的一个普通 enum（Null/Bool/Num/Str/Arr/Obj），不是类型系统里的 `any`；它不参与建模，只当边界类型。

**动机**：外部数据进来总要有地方落。给它一个有名有形的 enum，构造、匹配、派发全走普通规则，`match` 照样穷尽——比开一个万能 `any` 更诚实，因为「动态」这件事被限定在一个具体类型里，而不是污染整个类型系统。

**形态**：

```
enum Json {
  Null
  Bool b: Bool
  Num n: Float
  Str s: Text
  Arr xs: List<Json>
  Obj m: Map<Text, Json>
}

let j = Json.parse text          # Result<Json, Text>
j.render
```

它没有点访问动态键的能力（`json.user` 不存在），取键必须走路径（[42](42-path-segments.md)）。

**代价**：

- **`Num` 只能是 `Float`**：超过 2^53 的整数丢精度，ID 这类字段会坏。
- **`Obj` 用无序映射**，键序不保留，往返渲染会改文本形状。
- **取一次值就返回 `Option`**，每次都要判，比动态语言啰嗦——这是有意的（软杠杆），但对「只想读个配置」的场景很烦。
- **它是个口子**：语言说「类型必填」，这个类型又把所有类型都装进去了。

**冲突**：[43](43-jsonpath-query.md)（查询子集要动它的形状）、[44](44-text-repr.md)、[29](29-data-first-levers.md)（「类型必填」的软化处就在这里）。

**若改掉会动到**：[42](42-path-segments.md)、[62](62-byte-boundary.md)（宿主边界就是以它为中转）、[28](28-construct-controlled.md)（边界类型是否要包成不让直接构造的 newtype）。