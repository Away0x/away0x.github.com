---
layout: post
title: "Rust - Install, Basic"
date: 2021-10-01 08:00:00 +0800
comments: true
categories: rust
---

<!-- more -->

* [Install](#install)
* [Basic](#basic)


## <h2 id="install">Install</h2>
> 如安装困难, 可参考 https://rsproxy.cn/ 配置 proxy

### rustup
> Rust 安装和管理工具

```bash
# 安装 rustup
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
# 验证是否安装成功
rustc --version 
# 更新 rust 版本
rustup update
# 更新 rustup
rustup self update
# 卸载 rust
rustup self uninstall
# 打开本地 rust 文档
rustup doc
```

### cargo
> Rust 的 package 管理工具

- [Cargo book](https://doc.rust-lang.org/cargo/index.html)
- [crates.io](https://crates.io/)

```bash
cargo --version
# 创建项目
# --lib 库项目
# --vcs=git 指定版本控制工具
cargo new demo
# 编译, --release 发布时编译
cargo build
# 清除 cargo 构建文件
cargo clean
# 检查代码 (不生成可执行文件)
cargo check
# 编译并运行
cargo run
# 安装二进制文件
cargo install <crate-name> # 会安装到 ～/.cargo/bin
cargo install --force <crate-name> # 重新安装
# 生成文档并打开
cargo doc --open
# 发布 crate
cargo publish
cargo yank # 撤回版本
```
```bash
# cargo 常用的第三方插件, 需要 cargo install 这些插件才可运行
# 自动修复 warning
cargo fix
# 添加项目依赖
cargo add
# 审核项目依赖是否有漏洞风险
cargo audit
# lint 工具
cargo clippy
# 格式化代码
cargo fmt
# 可展开宏代码, 方便调试
cargo expand
```

## <h2 id="basic">Basic</h2>
- Rust 语言学习的难点
    1. 所有权机制
    2. 借用和生命周期
    3. 类型系统与 trait
    4. 突破抽象范式
    5. Unsafe Rust
- Rust 语言的内存安全方案针对的是 C 语言的不足
    1. 禁止对空指针和悬垂指针进行解引用
    2. 读取未初始化的内存
    3. 缓冲区溢出
    4. 非法释放已经释放或未分配的指针
- Rust 可安全且无缝地沟通 C 语言
    1. 通过 C-ABI 零成本和 C 语言打交道
    2. 划分了 Safe Rust 和 Unsafe RUst
- 函数和变量名使用 snake case 命名规范
- trait、struct 使用 camel case 命名规范

### 注释
```rust
// 单行注释
/**
    多行注释
*/
```
```rust
//! 这是一个模块级别的文档注释

/// 这是一个函数的文档注释 (可使用 markdown 语法)
/// 可以使用 cargo doc --open 生成文档
/// 
/// # Examples
/// ```
/// // 编写一些示例 (运行 cargo test 的时候，会把示例的代码作为测试来运行)
/// let arg = 5;
/// let answer = add_one(arg);
/// assert_eq!(6, answer);
/// ```
/// 
/// # Panics
/// 说明函数可能发生 panic 的场景
/// 
/// # Errors
/// 如果函数返回 Result，描述可能的错误种类，以及可导致错误的条件
/// 
/// # Safety
/// 如果函数处于 unsafe 调用，就应该解释 unsafe 的原因，以及调用者确保的使用前提
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

### Print
```rust
// print!、println!、eprint!、eprintln!
println!("{}", 2);
println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob");
println!("{a} {b} {c}", a="a", b="b", c="c");
```

### Variable
- 使用 let 声明变量，默认情况下变量是 Immutable 的
- 声明变量时，加上 mut，就可是变量可变
- Shadowing
    - 可以使用相同的名字声明新的变量，新的变量会 shadow 之前声明的同名变量

```rust
let x: u32 = 5; // 在栈上分配一个类型为 u32 的变量, 变量名为 x
x = 6; // Error: cannot assign twice to immutable variable

let mut y = 5;
y = 6; // OK
y = "y"; // 类型错误
```
```rust
// shadow
let x = 5;
let x = x + 1;
let x = "x";
```
```rust
// 解构
let (mut a, mut b) = (1, 2);
let Point { x: ref a, y: ref b } = p;
```

### Constant
- 常量在绑定值后也是不可变的，但是它和不可变的变量有很多区别:
    1. 不可以使用 mut, 常量永远都是不可变的
    2. 声明常量使用 const 关键字，类型必须得被标注
    3. 常量可以在任何作用域内声明，包括全局作用域
    4. 常量只可以绑定到常量表达式，无法绑定到函数的调用结果或只能在运行是才能计算出的值
- 程序运行期间，常量在其声明的作用域内一直有效
- 命名规范: 使用全大写字母，单词之间用下划线分隔

```rust
const MAX_POINTS: u32 = 100_1000;
```

### Static Variable
- 与常量的区别
    1. 静态变量有固定的内存地址, 使用它的值总会访问同样的数据
    2. 常量允许使用它们的时候对数据进行复制
    3. 静态变量: 可以是可变的, 访问和修改静态可变变量是不安全的 (unsafe)

```rust
// 全局变量
// 声明时必须初始化，且初始化必须为编译期可确定的常量
// 全局变量读写时需要用 unsafe 修饰
static GLOBAL: i32 = 0; // 生命周期('static)为整个程序运行期间
```
```rust
// 使用静态变量时, 由于一些限制, 可以使用 lazy_static 工具
// https://github.com/rust-lang-nursery/lazy-static.rs
static MAP: HashMap<String, String> = HashMap::new(); // 无法编译通过, 需要用 lazy_static
```
```rust
use lazy_static::lazy_static;
use std::collections::HashMap;
use std::sync::{Arc, Mutex};

lazy_static! {
    static ref HASHMAP: Arc<Mutex<HashMap<u32, &'static str>>> = {
        let mut m = HashMap::new();
        m.insert(0, "foo");
        m.insert(1, "bar");
        m.insert(2, "baz");
        Arc::new(Mutex::new(m))
    };
}

fn main() {
    let mut map = HASHMAP.lock().unwrap();
    map.insert(3, "waz");

    println!("map: {:?}", map);
}
```

### Types
- Rust 是静态编译语言，在编译时必须知道所有变量的类型
    - 基于使用的值，编译器通常能够推断出它的具体类型
    - 但如果可能的类型比较多(例如把 String 转为整数的 parse 方法)，就必须添加类型标注，否则编译会报错

```rust
let guess: u32 = "42".parse().expect("Not a number");
```

#### 标量类型
- 一个标量类型代表一个单个的值

##### 整数类型
- 整数类型没有小数部分
    - 例如 u32 就是一个无符号的整数类型，占据 32 位的空间
- 无符号整数类型以 u 开头
- 有符号整数类型以 i 开头

| Length | Signed | Unsigned |
| - | - | - |
| 8-bit | i8 | u8 |
| 16-bit | i16 | u16 |
| 32-bit | i32 | u32 |
| 64-bit | i64 | u64 |
| 128-bit | i128 | u128 |
| arch | isize | usize |

- isize 和 usize 类型的位数由程序运行的计算机的架构决定
    - 64 位计算机，那就是 64 位的
- 使用 isize/usize 的主要场景是对某种集合进行索引操作

###### 整数字面值
- 除了 byte 类型外，所有的数值字面值都允许使用类型后缀
    - 例如: `57u8`
- 整数的默认类型就是 **i32**

| Number Literals | Example |
| - | - |
| Decimal | `98_222` |
| Hex | `0xff` |
| Octal | `0077` |
| Binary | `0b1111_0000` |
| Byte(u8 only) | `b'A'` |

###### 整数溢出
- u8 范围是 0-255，如把 u8 变量的值设置为 256，那么:
    1. 调试模式下编译: Rust 会检查整数溢出，如发生, 程序运行时会 panic
    2. release 模式下编译: Rust 不会检查可能导致 panic 的整数溢出，如溢出发生，会执行 "环绕操作"
        - 256 变成 0，257 变成 1 ...
        - 但程序不会 panic

##### 浮点类型
- **f32**: 32 位，单精度
- **f64**: 64 位，双精度
- f64 是默认类型，因为现代 CPU 上 f64 和 f32 速度差不多，而且精度更高

```rust
let x = 2.0; // f64
let y: f32 = 3.0; // f32
```

###### 数值操作
```rust
let a = 5 + 10;
let b = 3.2 - 2.1;
let c = 2 * 3;
let d = 30 / 3;
let e = 54 % 5;
```

##### 字符类型 char
- Rust 语言中 char 类型被用来描述语言中最基础的单个字符
- 字符类型的字面值使用单引号
- 占用 4 字节大小
- 是 Unicode 标量值，可以表示比 ASCII 多得多的字符内容: 拼音、中日韩文、零长度空白字符、emoji 表情...
    - Unicode 中并没字符的概念，所以直觉上认为的字符也许和 Rust 中的概念并不相符

```rust
let x = 'z';
let y: char = '『';
let z = '😄';
```
```rust
use std::char;

fn main() {
    let tao = '道';
    let tao_u32 = tao as u32;
    assert_eq!(36947, tao_u32);
    println!("U+{:x}", tao_u32); // U+9053
    println!("{}", tao.escape_unicode()); // \u{9053}
    assert_eq!(char::from(65), 'A');
    assert_eq!(char::from_u32(0x9053), Some('道'));
    assert_eq!(char::from_u32(36947), Some('道'));
    assert_eq!(char::from_u32(129010101), None);
}
```

##### 布尔类型
- true/false
- 可转换为数字类型，反之不可

#### 复合类型
- 复合类型可以将多个值放在一个类型里
- Rust 提供了两种基础的复合类型: 元组(Tuple)、数组

##### Tuple
- 可以将多个类型的多个值放在一个类型里面，每个位置的类型不必相同
- 长度是固定的，一旦声明就无法更改 (不同长度的元组是不同类型)
- 其是 Rust 中唯一可异构的序列
- 单元类型的唯一实例等价于空元组
- 当元组只有一个元素的时候，要在元素末尾加逗号分隔，这是为了方便和括号操作符区分开来

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
println!("{} {} {}", tup.0, tup.1, tup.2);
// 使用模式匹配结构 tuple
let (x, y, z) = tup;

// Empty Tuple (Unit 类型)
let empty: () = (); // 空元组占用 0 内存空间
```

##### 数组
- 可以将多个类型的多个值放在一个类型里面，每个位置的类型必须相同
- 长度是固定的，一旦声明就无法更改
    - `[T;2]` 和 `[T;3]` 是不同的类型
- 如想让数据存放在 stack 上而不是 heap 上，或者想保证有固定数量的元素，这时使用数组更有好处
    - 数组是在 stack 上分配的单个块的内存
- 数组没有 Vector 灵活

```rust
let a = [1, 2, 3];
// 数组的类型 [类型; 长度]
let b: [u32; 2] = [1, 2];

// 如果数组中每个元素值都相同可以这样声明
let c = [3; 5]; // 相当于 let c = [3, 3, 3, 3, 3];

// 访问数组元素
// - 如果索引超出数组范围，编译会报错
// - 如果索引是动态的，那么编译会通过，运行时会报错
let d = c[0];
```

#### 指针类型
- Rust 包含了三种指针类型
    1. 原始指针: `*const T`, `*mut T`
        - 等价与 C 语言的指针，他可以为空，一般用于 Unsafe Rust 中
    2. NonNull 指针，他是 Rust 语言建议的 `*mut T` 指针的替代指针
        - NonNull 指针是非空指针，并且遵循生命周期类型协变规则
    3. 函数指针 `fn`，指向代码的指针，非数据，可以使用它直接调用函数

#### 智能指针
| 类型名 | 简介 |
| - | - |
| `Box<T>` | 指向类型 T 的、具有所有权的指针，有权释放内存 |
| `&T` | 指向类型 T 的借用指针、也称为引用，无权释放内存，无权写数据 |
| `&mut T` | 指向类型 T 的 mut 型借用指针，无权释放内存，有权写数据 |
| `*const T` | 指向类型 T 的只读裸指针，没有生命周期信息，无权写数据 |
| `*mut T` | 指向类型 T 的可读写裸指针，没有生命周期信息，有权写数据 |
| `Rc<T>` | 指向类型 T 的引用计数指针，共享所有权，线程不安全 |
| `Arc<T>` | 指向类型 T 的原子引用计数指针，共享所有权，线程安全 |
| `Cow<'a, T>` | Clone-on-write, 写时复制指针。可能是借用指针，也可能是具有所有权的指针 |

#### Never 类型
- 名为 `!` 的特殊类型, 没有任何值, 在不返回值的函数中充当返回类型
    - 不返回值的函数也被称为发散函数 (diverging function)
    - never 可强转为任何其他类型

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    // continue 返回 never 类型, never 可强转为任何其他类型,
    // 所以该分支可以使用 continue 作为 u32 类型
    Err(_) => continue,
};
```
```rust
fn bar() -> ! {
    loop {} // 此处永远没有返回值, 所以类型为 never
}

impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            // panic 是 never, 所以可以作为 T 使用
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
```

#### 特定类型
> 指专门有特殊用途的类型

1. `PhantomData<T>`, 幻影类型
    - 一般用于 Unsafe Rust 的安全抽象
2. `Pin<T>`, 固定类型
    - 为了支持异步开发而特意引入，防止被引用的值发生移动的类型

#### 泛型
- `::<T>` turbofish 操作符: 用于给泛型函数指定类型, 对于编译器可推断的类型，是可以省略的

```rust
fn foo<T>(x: T) -> T {
    return x;
}

assert_eq!(foo(1), 1);
assert_eq!(foo("abc"), "abc");

// rust 如果类型推断失败，则需要手工指定类型
foo(1);     // 等价于 foo::<i32>(1);
foo("abc"); // 等价于 foo::<&'static str>("abc");
```

#### 类型转换
- Rust 不提供原生类型之间的隐式转换，得使用 `as` 关键字显式转换
- 相似类型的转换:
    - `as`
    - `T.into()`: 变量需要显式标注类型

```rust
let v1: i8 = 41;
let v2: i16 = v1 as i16; // 也可 v1.into()

let i = 42;
let p = &i as *const i32 as *mut i32;
println!("{:?}", p);
```

#### 类型别名
```rust
type Age = u32;
type Double<T> = (T, Vec<T>);
```

### Function
#### 表达式
- 函数体由一系列语句组成，可选的由一个表达式结束
- **Rust 是一个基于表达式的语言**
    - 语句是执行一些动作的指令
    - 表达式会计算产生一个值
- 语句没有返回值，所以不可以使用 let 将一个语句赋值给一个变量

```rust
let y = 6; // 语句
let x = (let y = 6); // 报错

let y = {
    let x = 1;
    x + 3 // 表达式产生的值 (不加分号, 加分号产生的值位 unit type)
};
```
```rust
// 分号表达式返回值永远为自身的单元 (Unit) 类型: `()`
// 分号表达式只有在块表达式最后一行才会进行求值，其他时候只作为连接符存在
// 块表达式只对其最后一行表达式进行求值
fn main() { // -> ()
    ; // 作为连接符
    ; // 作为连接符
    {
        ()
    }
    {
        ();
        use std:vec::Vec; // 以 ; 结尾，所以该块返回 () 类型
    }
    ();   // -> ()
    &{;}; // 块表达式最后一个分号返回 Unit，与 & 结合产生 &() 类型，该行最后一个分号是连接符
        ;
        ; // 作为 main 函数的返回值 ()
}
// 想获取 &() 类型，不能通过 `&;` 产生，因为不符合求值规则，这里的分号是作为连接符存在的
// 得通过 `&{;}` 分号作为块表达式的结尾才会使分号能够求值，产生 &() 类型
```
```rust
// 声明函数 (函数的定义也是语句)
fn test() {
    println!("test");
}
fn test2(x: i32, y: i32) {}
test2(5, 6);

// 返回值是函数体里最后一个表达式的值
// 如想提前返回，可以使用 return 关键字，指定一个值
fn test3(x: i32) -> i32 {
    x + 5
}
let x = test3(6); // 11
```

#### 语句
- 声明语句
    - 例如 `use std::io;` 就是一个声明语句
- 表达式语句 (用来求值的语句)
    - 块中最后一行不加分号的表达式, 其会用于求值并返回
    - 除了基本声明语句外，其余基本都是表达式，**流程控制也是表达式**
        - 自变量，路径，数组结构体枚举函数闭包， 方法调用，字段访问操作符...都是表达式

```rust
fn plus_one(i &u32) -> u32 {
    let i = i + 1;
    i // 不加分号，求值并返回 (如果加了分号，返回的是 Unit Type)
    // 也可使用 return 进行显示返回, 使用 return 时，末尾加不加分号都会返回表达式的值
}
```

### Control Flow
#### if else
- if 是表达式，会产生值，if 条件必须得是 bool 类型
- 如果程序中使用了多于一个的 else if，那么最好使用 match 来重构

```rust
let number = 6;
if number % 4 == 0 {
    println!("number is divisible by 4");
} else if number % 3 == 0 {
    println!("number is divisible by 3");
} else {
    println!("number is not divisible by 4 or 3");
}

// if 是表达式
let condition = true;
let number = if condition { 5 } else { 6 }; // 每个分支产生的值，类型必须一致
```

#### loop
```rust
loop {
    println!("again!");
}

let mut counter = 0;
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2; // 作为表达式结果
    }
};

println!("The result is: {}", result); // The result is: 20
```

#### while
```rust
let mut number = 3;
while number != 0 {
    println!("{}", number);
    number = number - 1;
}
```

#### for
> for 可以用于任何实现了 IntoIterator trait 的数据结构

在执行过程中，IntoIterator 会生成一个迭代器，for 循环不断从迭代器中取值，直到迭代器返回 None 为止。因而，for 循环实际上只是一个语法糖，编译器会将其展开使用 loop 循环对迭代器进行循环访问，直至返回 None

```rust
let a = [10, 20, 30, 40, 50];
for element in a {
    println!("the value is: {}", element);
}
```
```rust
let v = vec![10, 42, 9, 8];
for item in v {
    println!("{}", item);
}

let v = vec![10, 42, 9, 8];
for (index, value) in v.iter().enumerate() {
    println!("index: {}, value: {}", index, value);
}
```

##### for 搭配 range 使用
```rust
for number in (1..4).rev() {
    println!("{}", number);
}
// rev 倒序循环
// 依次输出 3 2 1
```