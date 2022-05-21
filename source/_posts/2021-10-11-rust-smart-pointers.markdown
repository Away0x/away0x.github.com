---
layout: post
title: "Rust - Smart Pointers"
date: 2021-10-11 08:00:00 +0800
comments: true
categories: rust
---

- 指针: 一个变量在内存中包含的是一个地址 (指向其他数据)
- Rust 中最常见的指针就是 "引用"
    - 使用 `&`
    - 借用它指向的值
    - 没有其余开销
    - 最常见的指针类型
- **智能指针:**
    - 行为和指针类似
    - 有额外的元数据和功能
- **引用**和**智能指针**的不同
    - 引用: 只借用数据
    - 智能指针: 很多时候都拥有它所指向的数据
- 常用的智能指针类型:
    - `Box<T>`: 在 heap 内存上分配值
    - `Rc<T>`: 启用多重所有权的引用计数类型
    - `Ref<T>`、`RefMut<T>`: 通过 `RefCell<T>` 访问，在运行时而不是编译时强制借用规则的类型
- `String` 和 `Vec<T>` 就是智能指针
    - 它们都拥有一片内存区域，且运行用户对其操作
    - 拥有元数据 (例如容量等)
    - 提供额外的功能或保障 (String 保障其数据时合法的 UTF-8 编码)
- **通常使用 struct 实现，并且实现了: `Deref` 和 `Drop` 两个 Trait**
    - Deref trait: 允许智能指针 struct 的实例像引用一样使用
    - Drop trait: 允许你自动移当智能指针实例走出作用域时的代码
- **内部可变模式 (interior mutability pattern):** 不可变类型暴露出可修改其内部值的 API

## `Box<T>` 类型
- 其是最简单的智能指针:
    - 允许你在 Heap 上存储数据 (而不是 stack)
    - stack 上是指向 Heap 数据的指针
    - 没有性能开销
    - 没有其他额外功能
    - 适用于 "间接" 存储的场景
    - 实现了 `Deref` 和 `Drop` 两个 Trait
- 使用场景:
    1. 在编译时，某类型的大小无法确定。但使用该类型时，上下文却需要知道它的确切大小
        - `Box<T>` 是一个指针, Rust 知道它需要多少空间 (指针的大小不会基于它指向的数据的大小变化而变化)
        - `Box<T>`
    2. 当有大量数据，想移交所有权，但需要确保在操作时数据不会被复制
    3. 使用某个值时，你只关心它是否实现了特定的 Trait，而不关心它的具体类型

```rust
fn main() {
    let a = Box::new(5); // 整数 5 存储在 heap 上了
    println("a = {}", a); // "a = 5"
} // a 被释放了 (会释放其存储在 stack 上的指针，以及存储在 heap 上的数据)
```

### Cons List (递归类型)
- 在编译时，Rust 需要知道一个类型所占的空间大小
- 而递归类型的大小无法在编译时确定, 但 Box 类型的大小确定，在递归类型中使用 Box 就可解决上述问题
- 这种类型 `Cons List` 是来自 Lisp 的一种数据结构
    - Cons List 里每个成员由两个元素组成
        1. 当前项的值
        2. 下一个元素
    - Cons List 里最后一个成员只包含一个 Nil 值，没有下一个元素

```rust
// 使用 Rust 实现 Cons List
use crate::List::{Cons, Nil};

enum List {
    Cons(i32, Box<List>),
    Nil,
}

fn main() {
    // Box<T> 提供了 "间接" 存储和 heap 内存分配的方式
    let list = Cons(1, 
        Box::new(Cons(2, 
            Box::new(Cons(3, 
                Box::new(Nil))))));

    println!("{:?}", list); // Cons(1, Cons(2, Cons(3, Nil)))
}
```

## Deref Trait
- 实现该 Trait 可使我们能**自定义解引用运算符 `*` 的行为**
- 通过实现 Deref，智能指针可**像常规引用一样来处理**
- 解引用运算符 `*`
    - 常规的引用也是一种指针

```rust
let x = 5;
let y = &x;

assert_eq!(5, x); // ok
assert_eq!(5, *y); // ok

// 使用 Box<T> 替代上面的引用
let y = Box::new(x);
assert_eq!(5, *y); // ok
```
```rust
// 自定义智能指针
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl <T>std::ops::Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);
    assert_eq!(5, *y); // ok 
    // *y 相当于 *(y.deref())
}
```

