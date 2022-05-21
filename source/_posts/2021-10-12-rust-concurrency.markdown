---
layout: post
title: "Rust - Concurrency"
date: 2021-10-12 08:00:00 +0800
comments: true
categories: rust
---

- Concurrent 并发: 程序的不同部分之间可以独立的运行
- Parallel 并行: 程序的不同部分同时运行

## 线程
- 通过 `thread::spawn` 创建线程
    - 参数为一个闭包, 闭包里面是在新线程中运行的代码

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
    // 当主线程结束时，新线程也会结束，而不管子线程是否执行完毕
    // 可以使用 join 等待子线程执行完毕 (会堵塞, 直到线程执行完毕)
    handle.join().unwrap();
}
```
- move 闭包常与 spawn 一起使用, 它允许我们在一个线程中使用另一个线程的数据

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];
    // 将 v 的所有权移动到了线程中, 这样线程中就可以安全的使用 v 变量了
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

## 消息传递
- 线程 (或 Actor) 通过彼此发送消息(数据)来进行通信
    - **不用用共享内存来通信, 要用通信来共享内存**
- Rust 中使用 `Channel` 来实现消息传递
    - Channel 包含发送端和接收端, 如果这两端中任意一端被丢弃了, 那么 Channel 就关闭了
- 使用 `mpsc::channel` 来创建 Channel
    - mpsc (multiple producer, single consumer): 多个生产者, 一个消费者
- 发送端:
    - `send` 方法:
        - 参数: 要发送的数据
        - 返回值: `Result<T, E>`, 如果有问题(如接收端已经被丢弃), 就会返回一个 Err
- 接收端:
    - `recv` 方法 (堵塞当前线程运行, 直到 Channel 中有值传送过来)
        - 一但有值收到, 就会返回 `Result<T, E>`
        - 当发送端关闭, 就会收到一个错误
    - `try_recv` 方法 (不会堵塞)
        - 立即返回 `Result<T, E>`, 有数据到达会返回 Ok, 否则返回 Err
        - 通常会使用循环调用来检查该函数的结果

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    // tx: 发送端
    // rx: 接收端
    let (tx, rx) = mpsc::channel();

    // move: 使新的线程拥有了 tx 发送端的所有权了
    thread::spawn(move || {
        let val = String::from("hi");
        // 这里 send 之后, val 的所有权会被转移给接收者
        tx.send(val).unwrap(); // 发送数据
        // val 不可用了
    });

    // 接收数据 (阻塞主线程执行直到从通道中接收一个值)
    // 当通道发送端关闭，recv 会返回一个错误表明不会再有新的值到来了
    // 还有个方法 try_recv, 其不会堵塞则是立即返回 Result, Ok 表示有值, Err 表示值还没到来
    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```
```rust
// 发送多个值
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1)); // 暂停 1s
        }
    });

    // 可把接受者 rx 当作迭代器使用, 每过 1s 会接收到一个值
    for received in rx {
        println!("Got: {}", received);
    }
}
```
```rust
// 通过克隆发送者来创建多个生产者

// ...
let (tx, rx) = mpsc::channel();

let tx1 = tx.clone();
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

// 会接收到 tx1 和 tx 两个发送端发送的数据
for received in rx {
    println!("Got: {}", received);
}
// ...
```

## 共享状态
- Rust 支持通过共享状态来实现并发
    - Channel 类似单所有权: 一旦值的所有权转移至 Channel, 就无法使用它了
    - 共享内存的并发类似多所有权: 多个线程可以同时访问同一块内存
- 使用 `Mutex` (mutual exclusion 互斥锁) 来保证每次只允许一个线程来访问数据
    - 想要访问数据, 线程必须首先获取互斥锁 (lock)
        - lock: 是一种数据结构, 是 mutex 的一部分, 它能跟踪谁对数据拥有独占访问权
    - mutex 通常被描述为: 通过锁定系统来保护它所持有的数据
- **Mutex 的两条规则**
    1. 在使用数据之前, 必须尝试获取锁(lock)
    2. 使用完 mutex 所保护的数据, 必须对数据进行解锁, 以便其他线程可以获取锁
- Mutex 的 API:
    - 通过 `Mutex::new(数据)` 来创建 `Mutex<T>`
        - `Mutex<T>` 是一个智能指针
    - 访问数据前, 通过 lock 方法来获取锁
        - 会堵塞当前线程
        - lock 可能会失败
        - 返回的是 MutexGuard (智能指针, 实现了 Deref 和 Drop)

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    } // mutex 使用了 Drop trait, 所以这里会自动 unlock

    println!("m = {:?}", m);
}
```
- 原子引用计数 `Arc<T>` (A: atomic)
    - 类似 `Rc<T>` (Rc 线程不安全) 并可以安全的用于并发环境的类型, 和 `Rc<T>` 有着相同的 API

```rust
// 多线程共享 mutex
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        // 利用 Arc 实现多线程之间共享所有权
        // 不使用 Arc 的话, 在第一次循环时, counter 的所有权就会被移动到线程中, 第二次循环就会报错 (不能将 counter 锁的所有权移动到多个线程中)
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            // 线程中获取 lock, 并修改值
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    // 等所有线程都执行完毕后再往下执行
    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

### `RefCell<T>`/`Rc<T>` 与 `Mutex<T>`/`Arc<T>` 的相似性
- `Mutex<T>` 提供了内部可变性, 和 Cell 家族一样
    - 使用 `RefCell<T>` 来改变 `Rc<T>` 里面的内容
    - 使用 `Mutex<T>` 来改变 `Arc<T>` 里面的内容
    - 注意: `Mutex<T>` 有死锁风险

## Send & Sync
### Send Trait
- `std::marker::Send`: 允许线程间转移所有权
- 实现 Send Trait 的类型就可以在线程间转移所有权
    - Rust 中几乎所有的类型都实现了 Send
    - `Rc<T>` 没有实现 Send, 它只用于单线程场景
- 任何完全由 Send 类型组成的类型也是 Send
- 除了原始指针之外, 几乎所有的基础类型都是 Send

### Sync Trait
- `std::marker::Sync`: 允许从多线程访问
- 实现 Sync Trait 的类型可以安全的被多个线程引用
    - 即: **如果 `T` 是 Sync, 那么 `&T` 就是 Send**, `&T` 可以被安全的送往另外一个线程
- 任何完全由 Sync 类型组成的类型也是 Sync
- 基础类型都是 Sync
- `Rc<T>`, `RefCell<T>`, `Cell<T>` 家族都不是 Sync 的
- `Mutex<T>` 是 Sync 的
