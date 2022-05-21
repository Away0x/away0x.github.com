---
layout: post
title: "Rust - Lifetime"
date: 2021-10-08 08:00:00 +0800
comments: true
categories: rust
---

- Rust 的每个引用都有自己的生命周期
- 生命周期: 引用保持有效的作用域
- 大多数情况: 生命周期是隐式的、可被推断的
- 当引用的生命周期可能以不同的方式互相关联时: 需要手动标注生命周期

## 生命周期的标注语法
- 生命周期的标注并不会改变引用的生命周期长度
- 当指定了泛型的生命周期参数，函数可以接收带有任何生命周期的引用
- 生命周期的标注: 描述了多个引用的生命周期间的关系，但不影响生命周期
- 语法: 以 `'` 开头, 例如: `'a`
- 标注位置
    - 在引用的 `&` 符号后
    - 使用空格讲标注和引用类型分开
- 单个生命周期的标注本身并没有意义

```rust
&i32        // 一个引用
&'a i32     // 带有显示生命周期的引用
&'a mut i32 // 带有显示生命周期的可变引用
```
- 函数签名中的生命周期标注
    - 泛型生命周期参数声明在: 函数名和参数列表之间的 `<>` 里

```rust
fn test<'a>(x: &'a str, y: &'a str) -> &'a str {...}
```

## 生命周期 - 避免悬垂引用 (dangling reference)
- 生命周期的主要目标: 避免悬垂引用
- Rust 编译器的借用检查器: 比较作用域来判断所有的借用是否合法

```rust
fn main() {
    {
        let r;                                      // ----------------+-- 'a
        {                                           //                 /
            let x = 5;                              // -+-- 'b         /
            r = &x;                                 //  /              /
        }                                           // -+--            /
        // 这里使用 r 时，r 所指向的 x 已经由于脱离作用域, //                /
        // 被销毁了，所以会报错                         //                /
        // 这里 Rust 编译器会使用借用检查器发现这种错误.   //                /
        println!("r: {}", r);                       //                 /
    }                                               // ----------------+--
}
// 上例 r 的生命周期为 'a，x 的生命周期为 'b
// 借用检查器在编译时发现 r 指向了生命周期为 'b 的一块内存 x，因为 'b 的生命周期比 'a 短，所以会编译失败
// 解决: 保证 x 的生命周期不小于 r 的生命周期 'a

fn main() {
    {
        let x = 5; // 移动 x 到 r 之前
        let r;
        {
            r = &x;
        }
        println!("r: {}", r); // x 的生命周期覆盖了 r 的生命周期，编译通过
    }
}
```

## 函数中的泛型生命周期
```rust
// 编译不通过，返回值包含了借用的值，但是**函数签名**没有说明这个借用的值是来自 x 还是 y
// - 编译器不晓得，返回类型的生命周期是和 x 还是和 y 一样
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// 加上泛型生命周期，编译通过
// - 'a 的实际生命周期是: x y 两个生命周期中较小的那一个 (两个生命周期重叠的那一部分)
// - 'a 并不会改变传入的参数的生命周期，只是向借用检查器指出了一些可用于检查非法调用的约束而已
// - 所以 longest 并不需要准确的知道传入的变量 x y 的存活时长
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// 可编译通过
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";
    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```