### Deref Coercion
- Deref Coercion (函数和方法的隐式解引用转化) 是为函数和方法提供的一种便捷特性
- 假设 `T` 实现了 Deref trait
    - Deref Coercion 可以把 `T` 的引用转化为 `T` 经过 Deref 操作后生成的引用
- 当把某类型的引用传递给函数或方法时，但它的类型与定义的参数类型不匹配:
    - Deref Coercion 就会自动发生
    - 编译器会对 deref 进行一系列调用，来把它转为所需的参数类型 (在编译时完成，没有额外的性能开销)

```rust
fn hello(name: &str) {
    println!("Hello, {}", name);
}

fn main() {
    let m = MyBox::new(String::from("world"));
    // &m: &MyBox<String>
    // - 由于 MyBox 实现了 deref trait，所以编译器可以把 &MyBox<String> 转化为 &String
    // - 由于标准库中 String 也实现了 deref trait，调用 deref 方法返回一个 &str 类型
    // - 所以 &m 可以作为参数传入，满足 &str 的参数类型要求
    hello(&m);
    // 如果没有实现 deref trait，则这里得这样调用: hello(&(*m)[..])
}
```

### 解引用与可变性
- 可使用 `DerefMut Trait` 重载可变引用 `*` 运算符
- 在类型和 trait 在下列三种情况发生时，Rust 会执行 Deref Coercion:
    1. 当 `T: Deref<Target=U>`，允许 `&T` 转换为 `&U`
    2. 当 `T: DerefMut<Target=U>`，允许 `&mut T` 转换为 `&mut U`
    3. 当 `T: Deref<Target=U>`，允许 `&mut T` 转换为 `&U`

## Drop Trait
- 类似实现 `Drop Trait`，可以让我们自定义**当值将要离开作用域时发生的动作**
    - 例如: 文件、网络资源释放等
- 任何类型都可以实现 Drop Trait, 该 trait 在 prelude 里面

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("drop");
    }
}

fn main() {
    let a = CustomSmartPointer { data: String::from("hello") };
} // "drop"
```
- Rust 不允许手动调用 Drop trait 的 drop 方法
    - 但可以调用标准库的 `std::mem::drop` 函数，来提前 drop 值

## `Rc<T>` 引用计数智能指针
- 有时, 一个值会有多个所有者, 为了**支持这种多重所有权的情况**, Rust 实现了 `Rc<T>`
    - RC: reference counting (引用计数)
    - 该类型会在实例内部维护一个用于记录值引用次数的计数器, 用于判断这个值是否被使用 (可以追踪值的所有引用)
    - 0 个引用: 该值可以被清理掉
    - `Rc<T>` 通过不可变引用, 使我们可以在程序不同部分共享**只读**数据
- 使用场景: 需要在 heap 上分配数据, 这些数据被程序的多个部分读取(只读), 但在编译时无法确定哪个部分最后使用完这些数据
- `Rc<T>` 只能用于单线程场景, 多线程场景可以使用 `Arc<T>`
    - Rc 为了性能使用的不是线程安全的引用计数器
    - Arc 内部引用计数使用了 atomic usize, 线程安全
- `Rc::clone(&a) 函数`: 会增加引用计数
- `Rc::strong_count(&a)`: 获得引用计数
    - 还有 `Rc::weak_count` 函数

```rust
use std::rc::Rc;

