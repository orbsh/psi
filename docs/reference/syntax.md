# 语法总表

纯参考件：这里只有形状，不承载判断。每条形状属于哪份想法、代价在哪，见 `docs/ideas/`。

## 声明形式

```
# 注释到行尾

let name: Text = "psi"          # 不可变槽
mut count: Int = 0              # 可重绑槽
count = count + 1               # 重绑语句：只能绑 mut 槽，是语句不是表达式

def area -> Float = ...                     # 零参方法
def scale k: Float -> Circle = ...          # 一参
def clamp lo: Float hi: Float -> Float = ...  # 两参，arity 2
def describe c: Circle -> Text = {          # 多语句 body 用块
  let side = c.radius * 2.0
  "side {side}"
}

@export("parse")                # 装饰器：声明前的编译期标签，可叠
@deprecated("用 Csv.parse 代替")
def parse line: Text -> Row = ...

struct Circle {                 # 数据形状：字段 + 固有方法
  let radius: Float
  def area -> Float = 3.14159 * self.radius * self.radius
}

enum Shape {                    # 和类型：变体带字段
  Circle radius: Float
  Rect w: Float h: Float
}

trait Area {                    # 标签：签名 + 可选默认实现
  def area -> Float
  def twice -> Float = self.area * 2.0
}

impl Area for Shape {           # 唯一的附着动作
  def area -> Float = match self {
    Shape.Circle r -> 3.14159 * r * r
    Shape.Rect w h -> w * h
  }
}

object Geo {                    # 单例：名字即那个实例
  let pi: Float = 3.14159
  def unit -> Circle = Circle { radius: 1.0 }
}

struct Box<T> {                 # 类型参数写在名字后
  let item: T
}

object Sum {
  def total<T: Add> xs: List<T> -> T = ...   # 泛型算法挂在 object 上
}

trait Functor<F<_>> {           # 种类参数：接构造子，不接类型
  def map<A, B> self: F<A> f: Fn<A, B> -> F<B>
}

impl Functor for List {         # impl 目标不带实参 = 按构造子实现
  def map<A, B> f: Fn<A, B> -> List<B> = ...
}

let Shapes = import "./shapes.psi"   # 导入：得到被导入文件的模块对象
```

## 表达式形式

```
42   3.14   true   "text"                 # 字面量
-1                                        # 负字面量：- 紧贴数字且在 primary 位
"psi {version} is {n} bytes"              # Text 插值，展开成 concat 链
[1, 2, 3]                                 # 数组字面量：定长只读
Circle { radius: 2.0 }                    # struct 字面量
{ x | x * 2 }                             # 一参块
{ acc x | acc + x }                       # 两参块
{ it.trim }                               # 单参块，参数名 it
{ do_thing }                              # 零参块

c.radius                                  # 零参调用
s.replace "a" "b"                         # 两参调用
xs.map { it.area }                        # 尾块就是普通实参
f a b                                     # 调用函数值（块），arity 分组
f (a + b)                                 # 传表达式要括号
a + b * c                                 # 中缀，按首字符定级
a <+> b                                   # 自定义符号方法的中缀形态
x?                                        # Result 传播
user:name                                 # 路径字面量：段是名字或整数
data:0:items                              # 整数段 = 索引
:usr:local:bin                            # 前导冒号 = 绝对路径

if cond { a } else { b }                  # 条件不加括号，是表达式
match shape {
  Shape.Circle r -> 3.14159 * r * r
  Shape.Rect w h -> w * h
  _ -> 0.0
}
while cond { ... }                        # 唯一已有的循环形式
```

## 类型形式

```
Int   Float   Bool   Text
List<T>   Option<T>   Result<T, E>   Map<K, V>   # 泛型实例
Json                                      # 边界上的动态值：普通 enum，不是 any
Path                                      # 段序列；字面量 user:name
[T]                                       # 数组
Fn<A, B>                                  # 函数（块）类型，多元版本按 arity
dyn Area                                  # 存在类型装箱
Read + Write                              # trait 合取
Shapes.Circle                             # 类型路径：模块对象上的投影
F<A>                                      # 只出现在带种类参数的 trait 里
```

prelude 提供 `List`、`Map`、`Option`、`Result`、`Json`、`Path` 与 `Eq`、`Ord`、`Add`、`Sub`、`Mul`、`Div` 这组 trait；`Add` 实现了 `+`，其余同理。

## 记法总则

- 调用不带括号：实参是 primary，一个一步，作用于整条前缀。
- 名字即调用：`c.area` 是求值。
- `self` 是当前容器实例；`it` 是单参块的参数；`_` 是通配模式。
- arity 由声明决定，调用必须给满；不够即报错。
- 类型实参只在声明与 impl 目标里写；调用点靠推断。
- `:` 只出现在三处：记录字段、类型标注、路径字面量。
- 中缀按首字符定级（`*` `/` `%` > `+` `-` > `= < > !` > `&` > `|` > 其余）。
- 链即管道；`?` 挂在链尾。

## 续行规则

行尾有未闭合的 `(` `[` `{`，行尾是未完成的中缀运算符或 `=`，或下一行以 `.` 开头时，语句继续；否则换行即语句结束。

## 文法

```
stmt    = item | assignment | expr
item    = decorator* decl
decorator = '@' name ( '(' expr,* ')' )?
decl    = let | mut | def | struct | enum | trait | impl | object
body    = '{' (item | variant)* '}'
assignment = name '=' expr
type    = name ( '<' type,* '>' )? | '[' type ']' | type '+' type | 'dyn' name
expr    = chain ( infix-op chain )*
chain   = primary step* '?'?
step    = '.' name | primary
primary = literal | name | path | '(' expr ')' | block | record | '[' expr,* ']'
path    = ':'? seg ( ':' seg )*
seg     = name | int
block   = '{' stmt* '}' | '{' name* '|' stmt* '}'
record  = '{' name ':' expr ( ',' name ':' expr )* '}'
```

## 一段完整程序

```
enum Shape {
  Circle radius: Float
  Rect w: Float h: Float
}

trait Area {
  def area -> Float
}

impl Area for Shape {
  def area -> Float = match self {
    Shape.Circle r -> 3.14159 * r * r
    Shape.Rect w h -> w * h
  }
}

impl Area for List<Shape> {
  def area -> Float = self.fold 0.0 {acc s| acc + s.area}
}

object Report {
  def line s: Shape -> Text = "area = {s.area}"
}

def main args: List<Text> -> Int = {
  let shapes = [
    Shape.Circle 1.0,
    Shape.Rect 2.0 3.0
  ]
  shapes
    .filter {s| s.area > 2.0}
    .map {it| Report.line it}
    .each {it| Io.stdout.println it}
  shapes.area.println
  0
}
```