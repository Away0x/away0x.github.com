---
layout: post
title: "Rust - Macro, Unsafe"
date: 2021-10-14 08:00:00 +0800
comments: true
categories: rust
---

<!-- more -->

* [Macro](#Macro)
* [Unsafe](#Unsafe)


## <h2 id="Macro">Macro</h2>
- macro 在 Rust 里指的是一组相关特性的集合称谓:
    - 使用 `macro_rules!` 构建声明宏 (declarative macro)
    - 三种过程 macro
        1. 自定义 `#[derive]` 宏, 用于 struct 或 enum, 可以为其指定随 derive 属性添加的代码
        2. 类似属性的宏, 在任何条目上添加自定义属性
        3. 类似函数的宏, 看起来像函数调用, 对其指定为参数的 token 进行操作

### 函数与宏的区别
1. 本质上，宏是用来编写可以生成其他代码的代码 (元编程 metaprogramming)
2. 函数在定义签名时，必须声明参数的个数和类型, 宏可处理可变的参数
3. 编译器会在解释代码前展开宏
4. 宏的定义比函数复杂得多, 难以阅读、理解、维护
5. 在某个文件调用宏时,必须提前定义宏或将宏引入当前作用域
6. 函数可以在任何位置定义并在任何位置使用

### 声明宏 `macro_rules!` (可能被弃用)
```rust
// 简化版的 vec! 实现
#[macro_export]
macro_rules! vec {
( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```
```rust
#![allow(unused)] // 允许代码中保留未使用的函数或变量，其是一个属性宏
macro_rules! calculate {
    // (eval 1 + 2)
    // $e:expr 表示 $e 这个宏变量的类型是 expr(表达式类型), 即 1 + 2
    (eval $e:expr) => { {
        {
            let val: usize = $e; // Force types to be integers
            println!("{} = {}", stringify!{$e}, val);
        }
    } };
}

fn main() {
    calculate! {
        eval 1 + 2
    }
    calculate! {
        eval (1 + 2) * (3 / 1)
    }
}
```

### 基于属性来生成代码的过程宏
- 会接收并操作输入的 Rust 代码, 生成另外一些 Rust 代码作为结果
- 有三种过程宏
    1. 自定义派生
    2. 属性宏
    3. 函数宏
- 创建过程宏时:
    - 宏定义必须单独放在它们自己的包中,并使用特殊的包类型

```rust
use proc_macro;

#[some_attribute] // 用于指定过程宏的占位符
pub fn some_name(input: TokenStream) -> TokenStream {}
```

### 类似函数的宏
- 函数宏定义类似于函数调用的宏, 但比普通函数更灵活

```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {}

// 使用 let sql = sql!(SELECT * FROM posts WHERE id=1);
```


## <h2 id="Unsafe">Unsafe</h2>
- Unsafe Rust: 没有强制的安全保证
- Unsafe Rust 存在的原因:
    1. 静态分析是保守的
        - 使用 Unsafe Rust 我知道自己在做什么, 并承担相应风险
    2. 计算机硬件本身就是不安全的, Rust 需要能够进行底层系统编程
- Unsafe 代码块里可执行的四个动作
    1. 解引用原始指针
    2. 调用 unsafe 函数或方法
    3. 访问或修改可变的静态变量
    4. 实现 unsafe trait
- 注意: 
    - unsafe 并没有关闭借用检查或停用其他安全检查
    - 任何内存相关的错误必须留在 unsafe 块里
    - 尽可能隔离 unsafe 代码,最好将其封装在安全的抽象里, 提供安全的 API

### 解引用原始指针
- 原始指针:
    - 可变的: `*mut T`
    - 不可变的: `*const T`. 意味着指针在解引用后不能直接对其进行赋值
    - 注意: 这里的 `*` 不是解引用符号, 它是类型名的一部分
- 与引用不同, 原始指针:
    - 允许通过同时具有不可变和可变指针或多个指向同一位置的可变指针来忽略借用规则
    - 无法保证能指向合理的内存
    - 允许为 null
    - 不实现任何自动清理
- 放弃保证的安全, 换取更好的性能/与其他语言或硬件接口的能力
- 为啥要使用原始指针:
    - 与 C 语言进行接口
    - 构建借用检查器无法理解的安全抽象

```rust
fn main() {
    let mut num = 5;

    // 可以在 unsafe 外面创建原始指针
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;

    // 只能在 unsafe 内对原始指针进行解引用
    unsafe {
        println!("{}", *r1);
        println!("{}", *r2);
    }

    let addr = 0x012345usize;
    let r3 = addr as *const i32; // 有问题的地址, 编译不会报错
    unsafe {
        println!("{}", r3); // 运行时报错
    }
}
```

### 调用 unsafe 函数或方法
- unsafe 函数或方法: 在定义前加上了 unsafe 关键字
    - 调用前需手动满足一些条件 (主要靠看文档), 因为 Rust 无法对这些条件就行验证
    - 需要在 unsafe 块里进行调用

```rust
unsafe fn dangerous() {}

fn main() {
    unsafe {
        dangerous();
    }
}
```

### 创建 unsafe 代码的安全抽象
- 函数包含 unsafe 代码并不意味着需要将整个函数标记为 unsafe
- 将 unsafe 代码包裹在安全函数中是一个常见的抽象

### 使用 extern 函数调用外部代码
- extern 关键字: 简化成绩和使用外部函数接口 (FFI) 的过程
    - FFI (Foreign Function Interface): 外部函数接口, 它允许一种编程语言定义函数, 并让其他编程语言能调用这些函数
- 可以使用 extern 来创建一个允许其他语言调用 Rust 函数的接口

```rust
// 任何在 extern 块里声明的函数都是 unsafe 的
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

### 访问或修改可变静态变量
```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```
```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

### 实现不安全 trait
- 当某个 trait 中存在至少一个方法拥有编译器无法校验的不安全因素时, 就称这个 trait 是不安全的

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```
