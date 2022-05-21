---
layout: post
title: "Rust - Pattern Match"
date: 2021-10-13 08:00:00 +0800
comments: true
categories: rust
---

- 模式是 Rust 的一种特殊语法，用于匹配复杂和简单类型的结构
- 将模式与匹配表达式和其他构造结合使用，可以更好地控制程序的控制流
- 模式由以下元素 (的一些组合) 组成:
    - 字面值
    - 解构的数组、enum、struct 和 tuple
    - 变量
    - 通配符
    - 占位符
- 想要使用模式，需要将其与某个值比较
    - 如果可以模式匹配，就可以在代码中使用这个值的相应部分

## 可用用到模式匹配的地方
### match 的分支
- 要求分支需要包含所有的可能性
    - `_`: 可以匹配任何东西，不会绑定到变量，通常用于 match 的最后一个分支，或用于忽略某些值

```rust
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### if let
- 简单的控制流表达式，处理只关心一种匹配而忽略其他匹配的情况，放弃了穷举的可能性，可以理解成 match 的简化版
- 主要是作为一种简短的方式来等价的代替只有一个匹配项的 match 
- `if let` 可选的可以拥有 `else` `else if` `else if let`
- `if let` 不会检查穷尽性

```rust
let v = Some(0u8);

match v {
    Some(3) => println!("three"),
    _ => println!("others"),
}

// 可使用 if let 改写
if let Some(3) = v {
    println!("three");
} else {
    println!("others");
}
```

### while let 条件循环
- 只有模式继续满足匹配的条件，那它运行 while 循环一直运行

```rust
let mut stack = vec![1, 2, 3];
while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

### for 循环
```rust
let arr = vec!['a', 'b', 'c'];
for (i, v) in arr.iter().enumerate() {
    println!("{}: {}", i, v);
}
```

### let 语句
```rust
let a = 5; // 这个其实也是一个模式
let (x, y, z) = (1, 2, 3);
```

### 函数参数
```rust
fn foo(x: i32) {} // 参数 x 也是一个模式
fn foo2(&(x, y): &(i32, i32)) {}

let point = (3, 5);
foo2(&point);
```

## 模式的可辩驳性
- 模式有两种形式: 可辩驳的(可失败的), 无可辩驳的(不可失败的)
    - 能匹配任何可能传递的值的模式: 无可辩驳的
        - 例如: `let x = 5`
    - 对于某些可能的值，无法继续匹配的模式: 可辩驳的
        - 例如: `if let Some(x) = val`
- 函数参数、let 语句、for 循环只接收**无可辩驳**的模式
- `if let` 和 `while let` 接收**可辩驳**和**无可辩驳**的模式
- match 语句，除了最后一个分支是**无可辩驳**的，其他分支为**可辩驳**

## 匹配语法
### 匹配字面值
```rust
let x = 1;

match x = {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

### 匹配命名变量
- 命名的变量是可匹配任何值的无可辩驳模式

```rust
let x = Some(5);
let y = 10;

match x = {
    Some(50) => println!("Got 50"),
    Some(y) => println!("Matched, y = {:?}", y), // 作用域只在子句中
    _ => println!("Default case, x = {:?}", x),
}

println!("at the end: x = {:?}, y = {:?}", x, y);
// "Matched, y = 5"
// "at the end: x = Some(5), y = 10"
```

### 多重模式
- match 可使用 `|`(或运算符), 可以匹配多种模式

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

### 匹配范围
```rust
let x = 5;

match x {
    1..=5 => println!("one through five"), // 1,2,3,4,5
    _ => println!("something else"),
}
```

### 使用模式来解构分解值
```rust
// 解构 struct
struct Point { x: i32, y: i32, }

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);

    match p {
        // 子分支会创建 x y 变量
        // 匹配 x = any, y = 0 的 Point
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        // 匹配 x = 0, y = any 的 Point
        Point { x: 0, y } => println!("On the y axis at {}", y), // 匹配到该行
        // 匹配 x = any, y = any 的 Point
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```
```rust
// 解构 enum
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {...}
        // 会创建 x y 变量
        Message::Move { x, y } => {...}
        Message::Write(text) => {...},
        Message::ChangeColor(r, g, b) => {...}
    }
}
```
```rust
// 解构嵌套的类型
enum Color {
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {...}
        Message::ChangeColor(Color::Hsv(h, s, v)) => {...}
        _ => ()
    }
}
```
```rust
// 解构 tuple
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
```

#### 在模式中忽略值
- `_`: 可用来忽略某个值
- 使用以 `_` 开头的名称: 标记该值不会被用到
- `..`: 忽略值的剩余部分

```rust
// 使用 `_` 来作为匹配但不绑定任何值的通配符
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

// --------
let mut setting_value = Some(5);
let new_setting_value = Some(10);
match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}
println!("setting is {:?}", setting_value);

// --------
let numbers = (2, 4, 8, 16, 32);
match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
}
```
```rust
// 使用以 `_` 开头的名称: 忽略被使用的变量
let _x = 5; // _x 没被使用也不会有警告

// -------
let s = Some(String::from("Hello!"));
// s 多所有权被移入了 if let 子句了, _s 会发生绑定操作获取所有权
// 如果将 Some(_s) 修改为 Some(_)，_ 不会发生绑定操作，所以 s 的所有权也不会移动了
if let Some(_s) = s {
    println!("found a string");
}
```
```rust
// 用 .. 忽略剩余值
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}

// ----------
let numbers = (2, 4, 8, 16, 32);

match numbers {
    // (.., second, ..) 这种模糊的匹配会失败
    (first, .., last) => {
        println!("Some numbers: {}, {}", first, last);
    },
}
```

#### match guard
```rust
// 指定于 match 分支模式之后的额外 if 条件，它也必须被满足才能选择此分支
let x = Some(5);
let y = 10;

match x {
    Some(50) => println!("Got 50"),
    // 匹配 Some(n), 然后绑定 n, 如果 n == y, 匹配成功
    // 该例中, 匹配 Some(n) 成功，但是由于 n != y，所以进入 _ 分支
    Some(n) if n == y => println!("Matched, n = {}", n),
    _ => println!("Default case, x = {:?}", x), // 最终匹配到这个分支
}
```
```rust
let x = 4;
let y = false;

match x {
    // 匹配 4, 5, 6; 并且 y == true
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"), // 最终匹配到这个分支
}
```

#### @ 绑定
```rust
// @ 符号让我们可以创建一个变量，该变量可以在测试某个值是否与模式匹配的同时保存该值
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    // 要求 id 的值在 3~7 范围内，同时使用 @ 将匹配到的值存储在 id_variable 变量中
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable) // 
    },
    // 这里由于 id 匹配的是一个范围，其可能是 10 11 12, 没有具体的值，所以即使匹配到了也无法绑定
    // 如果匹配到该子句了，想在子句内使用具体匹配到的值，也得 { id: id_variable @ 10..=12 }
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    // 该子句指定了一个没有范围的比例，所以如果匹配到这行，id 可以绑定具体的值
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
// Found an id in range: 5
```