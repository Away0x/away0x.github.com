---
layout: post
title: "Rust - Generic, Trait"
date: 2021-10-07 08:00:00 +0800
comments: true
categories: rust
---

<!-- more -->

* [Generic](#Generic)
* [Trait](#Trait)


## <h2 id="Generic">Generic</h2>
- 泛型: 是具体类型或其他属性的抽象代替，能够提高代码的复用能力
- Rust 中使用泛型的代码和使用具体类型的代码运行速度是一样的
    - Rust 在编译过程中会做[单态化 (monomorphization)](https://en.wikipedia.org/wiki/Monomorphization) 处理 (编译时将泛型替换为具体类型的过程), 泛型函数里每个用到的类型会编译出一份代码

```rust
// 函数中使用
fn largest<T>(list: &[T]) -> T {...}
```
```rust
// struct 中使用
struct Point<T> {
    x: T,
    y: T,
}
struct Point2<T, U> {
    x: T,
    y: U,
}

// impl<T> 表示在类型 T 上实现方法
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// 针对具体的类型创建方法，其他 Point<T> 类型没有 x1 方法
impl Point<i32> {
    fn x1(&self) -> &i32 {
        &self.x
    }
}

// 方法的参数泛型可以与 struct 不同
impl<T, U> Point2<T, U> {
    fn mixup(V, W)(self, other: Point2<V, W>) -> Point2<T, W> {
        Point2 {
            x: self.x,
            y: other.y,
        }
    }
}

let p1 = Point2 { x: 5, y: 4 };
let p2 = Point2 { x: "hello", y: 'c' };
let p3 = p1.mixup(p2);
println!("{}", p3); // Point2 { x: 5, y: 'c' }
```
```rust
// enum 中使用
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```


## <h2 id="Trait">Trait</h2>
- Trait: 抽象的定义共享行为
    - 告诉编译器，某种类型具有哪些并且可以与其他类型共享的功能
- Trait bounds(约束): 泛型类型参数指定为特定行为的类型
- 可以在某个类型上实现某个 Trait 的前提条件是:
    - 这个类型或这个 trait 是在本地 crate 里定义的
    - 无法为外部类型来实现外部的 trait (孤儿原则: 确保其他人的代码不能破坏您的代码，反之亦然)
- Rust 的面相对象特性
    - 封装: 私有字段和 pub 关键字
    - 继承: Rust 中使用 trait 方法来进行代码共享
        - trait 默认方法，可使实现改 trait 的对象复用逻辑
        - 而实现 trait 的对象也可以重写这些默认的方法
    - 多态: 使用泛型和 trait 约束 (限定参数化多态 bounded parametric)
    

```rust
// 定义 Trait
// - 只有方法签名，没有方法实现
pub trait Summary {
    fn summarize(&self) -> String;
}

// 为类型实现 Trait
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```
```rust
// trait 作为函数参数
// 1. 使用 imple trait 语法: 适用于简单情况
//    参数必须为实现了 Summary trait 的类型
pub fn notify1(item1: impl Summary, item2: impl Summary) {
    println!("Breaking news! {} {}", item1.summarize(), item2.summarize());
}
// 2. 使用 trait bounds 语法
//    泛型 T 需要实现多个 trait 的话可以使用 +，例如 <T: Summary + Display>
pub fn notify2<T: Summary>(item1: T, item2: T) {
    println!("Breaking news! {} {}", item1.summarize(), item2.summarize())
}
// 3. trait bounds 搭配 where (适合约束复制的情况)
// 例如 pub fn notify3<T: Summary + Display, U: Clone + Debug>(a: T, b: U) -> String
pub fn notify3<T, U>(a: T, b: U) -> String
where
    T: Summary + Display,
    U: Clone + Debug,
{
    format!("{}", a.summarize())
}
```
```rust
// trait 作为函数返回值
// - impl trait 只能返回确定的同一种类型，返回可能不同类型的代码会报错
pub getSummary() -> impl Summary {
    NewsArticle {...}
}
// 返回的类型需要唯一，返回了不同的类型，虽然都实现了 Summary trait，但是 rust 不允许
pub getSummary2(flag: bool) -> impl Summary {
    if flag {
        NewsArticle {...}
    } else {
        Tweet {...}
    }
}
```

### 使用 trait bound 有条件的实现方法
```rust
// 在使用泛型类型参数的 impl 块上使用 Trait bound, 我们可以有条件的为实现了特定 Trait 的类型来实现方法
struct Pair<T> {
    x: T,
    y: T,
}

// 所有的 Pair 类型都有 new 方法
impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

// 只有 T 实现了约束的 Pair 类型，才有 cmp_display 方法
impl <T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```
```rust
// 可以为实现了其他 trait 的任意类型有条件的实现某个 trait
// 为满足 Trait bound 的所有类型上实现 trait 叫做覆盖实现 (blanket implementations)

// 标准库的代码, 可以为任意实现了 Display trait 的类型实现 ToString trait
// 所以所以有 Display 的类型，都有 to_string 方法
impl<T: Display> ToString for T {
    fn to_string(&self) -> String {...}
}
```

### 默认实现
- 默认实现的方法，可以调用 trait 中的其他方法，即使这些方法没有默认实现
- 无法在方法的重写实现里面调用默认的实现

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("Read more ...")
    }
}

// 不自己实现，使用默认实现的方法
struct A;
impl Summary for A {}

fn main() {
    let a = A {};
    println!("{}", a.summarize());
}
```

### 使用 trait 对象存储不同类型的值
- 需求: 创建一个 GUI 工具:
    - 它会遍历某个元素的列表，依次调用元素的 draw 方法进行绘制
    - 例如: Button, TextField 等元素

```rust
// 为共有的行为定义 trait
pub trait Draw {
    fn draw(&self);
}

// 使用 Trait 对象实现多态
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}

