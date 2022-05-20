---
layout: post
title: "Rust - Ownership, Borrowing"
date: 2021-10-02 08:00:00 +0800
comments: true
categories: rust
---

<!-- more -->

* [Ownership](#Ownership)
* [Borrowing](#Borrowing)

## <h2 id="Ownership">Ownership</h2>
> 所有权是 Rust 最独特的特性，它让 Rust 无需 GC 就可以保证内存安全

- 所有程序在运行时都必须管理它们使用计算机内存的方式
    - 有些语言有垃圾收集机制，在程序运行时，它们会不断的寻找不再使用的内存
    - 而有些语言中，程序员必须显式地分配和释放内存
    - Rust 则时通过所有权系统来管理，其中包含一组编译器在编译时检查的规则, 当程序运行时，所有权特性不会减慢程序的运行速度 (因为 Rust 把内存管理相关的工作都提前到了编译时)

### Stack vs Heap
在 Rust 这样的系统级编程语言来说，一个值是在 stack 上还是在 heap 上对语言的行为和你为什么要做某些决定是由很大的影响的

- Stack 会按值的接收顺序来存储，按相反的顺序将它们移除 (后进先出, LIFO)
    - 添加数据(压入栈)，移除数据(弹出栈)
    - **所有存储在 Stack 上的数据必须拥有已知的固定的大小**
- 编译时大小未知的数据或运行时大小可能发生改变的数据必须放在 heap 上

#### 存储数据
- Heap 对内存的组织性差一些
    - 当把数据放入 heap 时，会请求一定数量的空间
    - 操作系统在 heap 里找到一块足够大的空间，把它标记在用，并返沪一个指针，也就是这个空间的地址 (这个过程叫做在 heap 上进行**分配**)
- 把值压到 stack 上不叫分配
- 因为指针是已知固定大小, 可以把指针存放在 stack 上
    - 但如果想要实际数据，必须使用指针来定位
- 把数据压到 stack 上要比 heap 上分配快得多 (因为操作系统不需要寻找用来存储新数据的空间，那个位置永远都在 stack 的顶端)
- 在 heap 上分配空间需要更多的工作 (操作系统首先需要找到一个足够大的空间来存放数据，然后要做好记录方便下次分配)

#### 访问数据
- 访问 heap 中的数据要比访问 stack 中的数据慢，因为需要通过指针才能找到 heap 中的数据
    - 对于现代处理器来说，由于缓存的缘故，如果指令在内存中跳转的次数越少，那么速度就越快
- 如果数据存放的距离比较近，那么处理器的处理速度就会更快一些 (stack 上)
- 如果数据之间的距离比较远，那么处理速度就会慢一些 (heap 上)
    - 在 heap 上分配大量的空间也是需要时间的

#### 函数调用
当代码调用函数时，值被传入函数(也包括指向 heap 的指针). 函数本地的变量被压到 stack 上. 当函数结束后，这些值会从 stack 上弹出

### 所有权解决的问题
> 管理 heap 数据是所有权存在的原因

1. 跟踪代码的哪些部分正在使用 heap 的哪些数据
2. 最小化 heap 上的重复数据量
3. 清理 heap 上未使用的数据以避免空间不足

### 所有权规则 (保证单一所有权)
- 每个值都有一个变量，这个变量是该值的所有者
- 每个值同时只能有一个所有者 (Move 语义)
- 当所有者超出作用域 (scope) 时，该值将被删除

```rust
fn main() {
    // s 不可用
    let s = "s"; // s 可用
    // s 可用
}
// s 作用域结束，s 不可用
```

#### 从 String 类型了解所有权
- String 比那些标量类型更复杂
- 字符串字面值: 程序里手写的那些字符串值，它们是不可变的 (`&str` 类型)
- Rust 还有第二种字符串类型: String
    - 在 heap 上分配，能够存储在编译时未知数量的文本
- String 类型的值是可以修改的，而字符串字面值却不能修改
    - 这是因为它们处理内存的方式不同
    1. 字符串字面值，在编译时就知道它的内容，其文本内容直接被硬编码到最终的可执行文件里 (速度快，高效是因为其不可变)
    2. String 类型，为了支持可变性，需要在 heap 上分配内存来保存编译时未知的大小内容
        1. 操作系统必须在运行时来请求数据 (通过调用 `String::from` 实现)
        2. 当用完 String 后，需要使用某种方式将内存返回给操作系统
            - 在拥有 GC 的语言中，GC 会跟踪并清理不再使用的内存
            - 没有 GC 就需要我们去识别内存何时不再使用，并调用代码将它返回
                - 如忘了，就会浪费内存
                - 如提前做了，变量就会非法
                - 如果做了两次，也是 bug，必须一次分配对应一次释放
- Rust 采用了不同的内存管理方式: 对于某个值来说，当拥有它的变量走出作用域范围时，内存会立即自动的交还给操作系统
    - 离开作用域时，Rust 会调用该变量的 drop 方法，释放内存

```rust
fn main() {
    let mut s = String::from("s");
    s.push_str("ss");
    println!("{}", s);
}
// s 离开作用域，调用 drop 函数释放内存，之后将不可用
```

#### 变量和数据交互的方式: Move
- 多个变量可以与同一个数据使用一种独特的方式来交互

```rust
// 整数是已知固定大小的简单的值，这两个 5 被压到 stack 中
let x = 5;
let y = x; // 复制了一个 5，压入栈
```
```rust
// String 类型由三部分组成:
// 1. 指向存放字符串内容的内存的指针
// 2. 长度 len (存放字符串内容所需的字节数)
// 3. 容量 capacity (String 从操作系统总共获得内存的总字节数)
// 上面这些内容放在 stack 中
// 存放字符串内容的部分在 heap 上
let s1 = String::from("s");
let s2 = s1;
// 其他语言的做法
// 当把 s1 赋值给 s2 时，String 在 stack 中存储的东西就被复制了一份给 s2 了
// 但是 heap 的内容不复制，所以 s1 s2 的指针所指向的 heap 内存是一样的
// 当变量离开作用域时，Rust 会自动调用 drop 函数，并将变量使用的 heap 内存释放
// **当 s1 s2 离开作用域时，它们都会尝试释放相同的内存**
//  - 这会引起二次释放的 bug (double free)
```
- 所以为了避免以上其他语言会发生的问题，保证内存安全:
    - Rust 没有尝试复制被分配的内存
    - s1 赋值给 s2 时，Rust 让 s1 失效
        - 当 s1 离开作用域时，Rust 不需要释放任何东西了 (不会发生其他语言的那种二次释放的 bug 了)

```rust
let s1 = String::from("s");
let s2 = s1; // s1 失效了，move 给了 s2

println!("{}", s1); // s1 不可用, 报错
```
- 一般将复制指针、长度、容量视为浅拷贝，但由于 Rust 让 s1 失效了，所以使用 Move 来称呼这个行为

#### 变量和数据交互的方式: Clone
- 如果真的想对 heap 上的数据进行深拷贝，而不仅仅是拷贝 stack 上的数据，可以使用 **clone** 方法

```rust
let s1 = String::from("s");
let s2 = s1.clone();

println!("{} {}", s1, s2); // ok
```

#### 变量和数据交互的方式: Copy
```rust
// 已知大小的数据会存放在 stack 中，它们这样的操作是 copy 行为，会将 stack 上的数据 copy 一份
let x = 5;
let y = x; // 复制了一个 5，压入栈

println!("{} {}", x, y); // ok
```
- Copy trait: 可以用于像整数这样完全存放在 stack 上面的类型
- 如果一个类型实现了 Copy trait，那么旧的变量在赋值后仍然可用
- 如果一个类型或者该类型的一部分实现了 Drop trait，那么 Rust 不允许让它再去实现 Copy trait 了

##### 一些拥有 Copy trait 的类型
- 任何简单标量的组合类型都是可 Copy 的
- 任何需要分配内存或者某种资源的都不是可 Copy 的
- 一些拥有 Copy trait 的类型
    - 所有的整数类型，例如 u32
    - 所有的浮点类型，例如 f64
    - bool
    - char
    - Tuple，如果其拥有的所有字段都是可 Copy 的
        - `(i32, i32)` 是 Copy
        - `(i32, String)` 不是 Copy

#### 所有权与函数
- 在语义上，将值传递给函数和把值赋值给变量是类似的
    - 将值传递给函数将发生移动或复制

```rust
fn main() {
    let s = String::from("hello world");
    take_ownership(s); // s move 到了函数中
    // s 不可用

    let x = 5;
    make_copy(x); // x copy 到了函数中
    println!("x: {}", x); // x 还是可用
}

fn make_ownership(some_str: String) {
    println!("{}", some_str);
}

fn make_copy(some_num: i32) {
    println!("{}", some_num);
}
```
- 函数在返回值的过程中同样也会发生所有权的转移

```rust
fn main() {
    let s1 = gives_ownership(); // 函数里面的返回值 move 给了 s1
    let s2 = String::from("hello world");
    let s3 = takes_and_gives_back(s2); // s2 move 进了函数，又 move 给了 s3
}

fn gives_ownership() -> String {
    let some_str = String::from("hello world");
    some_str
}

fn takes_and_gives_back(a: String) -> String {
    a
}
```
- 一个变量的所有权总是遵循同样的模式
    - 把一个值赋给其他变量时就会发生移动
    - 当一个包含 heap 数据的变量离开作用域时，它的值就会被 drop 函数清理，除非数据的所有权移动到另一个变量上


## <h2 id="Borrowing">Borrowing</h2>
- `&` 符号表示引用，允许你引用某些值而不取得其所有权
- 引用作为函数参数的行为叫做借用
    - 和变量一样，引用默认是不可变的

```rust
fn main() {
    let s1 = String::from("hello world");
    let len = calc_len(&s1); // 将 s1 的引用传入了函数，没有发生所有权的转移

    println!("{}, {}", s1, len); // 由于 s1 所有权没有转移，所以其在该作用域内还是存活的
}

fn calc_len(s: &String) -> usize {
    // s.push_str("ss") // 报错，引用默认是不可变的
    s.len()
}
```
```rust
let mut s1 = String::from("hello world");
let len = calc_len(&mut s1);

fn calc_len(s: &mut String) -> usize {
    s.push_str("ss"); // 借用到的是可变引用，可以修改
    s.len()
}
```

### 可变引用:
- **限制1: 在特定作用域内，对某一块数据，只能有一个可变的引用**
    - 好处: 编译时就可以防止数据竞争
    - 在一个作用域内，仅允许一个活跃的可变引用。所谓活跃，就是真正被使用来修改数据的可变引用，如果只是定义了，却没有使用或者当作只读引用使用，不算活跃
- 以下三种情况会发生数据竞争 (这些数据竞争的情况，运行时是很难发现的，所以 Rust 在设计上避免了这种情况)
    1. 两个或多个指针同时访问同一个数据
    2. 至少有一个指针用于写入数据
    3. 没有使用任何机制来同步对数据的访问

```rust
// 可以通过创建新的作用域，来允许非同时的创建多个可变引用
fn main() {
    let mut s = String::from("hello world");
    {
        let s1 = &mut s;
    }
    let s2 = &mut s;
}
```
- **限制2: 可变引用（写）和只读引用（读）是互斥的关系，就像并发下数据的读写互斥那样**
    - 只有多个不可变的引用是可以的

```rust
fn main() {
    let mut s = String::from("hello world");
    let r1 = &s;
    let r2 = &s;
    let s1 = &mut s; // 不可以，当前作用域已经有不可变引用了
}
```
```rust
fn main() {
  let mut arr = vec![1, 2, 3];
  let last = arr.last(); // 访问不可变引用
  arr.push(4); // 访问可变引用
  println!("last: {:?}", last); // 访问不可变引用 (报错), 此时之前的可变引用
  // 该例中可变不可变相互纠缠, 相互交叉, 破坏了互斥原则, 所以报错
}

// 修改为下
// 不可变和可变引用的界限分明了, 编译通过
fn main() {
  let mut arr = vec![1, 2, 3];
  let last = arr.last(); // 访问不可变引用
  println!("last: {:?}", last); // 访问不可变引用, print "last: Some(3)"
  // 不可变引用访问结束, 此时再访问可变引用, 就不冲突了
  arr.push(4); // 访问可变引用
}
```
```rust
fn main() {
    let mut data = vec![1, 2, 3];
    let data1 = vec![&data[0]]; // 访问了 data 的不可变引用
    println!("data[0]: {:p}", &data[0]);

    for i in 0..100 {
        // 如果继续添加元素，堆上的数据预留的空间不够了，就会重新分配一片足够大的内存，
        // 把之前的值拷过来，然后释放旧的内存。
        // 这样就会让 data1 中保存的 &data[0] 引用失效，导致内存安全问题
        data.push(i); // 访问了 data 的可变引用
    }

    println!("data[0]: {:p}", &data[0]);
    println!("boxed: {:p}", &data1); // 访问了 data 的不可变引用 (和可变引用互斥了, 报错)
}
```

### 悬垂引用 Dangling References
> 一个指针引用了内存中的某个地址，而这块内存可能已经释放并分配给其他人使用了

- Rust 里，编译器可以保证引用永远都不是悬空引用
    - 如果引用了某些数据，编译器将保证在引用离开作用域之前数据不会离开作用域

```rust
fn dangle() -> &String {
    let s = String::from("hello world");
    &s // 报错, 离开作用域 s 被销毁了，s 的引用会指向一个已经被释放的空间，所以编译器不允许返回 &s
}
```

### 引用的规则总结
- 在任何给定的时刻，只能满足以下条件之一
    1. 一个可变的引用
    2. 任意数量不可变的引用
- 引用必须一直有效
- 引用和指针的主要区别
    1. 引用不可为空
    2. 拥有生命周期
    3. 受借用检查器保护不会发生悬垂指针等问题