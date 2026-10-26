# 语法总表

纯参考件：这里只有形状，不承载判断。每条形状属于哪份想法、代价在哪，见 `docs/ideas/`。

## 声明形式

```
// 注释到行尾；/// 是文档注释

let name: Text = "psi";          // 不可变绑定
let mut count: Int = 0;          // 可重绑槽
count = count + 1;               // 赋值语句，只能赋 mut 槽

fn area(c: Circle) -> Float { 3.14159 * c.radius * c.radius }   // 顶层 fn 是模块对象的方法
fn clamp(x: Float, lo: Float, hi: Float) -> Float { ... }        // 三参：括号逗号

struct Circle {                  // 数据形状：字段 + 固有方法
    radius: Float,
}

impl Circle {
    fn area(self) -> Float { 3.14159 * self.radius * self.radius }
    fn scale(self, k: Float) -> Circle { Circle { radius: self.radius * k } }
    fn clamp(self, lo: Float, hi: Float) -> Circle { ... }
}

enum Shape {                     // 和类型：变体带命名字段
    Circle { radius: Float },
    Rect { w: Float, h: Float },
    Empty,
}

trait Area {                     // 标签：签名 + 可选默认实现
    fn area(self) -> Float;
    fn twice(self) -> Float { self.area * 2.0 }
}

impl Area for Shape {            // 唯一的附着动作
    fn area(self) -> Float {
        match self {
            Shape::Circle { radius } => 3.14159 * radius * radius,
            Shape::Rect { w, h } => w * h,
            Shape::Empty => 0.0,
        }
    }
}

object Geo {                     // 单例：名字即那个实例
    let pi: Float = 3.14159;
    fn unit() -> Circle { Circle { radius: 1.0 } }
}

struct Box<T> {                  // 类型参数写在名字后
    item: T,
}

fn total<T: Add>(xs: List<T>) -> T { ... }    // 泛型算法

trait Functor<F> {               // 种类参数：接构造子，不接类型
    fn map<A, B>(self: F<A>, f: Fn(A) -> B) -> F<B>;
}

impl Functor for List {          // impl 目标不带实参 = 按构造子实现
    fn map<A, B>(self: List<A>, f: Fn(A) -> B) -> List<B> { ... }
}

#[export("parse")]               // 属性：声明前的编译期标签，可叠
#[deprecated("用 Csv 代替")]
fn parse(line: Text) -> Row { ... }

let Shapes = import("./shapes.psi");   // 导入：得到被导入文件的模块对象
```

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
xs.map(|x| x.area)                        // 闭包是普通实参
xs.fold(0.0, |acc, s| acc + s.area)       // 多实参
line(shapes.first)                        // 顶层 fn 的调用
Report::line(shapes.first)                // 跨模块调用
a + b * c                                 // 中缀：优先级同 Rust
a <+> b                                   // 自定义符号方法的中缀形态，按首字符定档
x?                                        // Result 传播