// 使用 trait bound
// pub struct Screen<T: Draw> {
//     trait bound 泛型类型参数一次只能替代一个具体类型，而 trait 对象运行在运行时提到多种具体类
//     pub components: Vec<T>,
// }
// impl<T> Screen<T>
// where
//     T: Draw
// {
//     pub fn run(&self) {
//         for component in self.components.iter() {
//             component.draw();
//         }
//     }
// }

pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}
impl Draw for Button {
    fn draw(&self) {
        // 绘制一个按钮
    }
}

pub struct SelectBox {
    pub width: u32,
    pub height: u32,
    pub options: Vec<String>,
}
impl Draw for SelectBox {
    fn draw(&self) { /* 绘制一个选择框 */ }
}

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![String::from("Yes"), String::from("Maybe"), String::from("No")],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("Ok"),
            }),
        ],
    };
    screen.run();
}
```

#### Trait 对象执行的是动态派发
- 将 trait 约束作用于泛型时，Rust 编译器会执行单态化
    - 编译器会为我们用来替换泛型类型参数的每一个具体类型生成对应函数和方法的非泛型实现
- 通过单态化生成的代码会执行**静态派发(static dispatch)**，在编译过程中确定调用的具体方法
- **动态派发(dynamic dispatch)** `dyn 关键字`
    - 无法在编译过程中确定你调用的究竟是哪一种方法
    - 编译器会产生额外的代码以便在运行时找出希望调用的方法
- 使用 trait 对象, 会执行动态派发
    - 会产生运行时开销
    - 会阻止编译器内联方法代码，使得部分优化操作无法进行

#### Trait 对象必须保证对象安全
- 只能把满足对象安全 (object-safe) 的 trait 转化为 trait 对象
- Rust 采用一系列规则来判定某个对象是否安全，只需要记住两条:
    1. 方法的返回类型不是 Self
    2. 方法中不包含任何泛型类型参数

```rust
pub trait Clone {
    fn clone(&self) -> Self; // 不满足对象安全
}

