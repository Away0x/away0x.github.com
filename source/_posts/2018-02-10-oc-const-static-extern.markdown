---
layout: post
title: "ObjectiveC - const, static, extern"
date: 2018-02-10 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

* [const](#const)
* [static](#static)
* [extern](#extern)

# <h2 id="const">const</h2>
> 苹果推荐使用 const 常量替代宏字符串常量

## const 与宏的区别
1. **编译时刻:** 宏是**预编译**(编译之前处理)，const 是**编译阶段**
2. **编译检查:** 宏不做检查，不会报编译错误，只是替换，const 会编译检查，会报编译错误
3. **宏的好处:** 宏能定义一些函数、方法。const 则不能
4. **宏的坏处:** 使用大量宏，容易造成编译时间久，每次都需要重新替换

```objectivec
// 宏
#define MyAccount @"account"
#define MyUserDefault [NSUserDefaults standardUserDefaults]

// 字符串常量
static NSString * const account = @"account";

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // 使用宏
    [MyUserDefault setValue:@"123", forKey:MyAccount];
    
    // 使用 const 常量
    [[NSUserDefaults standardUserDefaults] setValue:@"123", forKey:account];
}
```

## 使用
- 作用
    1. 修饰右边的基本变量或指针变量
    2. 被 cosnt 修饰的变量是只读的
- 使用场景
    1. 修饰全局只读变量
    2. 修饰方法中只读参数

```objectivec
int a = 3;
a = 5; // 可以修改

int const b = 3; // 也可这样写 const int b = 3;
b = 5; // 不可修改，报错
```
```objectivec
// const 修饰指针变量

int a = 3;
int *p = &a;
NSLog(@"%d", a); // 3
a = 5;
NSLog(@"%d", a); // 5
*p = 8;
NSLog(@"%d", a); // 8

// const 修饰 *p
int a = 3;
int b = 5;
int const *p = &a;
p = &b; // p 可以修改
*p = 8; // *p 被 const 修饰了，不可修改，会报错

// const 修饰 p
int a = 3;
int b = 5;
int * const p = &a;
p = &b; // p 被 const 修饰了，不可修改，会报错
*p = 8; // *p 可被修改
```
```objectivec
int * const p;       // p: 不可修改，*p: 可修改
int const *p;        // p: 可修改，  *p: 不可修改
const int * p;       // p: 可修改，  *p: 不可修改
const int * const p; // p: 不可修改，*p: 不可修改
int const * const p; // p: 不可修改，*p: 不可修改
```

# <h2 id="static">static</h2>
1. 修饰局部变量，被其修饰的局部变量，会延长生命周期，生命长度跟整个应用程序有关
    - 被 static 修饰的局部变量，只会分配一次内存
    - 被 static 修饰的局部变量，程序一运行就会给其分配内存
2. 修饰全局变量，被 static 修饰的全局变量，作用域会修改，只能在当前文件下使用

## static 和 const 联合使用
- cosnt 修饰全局变量，使其只读
- static 修饰全局变量，使其作用域为文件作用域

```objectivec
// 只在当前文件中可访问到的只读全局变量
static NSString * const name = @"wt";

@implementation ViewController
@end
```



# <h2 id="extern">extern</h2>
1. 用于声明(使用)外部全局变量，**注意: 其只能用于声明，不能用于定义**
2. 工作原理: 先会去当前文件下查找有无对应全局变量，如没有，才会去其他文件查找
3. 全局变量在程序运行时就会分配，如 A 文件中定义了全局变量 `int a = 0`，在 B 文件中可用 `extern int a` 来使用它
4. 如该全局变量使用 static 修饰，那么 extern 无法找到它，因为其是文件作用域

## extern 和 const 联合使用
```objectivec
// A 类
NSString * const a = @"a";

@implementation A
@end
```
```objectivec
// 使用全局变量 a
extern NSString *a;

@implementation B

- (void)test {
    NSLog(@"%@", a); // "a" 可读到
    a = @"aa";       // 报错，a 被 const 修饰了，不可修改
}

@end
```

## 注意点
- 注意: 全局变量不能定义在自己的类中
    - 如多个文件声明了同名的全局变量，会编译失败
- 可自己创建一个管理全局变量的类，如 `GlobeConst`
    - 在 m 文件中定义全局变量，在 h 中使用 extern 引用，这样其他文件要使用这些全局变量，只要引用 `GlobeConst.h` 文件即可
    - `GlobeConst` 中的全局变量使用 const 修饰，避免其他文件使用时对其修改了

```objectivec
// GlobeConst.h

extern NSString * const one_name;
```
```objectivec
// GlobeConst.m

NSString * const one_name = @"one name";
```
```objectivec
// 其他文件

#import "GlobeConst.h"

NSLog(@"%@", one_name); // "one name"
```
