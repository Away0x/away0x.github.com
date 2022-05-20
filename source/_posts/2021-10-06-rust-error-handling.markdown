---
layout: post
title: "Rust - Error Handling"
date: 2021-10-06 08:00:00 +0800
comments: true
categories: rust
---

<!-- more -->

* [Error Handling](#ErrorHandling)
* [thiserror](#thiserror)
* [anyhow](#anyhow)


## <h2 id="ErrorHandling">Error Handling</h2>
- 大部分情况下，Rust 在编译阶段就会提示错误
- 错误分类
    - 可恢复 (例如文件未找到，可再次尝试)
    - 不可恢复 (bug, 例如访问的索引超出范围)
- Rust 没有类似异常的机制
    - 可恢复错误: `Result<T, E>`
    - 不可恢复: `panic! 宏`
- 在定义一个可能失败的函数时，优先考虑返回 Result, 否则就 `panic!`

### 不可恢复的错误与 panic!
- 当 `panic!` 宏执行时:
    - 程序会打印错误信息
    - 展开 (unwind)、清理调用栈 (stack)
    - 退出程序

#### 为应对 panic，展开或中止 (abort) 调用栈
- 默认情况下，当 panic 发生
    - 程序展开调用栈 (这个工作量是很大的)
        - Rust 沿着调用栈往回走
        - 清理每个遇到的函数中的数据
    - 或立即中止调用栈
        - 不进行清理，直接停止程序
        - 内存需要稍后由操作系统进行清理
- 想让二进制文件更小，可以设置从 "展开" 改为 "中止"
    - `Cargo.toml` 中适当的 `profile` 部分设置 `panic = "abort"`

#### 查看 panic! 产生的回溯信息
```bash
set RUST_BACJTRACE=1 && cargo run
```

```rust
panic!("trigger panic");
```

### 可恢复的错误与 Result
#### Result Enum
- 和 Option Enum 一样，Result 和其变体也是由 prelude 带入作用域的

```rust
// 定义
enum Result<T, E> {
    Ok(T),
    Err(E),
}
// T: 操作成功情况下, 返回 Ok 变体里的数据的类型
// E: 操作失败情况下，返回 Err 变体里的数据的类型
```
```rust
// 使用 match 处理 Result
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            // 创建文件
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Error creating file: {:?}", e),
            },
            // 其他错误类型
            other_error => panic!("Error opening the file: {:?}", other_error),
        },
    };
}
```
```rust
// 使用 Result 闭包
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Error creating file: {:?}", error);
            })
        } else {
            panic!("Error opening the file: {:?}", error);
        }
    });
}
```

#### unwrap
- unwrap: match 表达式的一个快捷方法
    - 如果 Result 结果是 Ok, 返回 Ok 里面的值
    - 如果 Result 结果是 Err，调用 `panic!` 宏

```rust
fn main() {
    let f = match File::open("hello.txt") {
        Ok(file) => file,
        Err(error) => {
            panic!("Error opening the file: {:?}", error);
        },
    };

    // unwrap 等同于以上代码
    // 返回 Err 时，panic
    let f = File::open("hello.txt").unwrap();
}
```

#### expect
- expect: 和 unwrap 类似，但是可以指定错误信息

```rust
let f = File::open("hello.txt").expect("Error opening the file");
```

#### 传播错误与 ? 运算符
- `? 运算符` 只能应用于 Result 或者 Option 或者实现了 Try Trait 的类型

```rust
// 发生错误了，将错误交给调用方处理
fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = match File::open("hello.txt") {
        Ok(file) => file,
        Err(e) => return Err(e)
    };

    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => ok(s),
        Err(e) => Err(e),
    }
}

// 使用 `?` 运算符简化上面的传播错误代码
fn read_username_from_file2() -> Result<String, io::Error> {
    // let mut f = match File::open("hello.txt") {
    //     Ok(file) => file,
    //     Err(e) => return Err(e)
    // };
    let mut f = File::open("hello.txt")?; // 得到 Err 的话, 会直接 return Err, 等于上面的代码
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}

// 使用链式调用继续简化代码
fn read_username_from_file3() -> Result<String, io::Error> {
    let mut s = String::new();
    // 当某个 ? 操作符的地方得到 Err 了，会直接 return Err
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

##### ? 运算符与 from 函数
- Trait `std::convert::From` 上的 from 函数
    - 用于错误之间的转换
- 被 `?` 所应用的错误，会隐式的被 from 函数处理
    - 它所接收的错误类型会被转化为当前函数返回类型所定义的错误类型
- 用于: 针对不同错误原因，返回同一种错误类型
    - 前提: 需要错误类型实现了转化为所返回的错误类型的 from 函数

```rust
use std::convert::From;

impl From<Error1> for Error2 {
    fn from(_: Error1) -> Self {
        Error2 {}
    }
}
```

##### ? 运算符与 main 函数
```rust
// Box<dyn Error>: 是 trait 对象，可简单理解为代表任何可能的错误类型
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let mut f = File::open("hello.txt")?;
    Ok(())
}
```

### 自定义错误
```rust
// 不实现 Error trait 的错误
#[derive(Debug)]
struct CustomError(String);

fn main() -> Result<(), CustomError> {
    let path = "test.txt";
    let content = std::fs::read_to_string(path)
        .map_err(|err| CustomError(format!("Error reading `{}`: {}", path, err)))?;

    println!("file content: {}", content);
    Ok(())

    // Error: CustomError("Error reading `test.txt`: No such file or directory (os error 2)")
}
```
```rust
// 实现 Error trait 的错误
use std::error::Error;

#[derive(Debug)]
struct SuperError;
impl std::fmt::Display for SuperError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "SuperError")
    }
}