pub struct Screen {
    // 由于 Clone 不满足对象安全规则，所以不可使用 trait 对象，报错
    pub components: Vec<Box<dyn Clone>>,
}
```

### 在 trait 定义中使用关联类型来指定占位类型
- 关联类型 (associated type) 是 Trait 中的类型占位符, 它可以用于 trait 的方法签名中
    - 可以定义出包含某些类型的 trait, 而在实现前无需知道这些类型是什么

```rust
pub trait Iterator {
    type Item; // 关联类型 (类型占位符)
    // 迭代过程中, 使用 Item 替代具体迭代的值
    fn next(&mut self) -> Option<Self::Item>;
}
```

#### 关联类型和泛型的区别
| 泛型 | 关联类型 |
| - | - |
| 每次实现 trait 时需要标注类型 | 无需标注类型 |
| 可以为一个类型多次实现某个 trait (不同的泛型参数) | 无法为单个类型多次实现某个 trait |

```rust
// 关联类型
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter {}

// Iterator 这个 trait 只能实现一次
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        None
    }
}
```
```rust
// 泛型版本
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}

struct Counter {}

impl Iterator<String> for Counter {
    fn next(&mut self) -> Option<String> {
        None
    }
}

// 可以对不同的泛型多次实现 Iterator trait
// impl Iterator<u32> for Counter
```

### 默认泛型参数和运算符重载
- 可以在使用泛型参数时为泛型指定一个默认的具体类型 `<PlaceholderType=ConcreteType>`
    - 这种技术常用于运算符重载 (operator overloading)
    - Rust 不允许创建自己的运算符及重载任意的运算符, 但可以通过实现 `std::ops` 中列出的那些 trait 来重载一部分相应的运算符

```rust
// Add Trait 里面使用了默认泛型参数, trait Add<RHS=Self>, 如果没有指定参数的话, 就使用 Self
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point { x: self.x + other.x, y: self.y + other.y }
    }
}

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 }, Point { x: 3, y: 3 });
}
```
```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

// 显式指明泛型参数, 不使用 Add 的默认泛型参数
impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

### 完全限定语法
- 完全限定语法 `<Type as Trait>::function(receiver_if_method, next_arg, ...)`
    - 可以在任何调用函数或方法的地方使用
    - 允许忽略那些从其他上下文能推导出来的部分
    - 当 Rust 无法区分你期望调用哪个具体实现的时候, 才需使用这种语法

```rust
// 实例同名方法的调用
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) { println!("Pilot"); }
}

impl Wizard for Human {
    fn fly(&self) { println!("Wizard"); }
}

impl Human {
    fn fly(&self) { println!("Human"); }
}

fn main() {
    let p = Human;

    p.fly();         // Human
    Pilot::fly(&p);  // Pilot
    Wizard::fly(&p); // Wizard

}
```
```rust
// 静态同名方法的调用
trait Animal {
    fn name() -> String;
}

struct Dog;

impl Dog {
    fn name() -> String { String::from("Dog") }
}

impl Animal for Dog {
    fn name() -> String { String::from("Animal") }
}

fn main() {
    Dog::name(); // Dog
    <Dog as Animal>::name(); // Animal
}
```

### 使用 supertrait 来要求 trait 附带其他 trait 的功能
- 需要在一个 trait 中使用其他 trait 的功能
    - 需要被依赖的 trait 使用其他 trait 的功能
    - 那个被间接依赖的 trait 就是当前 trait 的 supertrait

```rust
use std::fmt;

// OutlinePrint 需要使用到 Display trait 的功能
trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
    }
}

struct A;

// 报错, 要求 A 必须实现了 Display trait 才行
impl OutlinePrint for A {}
```

### newtype 模式
- newtype 模式
    1. 可以用来静态的保证各种值之间不会混淆并表明值的单位
    2. 可以为类型的某些细节提供抽象能力
    3. 通过轻量级的封装来隐藏内部的实现细节

#### 外部类型上实现外部 trait
- 孤儿规则: 只有当 trait 或类型定义在本地包时, 才能为该类型实现这个 trait
- 可以使用 newtype 模式来绕过这一规则
    - 利用 tuple struct 创建一个本地的新类型

```rust
use std::fmt;

// 想要为 Vec 实现 Display trait, 但是 Vec Display 都定义在外部包, 所以不可实现
// 这里通过 tuple struct 包装 Vec 为本地类型, 这样便绕过了孤儿规则, 从而可以实现 Display trait 累
struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```
