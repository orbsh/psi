# 语法总表

纯参考件：这里只有形状，不承载判断。每条形状属于哪份想法、代价在哪，见 `docs/ideas/`。

## 声明形式

```
// 注释到行尾；/// 是文档注释

val name: Text = "psi"           // 不可变绑定
var count: Int = 0               // 可重绑槽
count = count + 1                // 赋值语句，只能赋 var 槽

def area(c: Circle): Float = { 3.14159 * c.radius * c.radius }   // 顶层 def 是模块对象的方法
def clamp(x: Float, lo: Float, hi: Float): Float = { ... }        // 三参：括号逗号

struct Circle {                  // 数据形状：字段 + 方法
    radius: Float
}

impl Circle {
    def area(self): Float = { 3.14159 * self.radius * self.radius }
    def scale(self, k: Float): Circle = { Circle { radius: self.radius * k } }
    def clamp(self, lo: Float, hi: Float): Circle = { ... }
}

enum Shape {                     // 和类型：变体带命名字段
    Circle { radius: Float }
    Rect { w: Float, h: Float }
    Empty
}

trait Area {                     // 标签：签名 + 可选默认实现
    def area(self): Float
    def twice(self): Float = { self.area * 2.0 }
}

impl Area for Shape {            // 唯一的附着动作
    def area(self): Float = {
        self match {
            case Shape::Circle { radius } => 3.14159 * radius * radius
            case Shape::Rect { w, h } => w * h
            case Shape::Empty => 0.0
        }
    }
}

object Geo {                     // 单例：名字即那个实例
    val pi: Float = 3.14159
    def unit(): Circle = { Circle { radius: 1.0 } }
}

struct Box<+T> {                 // 类型参数写在名字后；+ 表示协变
    item: T
}

def total<T: Add>(xs: List<T>): T = { ... }    // 泛型算法

trait Functor<F: * -> *> {       // 种类写在参数上：接构造子，不接类型
    def map<A, B>(self: F<A>, f: A => B): F<B>
}

impl Functor for List {          // impl 目标不带实参 = 按构造子实现
    def map<A, B>(self: List<A>, f: A => B): List<B> = { ... }
}

type IntEither<A> = Either<Int, A>   // 部分应用写成具名别名，不写类型 lambda

@export("parse")               // 装饰器：声明前的编译期标签，可叠
@deprecated("用 Csv 代替")
def parse(line: Text): Row = { ... }

val Shapes = import("./shapes.psi")   // 导入：得到被导入文件的模块对象
```

## 宏

```
@macro                          // 宏是普通函数，编译期执行，签名 Tokens -> Tokens
def derive_show(item: Tokens): Tokens = { ... }

@derive(Show)                   // 用户宏：吃被装饰声明，吐新声明
struct Circle { radius: Float }

@route("/circles")
@cached(ttl_seconds: 60)        // 可叠：自下而上应用，先 cached 再 route
def list(): Json = { ... }
```

装饰器只有一种位置：挂在其后的声明上，可叠，自下而上应用。内建四条是 `@export`、`@test`、`@deprecated`、`@target`（[25](../ideas/25-decorators-as-tags.md)），其余名字解析为用户宏，同名冲突是编译错误。展开是工作表：每个宏跑一次，产物重新入队，深度与燃料有上限。没有表达式位的宏（[27](../ideas/27-macros.md)）。

## 表达式形式