fn main() {
    let a = Rc::new(1);
    // 对一个 Rc 结构进行 clone()，不会将其内部的数据复制，只会增加引用计数
    let b = a.clone();
    let c = a.clone();

    // 作用域有三个 Rc(a, b, c), 它们共同指向堆上相同的数据, 即堆上的数据有 3 个共享的所有者
    // 而当一个 Rc 结构离开作用域被 drop() 时，也只会减少其引用计数，直到引用计数为零，才会真正清除对应的内存
}
```
```rust
// 两个 List 共享另一个 List 的所有权
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    // b, c 共享 a 的所有权
    let b = Cons(3, Rc::clone(&a)); // 引用计数+1
    {
        let c = Cons(4, Rc::clone(&a)); // 引用计数+1
    } // c 离开作用域了, 引用计数自动 -1
    // a.clone() 该方法会进行深度的 copy; Rc::clone 只会增加引用计数, 不会执行数据的深度 copy
    // a 在引用计数清零的时候才会被回收
}
```

## `RefCell<T>`
- 内部可变性: 是 Rust 的设计模式之一, 它允许你在只持有不可变引用的前提下对数据进行修改
    - 数据结构中使用了 unsafe 代码来绕过 Rust 正常的可变性和借用规则
- 与 `Rc<T>` 不同, `RefCell<T>` 类型代表了其持有数据的唯一所有权
    - Rc 是个只读的引用计数器, 无法拿到 Rc 结构内部数据的可变引用
- `RefCell<T>` 和 `Box<T>` 的区别
    - `RefCell<T>`: 只会在运行时检查借用规则, 运行时不满足借用规则会触发 panic
    - `Box<T>`: 在编译阶段强制代码遵守借用规则, 不满足规则会出现错误
- 借用规则在不同阶段进行检查的比较:
    1. 编译阶段:
        1. 尽早暴露问题
        2. 没有任何运行时开销
        3. 对大多数场景是最佳选择
        4. 是 Rust 的默认行为
    2. 运行时:
        1. 问题暴露延后, 甚至到生产环境
        2. 由于使用了借用计数, 会产生些许性能损失
        3. 实现某些特定的内存安全场景 (不可变环境中修改自身数据)
- 与 `Rc<T>` 相似, 只能用于**单线程**场景
    - 多线程场景使用需要使用 Mutex(互斥量) 和 RwLock(读写锁)
    - 例如: `Rc<RefCell<T>>` 在多线程场景下可替换为 `Arc<Mutex<T>>` 或 `Arc<RwLock<T>>`

### 内部可变性: 可变的借用一个不可变的值
- 外部可变性: 使用 `let mut`, `&mut`
    - 所有权检查: 编译时, 如果不符合规则, 产生编译错误
- 内部可变性: 使用 `Cell`, `RefCell`
    - 所有权检查: 运行时, 如果不符合规则, panic

```rust
fn main() {
    let x = 5;
    let y = &mut x; // 报错, 无法可变的借用一个不可变的值
    // 有时会需要有这一种场景, 值在外部是不可变的, 但是在方法内部需要可以修改这个值
    // 可以使用 RefCell 来实现这种内部可变性
}
```

### 使用 `RefCell<T>`
- `RefCell<T>` 的方法
    - `borrow` 方法: 返回 `Ref<T>`, 它实现了 Deref trait
    - `borrow_mut` 方法: 返回 `RefMut<T>`, 它实现了 Deref trait
- `RefCell<T>` 会记录当前存在多少个活跃的 `Ref<T>` 和 `RefMut<T>` 智能指针
    - 每次调用 borrow: 不可变借用计数 +1
    - 任何一个 `Ref<T>` 的值离开作用域被释放时: 不可变借用计数 -1
    - 每次调用 borrow_mut: 可变借用计数 +1
    - 任何一个 `RefMut<T>` 的值离开作用域被释放时: 可变借用计数 -1
- Rust 通过以上计数规则来维护借用的检查规则
    - 任何一个给定时间里, 只允许拥有多个不可变借用活一个可变借用

```rust
use std::cell::RefCell;

fn main() {
    let data = RefCell::new(1);
    // 在同一个作用域下，我们不能同时有活跃的可变借用和不可变借用
    // 通过这对花括号，我们明确地缩小了可变借用的生命周期，不至于和后续的不可变借用冲突
    { 
        // 获得 RefCell 内部数据的可变借用
        let mut v = data.borrow_mut();
        *v += 1;
    }
    println!("data: {:?}", data.borrow()); // data: 2
}
```

### 如何选择 `Box<T>`、`Rc<T>`、`RefCell<T>`
| | `Box<T>` | `Rc<T>` | `RefCell<T>` |
| - | - | - | - |
| 同一数据的所有者 | 一个 | 多个 | 一个 |
| 可变性、借用检查 | 可变、不可变借用 (编译时检查) | 不可变借用 (编译时检查) | 可变、不可变借用 (运行时检查) |

### 结合 `Rc<T>` 和 `RefCell<T>` 来拥有多个可变数据所有者
```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

### 其他可实现内部可变性的类型
- `Cell<T>`: 通过复制来访问数据
- `Mutex<T>`: 用于实现跨线程情形下的内部可变性模式
