# Python 式装饰器

**状态**：候选
**一句话**：`@deco` 等价于 `f = deco(f)`，装饰器是普通函数，把方法/函数值包成新的值；用户可定义任意装饰器。

**动机**：库能自己长出横切关注点——缓存、重试、计时、路由、鉴权，全在库侧写完，语言侧不需要为每个关心点开一个 `@` 语汇。这是 Python 生态活力的直接来源。

**形态**（若采用）：

```
#[retry] 3
@timed
def fetch url: Text -> Result<Text, IoError> = ...

# 展开成 fetch = timed(retry 3 fetch)
```

**代价**：

- **方法必须可值**，于是 [16](16-methods-are-not-values.md) 与 [21](21-unified-dot-access.md) 都要动。包装后的值是一个函数，调用点必须知道它是函数还是方法，或者语言接受「点上是值也可能是调用」。
- **性能与可预测性**：包一层就多一次间接调用；错误栈要穿过包装；编译期无法再假设「这个 def 就是一个函数」。
- **属性消失**：包装后的值不是方法，`#[export]` 之类的导出、文档、arity 都要重新定义。

**冲突**：[25](25-decorators-as-tags.md) 正面冲突。与 [17](17-currying.md) 同向（都要求函数是一等值）。

**若改掉会动到**：[16](16-methods-are-not-values.md)、[21](21-unified-dot-access.md)、[25](25-decorators-as-tags.md)、[27](27-macros.md)。