```
42   3.14   true   "text"                 // 字面量
-1                                        // 负字面量：- 紧贴数字且在 primary 位
"psi {version} is {n} bytes"              // Text 插值：捕获作用域内的名字与表达式
[1, 2, 3]                                 // 数组字面量：定长只读
Circle { radius: 2.0 }                    // struct 字面量，类型必写
|x| x * 2                                 // 一参闭包
|acc, x| acc + x                          // 两参闭包
|| do_thing                               // 零参闭包

c.radius                                  // 零参 = 属性式读法（字段与零参方法是同一件事）
s.replace("a", "b")                       // 两参调用
xs.map { |x| x.area }                     // 尾块实参：闭包写在花括号里
xs.fold(0.0) { |acc, s| acc + s.area }    // 先普通实参，再尾块
line(shapes.first)                        // 顶层 def 的调用
Report::line(shapes.first)                // 跨模块调用
a + b * c                                 // 中缀：优先级同 Rust
a <+> b                                   // 自定义符号方法的中缀形态，按首字符定档
x?                                        // Result 传播

if (cond) { a } else { b }                // 条件是表达式
shape match {
    case Shape::Circle { radius } => 3.14159 * radius * radius
    case Shape::Rect { w, h } => w * h
    case _ => 0.0
}
while (cond) { ... }
for (x <- xs) { ... }                     // .each 的糖
for (s <- shapes if s.area > 3.0) yield s.area   // map/flatMap 的糖
json.at("user:name:0")                    // 字符串直接当 Path：at 收 Into<Path>
Path::parse("user:name:0")?               // 需要校验段时
```

## 类型形式

```
Int   Float   Bool   Text
List<T>   Option<T>   Result<T, E>   Map<K, V>      // 泛型实例
Json                                     // 边界上的动态值：普通 enum，不是 any
Path                                     // 段序列
Tokens                                   // 宏的输入输出：Group / Ident / Punct / Literal
[T]                                      // 数组
A => B                                   // 函数（闭包）类型；两参写 (A, B) => C，零参写 () => C
dyn Area                                 // 存在类型装箱
Read + Write                             // trait 合取
Shapes::Circle                           // 类型路径：模块对象上的投影
F<A>                                     // 只出现在带种类参数的 trait 里
caller.Json                              // 区域约束：分配落在调用者的区域
```

prelude 提供 `List`、`Map`、`Option`、`Result`、`Json`、`Path` 与 `Eq`、`Ord`、`Add`、`Sub`、`Mul`、`Div` 这组 trait；`Add` 实现了 `+`，其余同理。

## 记法总则

- **调用带括号与逗号**：`f(a, b)`；方法同形 `x.f(a)`。零参不写括号：`x.f`。
- **尾块**：最后一个实参是闭包时写在花括号里，`xs.map { |x| ... }`；先写其余实参，`xs.fold(0.0) { ... }`。
- **闭包是 `|参数| 体`**：`|x| x * 2`，两参 `|a, b| a + b`，零参 `|| do_thing`。`||` 与逻辑或同形，靠位置区分：表达式开头是闭包，两个操作数之间是或。
- **类型写 `A => B`，值写 `|x| …`**：类型位不写参数名（Scala 形态），值位用竖线划出闭包边界（Rust 形态）。`=>` 因此有两种读法：类型位是函数类型，`match` 里是臂。
- **`.` 与 `::` 分工**：`.` 读值成员（字段即零参方法），`::` 是路径（类型、变体、模块成员、prelude 类型）。
- **`impl` 有两只手**：`impl Circle { ... }` 写自己的方法，`impl Shape { ... }` 写在外来类型上就是扩展方法；同一个点只有一条读法，两者同名时成员方法优先。
- **`self`** 是接收者，按值传入；`_` 是通配模式。
- **泛型实参只在声明与 impl 目标里写**；推不出时用 `f::<T>(x)`。
- **种类写在参数上**：`F: * -> *`；`F` 由实参的头部构造子解出，不支持部分应用，需要时写 `type` 别名。
- **方差写在参数上**：`+T` 协变、`-T` 逆变；含 `var` 类型的参数必须不变（不写）。
- **`:` 出现在六处**：类型标注、返回类型、struct 字段、变体字段、trait 约束（`T: Ord`）、种类（`F: * -> *`）。
- **中缀优先级取 Rust 的表**：`* / %` > `+ -` > `<< >>` > `&` > `^` > `|` > `== != < <= > >=` > `&&` > `||`；自定义符号按首字符挂到对应档，比较符不可连写。
- **语句按换行分隔**；同一行写多条时用 `;`。块尾表达式是返回值。
- **`?` 挂在表达式尾部**，只在返回 `Result` / `Option` 的 def 里合法。
- **没有匿名记录**：结构化数据用带类型名的 struct 字面量（类型必填，见 [29](../ideas/29-data-first-levers.md)）。
- **没有 `&`、`*`、生命周期**：值语义；跨区域赋值等于拷贝。
- **链即管道**：`xs.filter(p).map(f).len`。