```rust
// 可编译通过
// 因为 string1 string2 在 result 被移出作用域之前都是有效的
fn main() {
    let string1 = String::from("abcd"); // 生命周期在 main 作用域内
    {
        let string2 = "xyz"; // &str 类型，拥有静态生命周期，在整个出现运行期间都存活
        let result = longest(string1.as_str(), string2); // 生命周期在当前这个内部作用域内
        println!("The longest string is {}", result);
    } // result 移出作用域
}
```
```rust
// 编译失败
// string2 存活时间不够长，其借用给了 result，但是生命周期比 result 短
// 这与 longest 函数签名注明的生命周期不一致
fn main() {
    let string1 = String::from("abcd"); // 生命周期在 main 作用域内
    let result; // 生命周期在 main 作用域内
    {
        let string2 = String::from("xyz"); // 生命周期在当前这个内部作用域内
        result = longest(string1.as_str(), string2.as_str());
    } // string2 移出作用域
    println!("The longest string is {}", result);
}
```
```rust
// pub fn strtok<'b, 'a>(s: &'b mut &'a str, delimiter: char) -> &'a str
// &'b mut: 指向字符串引用的可变引用 &mut 的生命周期
// &'a str: 字符串引用 &str 本身的生命周期
// 由于返回值只和 &str 有关, 所以可只需标注他的生命周期, 其他无关的生命周期可省略, 让编译器来自动添加
pub fn strtok<'a>(s: &mut &'a str, delimiter: char) -> &'a str {
    if let Some(i) = s.find(delimiter) {
        let prefix = &s[..i];
        // 由于 delimiter 可以是 utf8，所以我们需要获得其 utf8 长度，
        // 直接使用 len 返回的是字节长度，会有问题
        let suffix = &s[(i + delimiter.len_utf8())..];
        *s = suffix;
        prefix
    } else { // 如果没找到，返回整个字符串，把原字符串指针 s 指向空串
        let prefix = *s;
        *s = "";
        prefix
    }
}

fn main() {
    let s = "hello world".to_owned(); // str
    let mut s1 = s.as_str(); // &str
    let hello = strtok(&mut s1, ' '); // 访问 s1 的可变引用 
    println!("hello is: {}, s1: {}, s: {}", hello, s1, s); // 访问 s1 的不可变引用
    // print: "hello is: hello, s1: world, s: hello world"
}
// 注意!, strtok 生命周期标注不能写成如下样子
// 会导致 print s1(访问 s1 的不可变引用) 时报错:
// - cannot borrow `s1` as immutable because it is also borrowed as mutable
// 这是因为这样标注就表示可变借用 &mut 和 返回值的生命周期一样了, 导致这个引用的生命周期在函数结束后还没有结束
// 这样在 print s1 时, 就会发生可变引用和不可变引用同时存在的情况, 导致报错
/*
    let hello = strtok(&mut s1, ' '); // 访问 s1 的可变引用
    // 此时 s1 的生命周期 == hello 的生命周期
    println!("hello is: {}, s1: {}, s: {}", hello, s1, s); // 访问 s1 的不可变引用
*/
pub fn strtok<'a>(s: &'a mut &str, delimiter: char) -> &'a str
```

## 深入理解生命周期
- 指定生命周期参数的方式依赖于函数所做的事

```rust
// 由于返回值的生命周期只和 x 的生命周期有关，所以 y 不用标注生命周期
// fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```
- 当函数返回引用时，返回类型的生命周期参数需要与其中一个参数的生命周期匹配
- 如果返回的引用没有指向任何参数，那么它只能引用函数内创建的值
    - 这就是悬垂引用: 该值在函数结束时就走出了作用域

```rust
// 悬垂引用 (编译不通过)
// result 离开作用域了，这块内存就失效了
fn demo1() -> &str {
    let result = String::from("abc");
    result.as_str()
}

// 将所有权返回给调用方 (可编译通过)
fn demo2() -> String {
    let result = String::from("abc");
    result
}
```

## Struct 定义中的生命周期标注
- Struct 里可包括
    - 自持有的类型
    - 引用: 需要在每个引用上添加生命周期标注

```rust
struct ImportantExcerpt<'a> {
    // path 这个引用必须要比 ImportantExcerpt 的实例的存活要长
    part: &'a str,
}

// first_sentence 的生命周期能够覆盖 ie 的生命周期，所以编译通过
fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");

    let first_sentence = novel.split('.')
        .next()
        .expect("Could not found a '.'");

    let ie = ImportantExcerpt {
        part: first_sentence,
    };
}
```

