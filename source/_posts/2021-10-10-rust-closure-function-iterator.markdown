---
layout: post
title: "Rust - Closure, Function Pointer, Iterator"
date: 2021-10-10 08:00:00 +0800
comments: true
categories: rust
---

<!-- more -->

* [Closure](#Closure)
* [Function Pointer](#FunctionPointer)
* [Iterator](#Iterator)


## <h2 id="Closure">Closure</h2>
- 闭包: 可以捕获其所在环境的匿名函数
    - 可以保存为变量、可以作为参数
    - 可在一个地方创建闭包，然后在另一个上下文中调用闭包来完成运算
    - 可以从其定义的作用域捕获值
- 闭包不要求像函数那样在参数和返回值上标注类型，函数需要是因为他们可能会暴露给外部使用, 而闭包则存储在变量中，不暴露给外部使用
    - 闭包像变量定义一样可从上下文环境中推断出类型

```rust
// 由于 rust 可从上下文推断类型，所以该例中参数类型可省略
let func = |n1, n2| {
    println!("{}, {}", n1, n2);
};

let func2 = |n: u32| {
    println!("{}", n)
};

let func3 = |n: u32| -> u32 {
    n
};

let func4 = |n| n + 1; // 只有一行，可省略 {}

func(1, 2);
func2(1);
let l = func3(3);
println!("{}", l);
let l = func4(4);
println!("{}", l);
```

### 让 struct 持有闭包    
- 每个闭包实例都有自己唯一的匿名类型，即使两个闭包的签名完全一样

```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    calculation: T, // 存储闭包，用于计算值
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher { calculation, value: None }
    }

    fn value(&mut self, arg: u32) -> u32 {
        // 有值表示闭包已执行过，直接返回值
        match self.value {
            Some(v) => v,
            None => { // 没值，则调用闭包计算值
                let v = (self.calculation)(arg); // 调用闭包
                self.value = Some(v);
                v
            }
        }
    }
}
```

### 闭包会捕获其所在的环境
- 闭包可以访问定义它的作用域内的变量，而普通函数则不能
    - 这会产生额外的开销

```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x; // 可以执行，闭包内部可以获取到执行环境的变量
    // fn equal_to_x(z: i32) -> bool { z == x } // 报错，函数获取不到执行环境的变量

    let y = 4;

    assert!(equal_to_x(y));
}
```

### Fn Trait
- 由标准库提供，所有的闭包都至少实现了以下 trait 之一 (根据闭包捕获环境的方式分类)
    - **Fn:** 取得捕获到的变量的不可变借用
    - **FnMut:** 取得捕获到的变量的可变借用
    - **FnOnce:** 会取得捕获到的变量的所有权，只能被调用一次
- 创建闭包时，通过闭包对环境值的使用，Rust 可推断出具体使用哪个 trait
    - 所有闭包都实现了 FnOnce
    - 没有移动捕获变量的闭包实现了 FnMut
    - 无需可变访问捕获变量的闭包实现了 Fn


### move 关键字
- 在参数列表前使用 move 关键字，可以强制闭包取得它所使用的环境值的所有权
    - 当将闭包传递给新线程以移动数据使其归新线程所有时，该技术很有用

```rust
fn main() {
    let x = vec![1, 2, 3];

    // x 所有权移动到了闭包内
    let equal_to_x = move |z| z == x;
    println!("can't use x here: {:?}", x); // 报错，x 所有权转移了，不可用

    let y = vec![1, 2, 3];
    assert!(equal_to_x(y));
}
```

### 返回闭包
- 闭包表现为 trait，这意味着不能直接返回闭包。对于大部分需要返回 trait 的情况，可以使用实现了期望返回的 trait 的具体类型来替代函数的返回值

```rust
// 报错
// trait 不是一个可返回的具体类型
fn returns_closure() -> Fn(i32) -> i32 {
    |x| x + 1
}

// ok
// 标明返回值是一个实现了 Fn(i32) -> i32 trait 的类型
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```


## <h2 id="FunctionPointer">Function Pointer</h2>
- 可以将函数传递给其他函数
- 函数在传递过程中会被强制转换成 **fn** 类型

```rust
fn add_one(x: i32) -> i32 { x + 1 }

// f: 函数指针
fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5); // 12
}
```

### 函数指针与闭包的不同:
    - `fn` 是一个类型, 不是一个 trait (这和闭包不同)
        - 可以直接指定 `fn` 为参数类型, 不用声明一个 `Fn Trait` 为约束的泛型参数
- 函数指针实现了全部三种闭包 Trait (Fn, FnMut, FnOnce)
    - 所以总时可以把函数指针用作参数传递给一个解释闭包的函数
    - 所以倾向于搭配闭包 trait 的泛型来编写函数: **可以同时接收闭包和普通函数**
- 某些情景下, 只想接收 fn 而不接收闭包
    - 与外部不支持闭包的代码交互: 如 C 函数

```rust
// 使用闭包
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(|i| i.to_string()) // 闭包
    .collect();

// 使用函数指针
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(ToString::to_string) // 函数指针
    .collect();
```
```rust
enum Status {
    Value(u32),
    Stop,
}

// let v = Status::Value(3)
// - 这里的构造器是被实现为返回由参数构造的实例的函数
// - 所以下例可以将  Status::Value 作为函数指针使用

let list_of_statuses: Vec<Status> =
    (0u32..20)
    .map(Status::Value)
    .collect();
```


## <h2 id="Iterator">Iterator</h2>
- Rust 的迭代器是懒惰的，除非调用消费迭代器的方法，否则迭代器本身没有任何效果
- Rust 的迭代器是零成本抽象，使用时不用担心性能问题

```rust
let v1 = vec![1, 2, 3];
for val in v1.iter() {
    println!("{}", val);
}
```

### Iterator Trait
- 所有的迭代器都实现了 Iterator Trait, 其定义于标准库
- Iterator Trait 仅要求实现 next 方法
    - next: 每次返回迭代器中的一项, 返回结果包裹在 Some 里, 迭代结束返回 None

```rust
// 大致定义如下
pub trait Iterator {
    type item;
    fn next(&mut self) -> Option<Self::item>;
}
```
```rust
// 在在迭代器类型上调用 next 方法
#[cfg(test)]
mod tests {
    #[test]
    fn test_iter() {
        let v1 = vec![1, 2, 3];
        let mut v1iter = v1.iter();

        assert_eq!(v1iter.next(), Some(&1));
        assert_eq!(v1iter.next(), Some(&2));
        assert_eq!(v1iter.next(), Some(&3));
    }
}
```
- `.iter()`: 在不可变引用上创建迭代器
- `.iter_mut`: 迭代可变的引用
- `.into_iter()`: 获取所有权并返回拥有所有权的迭代器

#### 消耗迭代器的方法
- 在标准库中, Iterator Trait 有一些默认实现的方法
- 其中有一些方法会调用 next 方法 (这些调用 next 的方法叫做 **消耗型适配器**)
    - 因为调用它们会把迭代器耗尽, 例如 `sum`, `collect` 方法
- `sum`: 对迭代器进行求和
- `collect`: 把结果收集到一个集合类型中

```rust
// sum
let v1 = vec![1, 2, 3];
let v1_iter = v1.iter();
let total: i32 = v1_iter.sum();

assert_eq!(total, 6);
```

#### 产生其他迭代器的方法
- 会把迭代器转换为不同种类的迭代器 (称为 **迭代器适配器**)
- 可通过链式调用使用多个迭代器适配器来执行复杂的操作, 例如 `map` 方法
- `map`、`filter` ...


```rust
let v1 = vec![1, 2, 3];
let v2: Vec<_> = v1.iter()
    .map(|x| x + 1) // 由于迭代器是惰性的，所以在这一行，并不会执行 +1 的操作
    .collect(); // 执行了这个消耗型适配器, 此时才会开始执行迭代器里面的操作，进行消耗

assert_eq!(v2, vec![2, 3, 4]);
```

### 迭代器配合闭包使用
```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|x| x.size == shoe_size).collect()
}

fn main() {
    let shoes = vec![
        Shoe { size: 10, style: String::from("sneaker") },
        Shoe { size: 13, style: String::from("sandal") },
        Shoe { size: 10, style: String::from("boot") },
    ];

    let in_my_size = shoes_in_my_size(shoes, 10);

    assert_eq!(
        in_my_size,
        vec![
            Shoe { size: 10, style: String::from("sneaker") },
            Shoe { size: 10, style: String::from("boot") },
        ]
    );
}
```

### 自定义迭代器
impl Iterator trait 即可得到迭代器的功能

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32; // 迭代器返回 u32 类型数据

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count < 6 { Some(self.count) } else { None }
    }
}

fn main() {
    let mut counter = Counter::new();

    // next
    assert_eq!(counter.next(), Some(1));
    assert_eq!(counter.next(), Some(2));
    assert_eq!(counter.next(), Some(3));
    assert_eq!(counter.next(), Some(4));
    assert_eq!(counter.next(), Some(5));
    assert_eq!(counter.next(), None);

    // 调用迭代器的其他方法
    let sum: u32 = Counter::new() // 1 迭代到 5
        // 整合两个迭代器, 每一项为 tuple，tuple.0 为第一个迭代器的元素, tuple.1 为第二个迭代器的元素
        // 第二个迭代器以第二次迭代为起始 (2 迭代到 5)
        .zip(Counter::new().skip(1))
        .map(|(a, b)| a * b)
        .filter(|x| x % 3 == 0)
        .sum();

    assert_eq!(18, sum);
}
```

### 自己实现迭代器模式
```rust
trait MyIterator<T: Copy> {
    fn each<F: Fn(T) -> T>(&mut self, f: F);
}
// 性能要比 Rust 的 Iterator 差很多, 这个是一次消费完的, 不是 lazy 模式
impl<T: Copy> MyIterator<T> for Vec<T> {
    fn each<F: Fn(T) -> T>(&mut self, f: F) {
        let mut i = 0;
        while i < self.len() {
            self[i] = f(self[i]);
            i += 1;
        }
    }
}

fn main() {
    let mut v = vec![1, 2, 3];
    v.each(|i| i * 3);
    assert_eq!([3, 6, 9], &v[..3]);
}
```

### for 循环和迭代器
for 是一个语法糖, 其可以用于任何实现了 IntoIterator trait 的数据结构 

```rust
let v = vec![1, 2, 3, 4];

for i in v {
    println!("{}", i);
}

// for 循环等价于以下代码
{ // 等价于 for 循环的 scope
    let mut _iterator = v.into_iter();
    loop {
        match _iterator.next() {
            Some(i) => {
                println!("{}", i);
            }
            None => break,
        }
    }
}
```

### IterTools
> https://crates.io/crates/itertools

该工具可用于扩展标准库里面的迭代器, 类似 python 的 itertools