## 续行规则

语句在换行处结束。以下情况判为这一行还没写完，继续到下一行：行尾是未闭合的 `(`、`[`、`{`；行尾是中缀符号、`=` 或 `=>`；下一行以 `.`、`?` 或 `{` 开头。同一行写多条语句时用 `;`。

## 文法

```
program    = item*
item       = attribute* decl
attribute  = '@' name ( '(' tokens? ')' )?
decl       = binding | def | struct | enum | trait | impl | object | alias
binding    = ( 'val' | 'var' ) name ( ':' type )? '=' expr
def        = 'def' name generics? '(' params? ')' ( ':' type )? ( block | ';' )
params     = param ( ',' param )*
param      = 'self' | name ':' type
generics   = '<' gparam ( ',' gparam )* '>'
gparam     = variance? name ( ':' bound )?
variance   = '+' | '-'
bound      = type
struct     = 'struct' name generics? body
enum       = 'enum' name generics? '{' variant* '}'
variant    = name ( '{' field* '}' )?
trait      = 'trait' name generics? ( ':' bound )? body
impl       = 'impl' type ( 'for' type )? body
object     = 'object' name body
body       = '{' ( field | def )* '}'
field      = name ':' type
alias      = 'type' name generics? '=' type

type       = fn_type | base_type
fn_type    = ( '(' type,* ')' | base_type ) '=>' type
base_type  = path ( '<' type,* '>' )? | '[' type ']' | base_type '+' base_type | 'dyn' name
path       = name ( '::' name )*

expr       = assign | infix
assign     = name '=' expr
infix      = chain ( symbol chain )*
chain      = primary ( '.' name ( '(' args? ')' )? | '(' args? ')' | '{' expr* '}' | '?' )*
primary    = literal | name | path | '(' expr ')' | closure | record
           | '[' args? ']' | block | if | match | for
closure    = '|' params? '|' ( expr | block )
record     = path '{' ( name ':' expr ','? )* '}'
if         = 'if' '(' expr ')' expr ( 'else' expr )?
match      = expr 'match' '{' case+ '}'
case       = 'case' pattern '=>' expr
for        = 'for' '(' gen ( ';' gen )* ')' ( 'yield' expr | block )
```

## 一段完整程序

```
enum Shape {
    Circle { radius: Float }
    Rect { w: Float, h: Float }
}

trait Area {
    def area(self): Float
}

impl Area for Shape {
    def area(self): Float = {
        self match {
            case Shape::Circle { radius } => 3.14159 * radius * radius
            case Shape::Rect { w, h } => w * h
        }
    }
}

impl Area for List<Shape> {
    def area(self): Float = {
        self.fold(0.0) { |acc, s| acc + s.area }
    }
}

def line(s: Shape): Text = { "area = {s.area}" }

def main(args: List<Text>): Int = {
    val shapes = [
        Shape::Circle { radius: 1.0 }
        Shape::Rect { w: 2.0, h: 3.0 }
    ]
    var biggest = 0.0
    for (s <- shapes) {
        if (s.area > biggest) { biggest = s.area }
    }
    shapes
        .filter { |s| s.area > 2.0 }
        .map { |s| line(s) }
        .each { |t| Io::stdout.println(t) }
    for (s <- shapes) yield s.area
    shapes.area.println
    biggest > 0.0
    0
}
```