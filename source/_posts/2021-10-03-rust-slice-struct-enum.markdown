---
layout: post
title: "Rust - Slice, Struct, Enum"
date: 2021-10-03 08:00:00 +0800
comments: true
categories: rust
---

<!-- more -->

* [Slice](#Slice)
* [Struct](#Struct)
* [Enum](#Enum)


## <h2 id="Slice">Slice</h2>
- 切片 Slice 是不持有所有权的数据类型

### 字符串切片
- 字符串切片: 指向字符串中一部分内容的引用
    - 形式: `[开始索引..结束索引]`
        - 开始索引: 切片起始位置的索引值
        - 结束索引: 切片终止位置的下一个索引值
    - 字符串切片的范围索引必须发生在有效的 UTF-8 字符边界内
    - 如果尝试从一个多字节的字符中创建字符串切片，程序会报错并退出

```rust
let s = String::from("hello world");

let hello = &s[0..5];        // 也可以写成 &s[..5]
let world = &s[6..11];       // 也可以写成 &s[6..]
let hello_world = &s[0..11]; // 也可以写成 &s[0..s.len()] &s[..]
```
- 字符串字面值 `&str` 是切片
    - 其是被直接存储在二进制程序中的
    - `&str` 是不可变引用，所以字符串字面值也是不可变的

```rust
let s = "hello world"; // &str 字符串切片类型
// s 是指向二进制程序特定位置的切片
```

### 其他类型的切片
```rust
let a = [1, 2, 3, 4, 5];
let slice = &a[1..3]; // 类型为 &[i32]
```


## <h2 id="Struct">Struct</h2>
```rust
// struct 又自身所有数据的所有权，只要 struct 实例是有效的，那么里面的字段数据也是有效的
// struct 里面的字段也可以放引用，但是需要使用生命周期 (生命周期可以保证只要 struct 实例有效，那么里面的引用也是有效的)
struct User {
    username: String, // 持有所有权
    email: String,    // 持有所有权
    age: i32,         // 标量类型
}

// 一旦 struct 的实例是可变的，那么实例中所有的字段都是可变的
let mut user = User {
    email: String::from("a@qq.com"),
    username: String::from("wt"),
    age: 16,
};
println!("{}", user.email);
user.email = String::from("b@qq.com");

// 字段名与变量名相同使，可初始化简写
fn build_user(email: String, username: String) -> User {
    User { email, username, age: 16 }
}

// 基于某个 struct 实例，创建新的 struct
let user2 = User { email: String::from("c@qq.com"), ..user1 };
```
```rust
struct Point3d {
    x: i32,
    y: i32,
    z: i32,
}

fn default() -> Point3d {
    Point3d { x: 0, y: 0, z: 0 }
}

let origin = Point3d { x: 5, ..default() };
let point = Point3d { z: 1, x: 2, ..origin };
```

### Tuple Struct
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
// Color, Point 是不同的类型
```
```rust
// 常用于包装一些基本数据类型，去扩展它的功能 (NewType 模式)
struct Score(u32);

impl Score {
    fn pass(&self) -> bool {
        self.0 >= 60
    }
}

let s = Score(59);
assert_eq!(s.pass(), false);
```

### Unit-Like Struct
- 适用于需要在某个类型上实现某个 trait，但是在里面又没有想要存储的数据
- 可以看作是占位类型，其实例就是其自身。不管创建多少实例，编译器都会优化成同一个，其也不会占用内存空间，是一个零大小类型

```rust
struct Unit;

let unit1 = Unit;
let unit2 = Unit;
```

### Recursive Enum
```rust
struct Recursive {
    Data: i32,
    rec: Box<Recursive>, // 需要用 Box 包装
}
```

### Print Struct
```rust
#[derive(Debug)]
struct User {
    a: u32,
    b: u32,
}
let u = User { a: 1, b: 2 };

println!("{:?}", u);
println!("{:#?}", u); // 比 {:?} 更漂亮些
```

### Struct Method
- C/C++ 调用方法: `obj->something()` 和 `(*obj).something()`
- Rust 没有 `->` 运算符, Rust 会自动引用或者解引用
- 在调用方法时，Rust 根据情况会自动添加 `&`、`&mut`、或 `*`，以便 Object 可以匹配方法的签名
    - `p1.distance(&p2)` 效果等价于 `(&p1).distance(&p2)`
- 每个 struct 可以有多个 impl 块

```rust
struct User {
    username: String,
    email: String,
}

impl User {
    // 实例方法
    fn say(&self, st: &str) {
        println!("{} say: {}", self.username, st);
    }

    // 静态方法 (关联函数)
    fn dosomething() {}
}

fn main() {
    let u = User { username: "wt".to_string(), email: "a@a.com".to_string() };
    u.say("hello"); // wt say: hello

    // 调用关联函数
    User::dosomething();
}
```

## <h2 id="Enum">Enum</h2>
```rust
enum IpAddrKind {
    V4,
    V6,
}

fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;

    route(four);
}

fn route(ip_kind: IpAddrKind) {}
```
```rust
// 数据附加到枚举到变体中
// - 优点:
//   - 不需要额外使用 struct 存储数据
//   - 每个变体可以拥有不同的类型以及关联的数据
enum IpAddrKind {
    V4(u8, u8, u8, u8), // 可嵌入任意类型的数据，String、struct ...
    V6(String),
}

fn main() {
    let home = IpAddrKind::V4(127, 0, 0, 1);
    let loopback = IpAddrKind::V6(String::from("::1"));
}
```
```rust
enum Message {
    Quite,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

// enum method
impl Message {
    fn call(&self) {}
}

fn main() {
    let q = Message::Quit;
    let m = Message::Move { x: 12, y: 24 };
    let w = Message::Write(String::from("Hello"));
    let c = Message::ChangeColor(0, 255, 255);

    c.call();
}
```

### Option Enum
- 定义于标准库中，在 Prelude(预导入模块)中
- 描述了: 某个值可能存在(某种类型)或不存在的情况
- Rust 没有 Null，有类似 Null 概念的枚举 `Option<T>`
    - 其包含在 Prelude 中可以直接使用
    - `Option<T>`、`Some<T>`、`None`

```rust
// 标准库中的定义
enum Option<T> {
    Some(T),
    None,
}
```
```rust
let some_num = Some(5);
let some_str = Some("a");

let absent_num: Option<i32> = None;
```

### Union
```rust
union U {
    u: u32,
    v: u64,
}
```