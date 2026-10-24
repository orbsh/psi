# 早退

**状态**：未定
**一句话**：函数体是表达式，目前唯一的「出口」是 `?`（`Result` 传播）；要不要再加一个直写的 `return`，没定。

**动机**：表达式体 + 无 `return` 是最干净的形态——所有路径都是值，没有「中途离开」这个第二类控制流。但守卫式写法（`if bad { return err }`）在没有 `return` 时只能写成嵌套 `if/else`，深度会上去。

**形态**（候选）：

```
# A：只有 ?（当前）
def parse s: Text -> Result<Doc, Error> = {
  let t = check s?
  build t
}

# B：加 return
def find xs: List<Int> p: Fn<Int, Bool> -> Option<Int> = {
  xs.each {x| if p.apply x { return Some x }}
  None
}
```

**代价**：

- **A** 的代价是嵌套，以及在闭包里无法早退（`each` 里不能停）。
- **B** 的代价是第二条出口：`return` 是语句还是表达式？块的值规则要不要改？在闭包里 `return` 返回的是闭包还是外层函数（Ruby 的非局部 return 是著名坑）？

**冲突**：[66](66-iteration-protocol.md)（`break` 与 `return` 的边界）、[37](37-option-result.md)。

**若改掉会动到**：块的值规则、[66](66-iteration-protocol.md)、语法总表的语句形式。