## 生命周期的省略
- 我们都知道
    - 每个引用都有生命周期
    - 需要为使用生命周期的函数或 struct 指定生命周期参数

```rust
// 这个函数没有指定生命周期也能编译通过
// - 在 Rust 早期版本时无法编译通过的，当时要求每个引用都必须得有显式的生命周期
//   那时候这个函数得这么声明: fn first_word<'a>(s: &'a str) -> &'a str
// - 由于这类标注生命周期的规则都有相同的特征，所以编译器帮我们做了，不用程序员显式标注
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}
```
- 在 Rust 引用分析中所编入的模式称为**生命周期省略规则**
    - 这些规则无需开发者来遵守
    - 它们是一些特殊情况，由编译器来考虑
    - 如果你的代码符合这些情况，那么就无需显式标注生命周期
- 生命周期的省略规则不会提供完整的推断:
    - 如果应用规则后，引用的生命周期仍然模糊不清，还是会导致编译错误
    - 解决办法: 添加生命周期标注，表明引用间的相互关系

### 生命周期省略的三个规则
- 输入、输出生命周期
    - 函数/方法的参数: 输入生命周期
    - 函数/方法的返回值: 输出生命周期
- 编译器使用三个规则在没有显式标注生命周期的情况下，来确定引用的生命周期
    - 规则 1 应用于输入生命周期
    - 规则 2、3 应用于输出生命周期
    - 如果编译器应用完三个规则之后，仍然有无法确定生命周期的引用会编译报错
    - 这些规则适用于 `fn` 定义和 `impl` 块
- **规则一:** 每个引用类型的参数都有自己的生命周期
- **规则二:** 如果只有一个输入生命周期参数，那么该生命周期被赋给所有的输出生命周期参数
- **规则三:** 如果有多个输入生命周期参数，但其中一个是 `&self` 或 `&mut self` (是方法), 那么 `self` 的生命周期会被赋给所有的输出生命周期参数

```rust
fn test(s: &str) -> &str {...}
// 应用第一条规则: fn test<'a>(s: &'a str) -> &str
// 应用第二条规则: fn test<'a>(s: &'a str) -> &'a str
// 可以确定生命周期了，不用程序员显式标注

fn test2(x: &str, y: &str) -> &str {...}
// 应用第一条规则: fn test2<'a, 'b>(x: &'a str, y: &'b str) -> &str
// 由于有多个参数，无法应用第二条规则
// 由于不是方法，无法应用第三条规则
// 无法确定生命周期，需要程序员自己显式标注
```

## 方法定义中的生命周期标注
- 在 struct 上使用生命周期实现方法，语法和泛型参数的语法一样
- 在哪声明和使用生命周期参数，依赖于:
    - 生命周期参数是否和字段、方法的参数或返回值有关
- struct 字段的生命周期名:
    - 在 `impl` 后声明
    - 在 `struct` 名后使用
    - 这些生命周期是 struct 类型的一部分
- `impl` 块内的方法签名中
    - **引用**必须绑定于 struct 字段引用的生命周期，或者**引用**是独立的也可以
    - 生命周期省略规则经常使得方法中的生命周期标注不是必须的

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    // 由于第一条规则: &self 可不用标注生命周期
    fn level(&self) -> i32 {
        3
    }

    // 由于第一条规则和第三条规则，可不用标注生命周期
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        self.part
    }
}
```

## 静态生命周期
- `'static` 是一个特殊的生命周期: 表示整个程序的持续时间
    - 例如: 所有的字符串字面值都有 `'static` 生命周期 (该类型是被直接存储在二进制文件中的)
        - `let s: &'static str = "abc";`
- 为引用指定 `'static` 生命周期前要注意
    - 是否需要引用在程序整个生命周期内都存活

## 例子
- 使用了使用泛型参数类型、Trait Bound、生命周期的例子

```rust
fn longest_with_an_announcement<'a, T>
    (x: &'a str, y: &'a str, ann: T) -> &'a str
where
    T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```