impl std::error::Error for SuperError {}


#[derive(Debug)]
struct CustomError {
    err: SuperError,
}

impl std::fmt::Display for CustomError {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "CustomError")
    }
}

impl std::error::Error for CustomError {
    // 可用于实现错误追溯
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> {
        Some(&self.err)
    }
}

fn new_custom_error() -> Result<(), CustomError> {
    Err(CustomError {
        err: SuperError,
    })
}

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // new_custom_error()?;

    match new_custom_error() {
        Err(e) => {
            println!("Error: {}", e);
            println!("Caused by: {}", e.source().unwrap());
        },
        _ => println!("Ok"),
    }

    Ok(())
}
```

## <h2 id="thiserror">thiserror</h2>
> 第三方库

- 为 `std::error::Error` 提供了方便的宏

```rust
#[derive(Debug, thiserror::Error)]
enum Error {
    #[error(transparent)]
    VarError(#[from] std::env::VarError), // 捕获 env::VarError 异常
    #[error(transparent)]
    IoError(#[from] std::io::Error)
}

fn main() -> Result<(), Error> {
    // std::env::var("不存在的环境变量")?; // 发生 std::env::VarError 异常
    std::fs::read_to_string("不存在的路径")?; // 发生 std::io::Error 异常
    Ok(())

    // Error: VarError(NotPresent)
    // Error: IoError(Os { code: 2, kind: NotFound, message: "No such file or directory" })
}
```

## <h2 id="anyhow">anyhow</h2>
> 第三方库

- 提供了 `anyhow::Error`
- 它的 Context trait 可以用来添加描述。除此之外，它还保留了原始 error ，因此我们会得到指向错误根源的 error

```rust
// Context trait 用来添加错误描述
// Result 会自动转换异常类型
use anyhow::{Context, Result};

fn main() -> Result<()> {
    let path = "test.txt";
    let content = std::fs::read_to_string(path)
        .with_context(|| format!("could not read file `{}`", path))?;

    println!("file content: {}", content);
    Ok(())
}

// Error: could not read file `test.txt`
// Caused by:
//     No such file or directory (os error 2)
```