if cond { a } else { b }                  // 条件不加括号，是表达式
match shape {
    Shape::Circle { radius } => 3.14159 * radius * radius,
    Shape::Rect { w, h } => w * h,
    _ => 0.0,
}
while cond { ... }
for x in xs { ... }                       // .each 的糖
json.at("user:name:0")                    // 字符串直接当 Path：at 收 Into<Path>
Path::parse("user:name:0")?               // 需要校验段时
```

## 类型形式

```
Int   Float   Bool   Text
List<T>   Option<T>   Result<T, E>   Map<K, V>      // 泛型实例
Json                                     // 边界上的动态值：普通 enum，不是 any
Path                                     // 段序列
[T]                                      // 数组
Fn(A) -> B                               // 函数（闭包）类型；两参写 Fn(A, B) -> C
dyn Area                                 // 存在类型装箱
Read + Write                             // trait 合取
Shapes::Circle                           // 类型路径：模块对象上的投影
F<A>                                     // 只出现在带种类参数的 trait 里
caller.Json                              // 区域约束：分配落在调用者的区域
```

prelude 提供 `List`、`Map`、`Option`、`Result`、`Json`、`Path` 与 `Eq`、`Ord`、`Add`、`Sub`、`Mul`、`Div` 这组 trait；`Add` 实现了 `+`，其余同理。

## 记法总则

- **调用带括号与逗号**：`f(a, b)`；方法同形 `x.f(a)`。零参不写括号：`x.f`。
- **`.` 与 `::` 分工**：`.` 读值成员（字段即零参方法），`::` 是路径（类型、变体、模块成员、prelude 类型）。
- **`self`** 是接收者，按值传入；`_` 是通配模式。
- **泛型实参只在声明与 impl 目标里写**；推不出时用 `f::<T>(x)`。
- **`:` 出现在四处**：类型标注、struct 字段、变体字段、trait 约束（`T: Ord`）。
- **中缀优先级取 Rust 的表**：`* / %` > `+ -` > `<< >>` > `&` > `^` > `|` > `== != < <= > >=` > `&&` > `||`；自定义符号按首字符挂到对应档，比较符不可连写。
- **语句以 `;` 结束**；块尾表达式是返回值。换行不承载语义。
- **`?` 挂在表达式尾部**，只在返回 `Result` / `Option` 的 fn 里合法。
- **没有匿名记录**：结构化数据用带类型名的 struct 字面量（类型必填，见 [29](../ideas/29-data-first-levers.md)）。
- **没有 `&`、`*`、生命周期**：值语义；跨区域赋值等于拷贝。
- **链即管道**：`xs.filter(p).map(f).len`。

## 续行规则

语句在 `;` 或 `}` 处结束，行尾不是它们就继续；括号与方括号内部任意换行。

## 文法

```
program    = item*
item       = attribute* decl
attribute  = '#[' name ( '(' literal ')' )? ']'
decl       = binding | fn | struct | enum | trait | impl | object
binding    = 'let' 'mut'? name ( ':' type )? '=' expr ';'
fn         = 'fn' name generics? '(' params? ')' ( '->' type )? ( block | ';' )
params     = param ( ',' param )*
param      = 'self' | name ':' type
generics   = '<' name ( ':' bounds )? ( ',' name ( ':' bounds )? )* '>'
struct     = 'struct' name generics? '{' ( name ':' type ',' )* '}'
enum       = 'enum' name generics? '{' ( variant ',' )* '}'
variant    = name ( '{' ( name ':' type ',' )* '}' )?
trait      = 'trait' name generics? ( ':' bounds )? '{' fn* '}'
impl       = 'impl' name ( 'for' type )? '{' fn* '}'
object     = 'object' name '{' ( binding | fn )* '}'

type       = path ( '<' type,* '>' )? | '[' type ']' | type '+' type | 'dyn' name
           | 'Fn' '(' type,* ')' '->' type
path       = name ( '::' name )*

expr       = assign | infix
assign     = name '=' expr
infix      = chain ( symbol chain )*
chain      = primary ( '.' name ( '(' args? ')' )? | '(' args? ')' | '?' )*
primary    = literal | name | path | '(' expr ')' | closure | record
           | '[' args? ']' | block | if | match | while | for
closure    = '|' params? '|' ( expr | block )
record     = path '{' ( name ':' expr ',' )* '}'
```

## 一段完整程序

```
enum Shape {
    Circle { radius: Float },
    Rect { w: Float, h: Float },
}

trait Area {
    fn area(self) -> Float;
}

impl Area for Shape {
    fn area(self) -> Float {
        match self {
            Shape::Circle { radius } => 3.14159 * radius * radius,
            Shape::Rect { w, h } => w * h,
        }
    }
}

impl Area for List<Shape> {
    fn area(self) -> Float {
        self.fold(0.0, | acc, s | acc + s.area)
    }
}

fn line(s: Shape) -> Text { "area = {s.area}" }

fn main(args: List<Text>) -> Int {
    let shapes = [
        Shape::Circle { radius: 1.0 },
        Shape::Rect { w: 2.0, h: 3.0 },
    ];
    let mut biggest = 0.0;
    for s in shapes {
        if s.area > biggest { biggest = s.area; }
    }
    shapes
        .filter(|s| s.area > 2.0)
        .map(|s| line(s))
        .each(|t| Io::stdout.println(t));
    shapes.area.println;
    biggest > 0.0;
    0
}
```