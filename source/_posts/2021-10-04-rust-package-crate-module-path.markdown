---
layout: post
title: "Rust - Package, Crate, Module, Path"
date: 2021-10-04 08:00:00 +0800
comments: true
categories: rust
---

<!-- more -->

* [Crate & Package](#CratePackage)
* [Module](#Module)
* [Path](#Path)

- Rust 的模块系统
    - Package: Cargo 的特性，让你构建、测试、共享 crate
    - Crate: 一个模块树，它可以产生一个 library 或可执行文件
    - Module、use 关键字: 让你控制代码的组织、作用域、私有路径
    - Path: 为 struct、function、或 module 等项命名的方式
- [例子](https://github.com/Away0x/rust-demo/blob/main/libconfig/src/conf/mod.rs)

## <h2 id="CratePackage">Crate & Package</h2>
- Create 的类型
    - binary
    - library
- Crate Root
    - 是源代码文件
    - Rust 编译器从这里开始(入口文件)，组成你的 Crate 的根 Module
- Package
    - 一个 Package 包含一个 `Cargo.toml`，它描述了如何构建这些 Crates
    - 只能包含 0-1 个 library crate
    - 可以包含任意数量的 binary crate
    - 但必须至少包含一个 crate (library 或 binary)

### Cargo 的惯例 (约定大于配置)
- `src/main.rs`
    - binary crate 的 crate root
    - crate 的名与 package 名相同
- `src/lib.rs`
    - package 包含一个 library crate
    - library crate 的 crate root
    - crate 的名与 package 名相同
- Cargo 把 crate root 文件交给 rustc 来构建 library 或 binary
- 一个 Package 可以同时包含 `src/main.rs` 和 `src/lib.rs`
    - 同时有时，表示该 Package 有一个 binary crate 和一个 libary crate
    - crate 名称与 package 名相同
- 一个 Package 可以有多个 binary crate
    - 可以把这些文件放在 `src/bin` 目录
    - 每个文件是单独的 binary crate

#### 惯例总结
- `tests`: 集成测试
    - 单元测试一般就写在各自的文件里面就行
- `beanches`: 性能测试
- `examples`:示例
- `src/lib.rs`: 库文件
- `src/main.rs`: 主二进制文件 (`cargo run` 会执行该文件)
- `src/bin`: 多个二进制文件 (如 `src/bin/demo.rs` 执行命令为 `cargo run --bin demo`)

### Crate 的作用
- Crate 可以将相关的功能组合到一个作用域内，便于在项目间进行共享
    - 防止冲突

### 使用外部 Package
1. `Cargo.toml` 添加依赖的 package
    - cargo 会从 `crates.io` 上下载依赖项
    - 下载完后可使用 `use` 将特定条目引入作用域
2. 标准库 (std) 也被当作外部包，但是不需要下载即可使用

### 发布 crate
#### release profile
- 是预定义的，可自定义(可使用不同的配置，对代码编译拥有更多的控制)
- 每个 profile 的配置都独立于其他的 profile
- Cargo 主要的两个 profile
    - dev profile: 适用于开发，`cargo build`
    - release profile: 适用于发布，`cargo build --release`
- 对于每个 profile, cargo 都提供了[默认的配置](https://doc.rust-lang.org/cargo/)
    - 可以在 `Cargo.toml` 里添加 `[profile.xxx]`，在里面覆盖默认配置的子集

```toml
# Cargo.toml (自定义 profile)

# cargo build 时 opt-level = 0
[profile.dev]
opt-level = 0 # 配置编译器的优化程度 (0~3)，值越高编译时间越慢

# cargo build --release 时 opt-level = 1
[profile.release]
opt-level = 3
```

### Cargo Workspaces
> 帮助管理多个相互关联且需要协同发布开发的 crate

- Workspaces 是一套共享同一个 `Cargo.lock` 和输出文件夹的包
- 可在 `Cargo.toml` 里配置 `[workspace]` 指定工作空间的 `members`
- workspace 下的 `Cargo.lock` 只有一个，在 workspace 的顶层目录
    - 保证了 workspace 内所有 crate 使用的依赖的版本都相同
    - workspace 内所有 crate 相互兼容
- [例子](https://github.com/Away0x/rust-demo/blob/main/workspace/README.md)

### 使用自定义命令拓展 cargo
- cargo 被设计成可以使用子命令来扩展
- 例: 如果 `$PATH` 中某个二进制是 `cargo-something`，你可以像子命令一样运行
    - `cargo something`
- 类似这样的自定义命令可以通过 `cargo --list` 列出
- 优点: 可使用 `cargo install` 来安装扩展，像内置工具一样来运行


## <h2 id="Module">Module</h2>
> Module 用来控制作用域和私有性

- Module
    - 在一个 crate 内，将代码进行分组
    - 增加可读性，易于复用
    - 可控制项目 (item) 的私有性: public、private
        - **默认都是私有的，需要公开可加 `pub` 关键字**
    - 使用 `mod` 关键字创建 Module, Module 可以嵌套

```rust
mod front_of_house() {
    mod hosting {
        fn add_to_waitlist() {}
        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}
```

## 将模块内容移动到其他文件
- 模块定义时，如果模块名后面时 `;`，而不是代码块
    - Rust 会从与模块同名的文件中加载内容
    - 模块树的结构不会变化
- 随着模块逐渐变大，该技术可以让你把模块的内容移动到其他文件中

```rust
// main.rs
// 会去同级目录的 aaa.rs 加载文件
mod aaa;

fn main() {
    aaa::bbb::func();
}
```
```rust
// aaa.rs
// 会去 aaa/bbb.rs 里面加载内容
pub mod bbb;
```
```rust
// aaa/bbb.rs
pub fn func() {}
```

## <h2 id="Path">Path</h2>
> 为了在 Rust 的模块中找到某个条目，需要使用路径

- Path 有两种形式
    1. 绝对路径: 从 crate root 开始，使用 crate 名或字面值 crate
    2. 相对路径: 从当前模块开始，使用 **self**, **super** 或当前模块的标识符
        - super: 类似文件系统中的 `..`，进入模块系统的上层目录
- Path 至少由一个标识符组成，标识符之间使用 `::` 连接

```rust
// lib.rs
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径: 使用字面值 crate
    crate::front_of_house::hosting::add_to_waitlist();
    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```
```rust
// lib.rs
fn a() {}

mod b {
    fn test() {
        super::a();
        crate::a();
    }
}
```

### use 关键字
- 可以使用 use 关键字将 Path 引入到作用域内
    - 仍遵循私有性规则

```rust
// lib.rs
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

// 引入当前作用域
use crate::front_of_house::hosting; // 绝对路径
// 由于 mod 都在同一级，所以直接相对路径 use front_of_house::hosting; 也是可以的

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

#### pub use (重新导出)
- 使用 use 将路径(名称)导入到作用域内后，该名称在此作用域内是**私有**的，加上 pub 即可使外部可访问
- 内部模块嵌套太深不方便外部使用时，也可以通过 `pub use` 重新导出，方便外部调用

```rust
// 外部可以访问到 func 了
pub use a::aa::func;
```

#### 嵌套 use
```rust
use std::{
    cmp::Ordering,
    io,
};
```
```rust
use std::io::{
    self, // 引入 std::io
    Write,
};
```

#### use *
```rust
// 将 io 下所有公共条目都引入到作用域
use std::io::*;
```

### Path 例子
```rust
#![allow(unused)]
fn main() {
    // 路径的第一种用法: 模块路径
    pub mod a {
        fn foo() { println!("a"); }

        pub mod b {
            pub mod c {
                pub fn foo() {
                    // a > b > c
                    super::super::foo();       // call a's foo function
                    self::super::super::foo(); // call a's foo function
                }
            }
        }
    }

    a::b::c::foo();

    // 路径的第二种用法: 方法调用
    struct S;
    impl S {
        fn f() { println!("s"); }
    }

    trait T1 {
        fn f() { println!("T1 f"); }
    }
    impl T1 for S {}

    trait T2 {
        fn f() { println!("T2 f"); }
    }
    impl T2 for S {}

    S::f(); // Calls the inherent impl, print "s"
    // 完全限定无歧义调用
    <S as T1>::f(); // Calls the T1 trait function, print "T1 f"
    <S as T2>::f(); // Calls the T2 trait function, print "T2 f"


    // 路径的第三种用法: 泛型函数 - turbofish 操作符 (泛型函数中使用 ::)
    // 调用 collect 时需要指定类型，于是使用 :: 指定类型后再调用
    (0..10).collect::<Vec<_>>();
    Vec::<u8>::with_capacity(1024);
}
```
