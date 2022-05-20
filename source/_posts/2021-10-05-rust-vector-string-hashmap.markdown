---
layout: post
title: "Rust - Vector, String, HashMap"
date: 2021-10-05 08:00:00 +0800
comments: true
categories: rust
---

<!-- more -->

* [Vector](#Vector)
* [String](#String)
* [HashMap](#HashMap)

> 这些类型都是存储在 Heap 中的, 动态大小

## <h2 id="Vector">Vector</h2>
- 类型: `Vec<T>`
    - 由标准库提供
    - 可存储多个值
    - 只能存储相同类型的数据
    - 值在内存中连续存放
- 当 Vector 离开作用域后，它就被清理了，它的所有元素也被清理了

```rust
fn main() {
    // 创建
    let v1: Vec<i32> = Vec::new();
    let mut v2 = vec![1, 2, 3]; // Vec<i32>

    // 更新
    v1.push(1);

    // 获取
    v2[0]; // 通过索引获取
    match v2.get(0) {
        Some(one) => println!("{}", one),
        None => println!("none"),
    }

    // 遍历
    for i in &v2 {
        println!("{}", i);
    }

    let mut v3 = vec![1, 2, 3];
    for i in &mut v3 {
        *i + 1; // 修改 v3 里面的每项元素
    }
```
```rust
// 引用与借用对 Vector 的影响

// 作用域内同时发生了可变借用和不可变借用，违反了引用借用规则
//    因为 Vector 在内存中是连续摆放的，所以再往 v 中添加元素的时候，内存中可能没有这么大的连续内存块了，
//    所以 push 可能会导致内存重新分配(原来的内存释放)，这样的话，之前的引用 first 还是指向原来的内存地址，就会出现问题
//    所以通过引用借用规则，就可以避免掉这类 Bug
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    let first = &v[0]; // 发生了一次不可变的借用
    v.push(6); // 发生了一次可变的借用 (该行会报错)
    println!("The first element is {}", first); // 发生了一次不可变的借用
}
```
```rust
// Vector + Enum
// Vector 只能存储相同类型的数据
// 这里使用附加数据的 Enum 创建 Vector，从而使 Vector 可存储多种数据
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

fn main() {
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("hello")),
        SpreadsheetCell::Float(10.12),
    ];
}
```

## <h2 id="String">String</h2>
- **字符串切片 `&str`**
    - Rust 的核心语言层面，只有一个字符串类型: 字符串切片 **str** (或 **`&str`**)
    - 字符串切片: 对存储在其他地方、UTF-8 编码的字符串的引用
        - 字符串字面值: 存储在二进制文件中，也是字符串切片
- **String 类型**
    - 来自标准库而不是核心语言
    - 其可增长、可修改、可获得所有权
    - UTF-8 编码
- Rust 标准库提供了很多字符串类型:
    - `&str`, `String`, `&[u8]`, `[&u8;N]`, `[Vec<u8>]`, `&u8`, `OsStr`, `OsString`, `Path`, `PathBuf`, `CStr`, `CString`, `&' static str`

### 创建 String
```rust
// 创建
let s1 = String::new();
let s2 = "init content".to_string(); // to_string 方法可用于实现了 Display Trait 的类型
let s3 = String::from("init content");
```

### 更新 String
```rust
// 添加
let mut s1 = String::new();

// push_str: 把一个字符串切片附加到 String
s1.push_str("123"); 
let s2 = String::from("456");
s1.push_str(&s4);
// push: 把单个字符附加到 String
s2.push('l');
```
```rust
// +: 拼接字符串
//   实际上是使用了类似 fn add(self, s: &str) -> String {...} 的这个方法
//   s1 + &s2，其实是把 s1 和 s2 的引用相加到一起，所以 s1 的所有权转移到 add 方法里了，下面拼接之后的打印 s1 会报错
let s1 = String::from("hello ");
let s2 = String::from("world");
let s3 = s1 + &s2; // s1 失去了所有权; s2 由于传递的是引用，保留了所有权

println!("{}", s1); // 报错 (s1 的所有权转移，s1 这里不可使用了)
println!("{}", s2); // 可打印
println!("{}", s3); // 可打印

// 问题: s1 + &s2，这里是 String + &String 类型，但是 add 函数签名要求是 &str，为何可运行？
//      这是因为标准库中这个 add 方法使用了泛型，只能把 &str 添加到 String
//      所以编译器在这里产生了解引用强制转换 (deref coercion)，把 &String 转换为了 &str 类型
```
```rust
// format!: 拼接字符串
//          format! 则不会获取任何参数的所有权
let s1 = String::from("aa");
let s2 = String::from("bb");
let s3 = String::from("cc");

let s4 = format!("{}-{}-{}", s1, s2, s3);
println!("{}", s1); // 可打印
println!("{}", s2); // 可打印
println!("{}", s3); // 可打印

// 上面的例子如果用 + 来实现就得 (使用 + 运算符会导致 s1 的所有权转移)
// let s4 = s1 + "-" + &s2 + "-" + &s3;
```

### 访问 String
- 不支持索引语法访问
    - 字符串其实是对 `Vec<u8>` 的包装，通过索引获取到的是字节，通常没有意义，且容易引发 Bug，所以 Rust 直接拒绝了这种行为
    - 索引操作应该是消耗一个常量时间(`O(1)`)，但是 String 的索引操作无法保证这种时间
        - 因为需要遍历所有内容，来确定有多少个合法字符

```rust
// 访问
// Rust 的 String 不支持索引语法访问
let s1 = String::from("abc");
let s = s1[0]; // 报错

// len: 返回 String 所占用的字节数
let len1 = Strnig::from("hola").len();  // 4 (使用 utf8 编码, hola 占用了 4 个字节)
let len2 = String::from("字节数").len(); // 9 (汉字每个字占用了 3 个字节)
// 由于字符串实际上是个 byte Vector 即 Vec<u8> 类型的包装，所以通过索引只能索引到 unicode 标量值，例如: "208"、"151" 这样的 code
// 为了避免得到这类无意义的数据，Rust 直接拒绝了这样通过索引操作字符串的行为
```

#### 字节(Bytes)、标量值(Scalar Values)、字形簇(Grapheme Clusters)
- Rust 有三种看待字符串的方式:
    - 字节
    - 标量值
    - 字形簇 (最接近所谓的 "字母")

```rust
let s = "नमस्ते";
// 字节
for b in s.bytes() {
    println!("{}", b); // 224 164 168 224 164 174 224 ...
}
// 标量值
for b in s.chars() { // chars 方法会将其分开并返回六个 char 类型的值
    println!("{}", b); // 六个奇奇怪怪的字符
}
// 字形簇
// 标准库没有提供从字符串中获取字形簇的功能
```

### 切割 String
- 可以使用 `[]` 和一个范围来创建字符串的切片
    - 必须谨慎使用
    - 如果切割时跨越了字符边界，程序运行时就会 panic

```rust
let s1 = String::from("字符串");
let s2 = &s1[0..3]; // s2 为 s1 的前是那个字节，汉字占 3 个字节，所以这里切出的是一个完整的汉字
println!("{}", s2); // 字

let s3 = &s1[0..4]; // 前 4 个字节不能切出完整的汉字
println!("{}", s3); // 报错 必须按照字符串的边界切割才行)
```

### 遍历 String
- 遍历标量值: `chars()` 方法
- 遍历字节: `bytes()` 方法
- 遍历字形簇: 很复杂，标准库未提供，需要找第三方库


## <h2 id="HashMap">HashMap</h2>
- 类型: `HashMap<K, V>`
- HashMap 不在 Prelude 中，需要自己导入，且不像 Vector 有内置宏来创建
- 数据存储在 heap 上

```rust
use std::collections::HashMap;
// 创建: new
let mut scores: HashMap<String, i32> = HashMap::new();
// 添加数据: insert
scores.insert(String::from("Blue"), 10);
```
```rust
// 使用 collect 方法创建 HashMap
// collect 方法可以把数据整合成很多种集合类型，也包括 hashmap，所以返回值需要显式指明类型
let teams = vec![String::from("Blue"), String::from("Yellow")];
let intial_scores = vec![10, 50];

// iter 返回遍历器，zip 创建 Tuple
// 需要指明 collect 创建类型为 HashMap，里面的 key value，编译器会自动推到出来，所以可以省略
let scores: HashMap<_, _> = teams.iter().zip(intial_scores.iter()).collect();
println!("{}", scores); // { "Blue": 10, "Yellow": 50 }
```
```rust
// 访问 get(key: &K) => Option<&V>

let mut map = HashMap::new();
map.insert(String::from("k"), String::from("v"));

let one = map.get(&String::from("k"));

match one {
    Some(s) => println!("{}", s),
    None => println!("not exist"),
}
```
```rust
// 遍历
for (k, v) in &map {
    println!("{}: {}", k, v);
}
```
```rust
// 更新
map.insert(String::from("k"), String::from("v")); // 如已存在 k，则会覆盖

// 只在 key 不对应任何值的情况下，才插入 v
// entry: 检查是否存在指定的 key
map.entry(String::from("k"))
    .or_insert(String::from("v2")); // 不存在再插入, or_insert 返回的是 value 的可变引用

// 基于现有值来更新
{
    let text = "hello world wonderful world";
    let mut map = HashMap::new();
    
    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0); // 返回 value 的可变引用
        *count += 1; // 修改 value
    }
    println!("{:#?}", map); // {"hello": 1, "world": 2, "wonderful": 1} // world 单词出现 2 次，其他单词出现 1 次
}
```

### HashMap 和所有权
- 对于实现了 Copy trait 的类型(例如 i32)，值会被复制到 HashMap 中
- 对于拥有所有权的值(例如 String), 值会被移动，所有权会转移给 HashMap
- 如果将值的引用插入到 HashMap，值本身不会移动
    - 在 HashMap 有效的期间，被引用的值必须保持有效

```rust
let k = String::from("k");
let v = String::from("v");
let mut map = HashMap::new();
map.insert(k, v); // k, v 所有权转移给了 map

println!("{}, {}", k, v); // 报错，k v 不可使用了
```
