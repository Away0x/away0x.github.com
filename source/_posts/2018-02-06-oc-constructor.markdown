---
layout: post
title: "ObjectiveC - Constructor"
date: 2018-02-06 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

* [new & init](#o1)
* [instancetype & id](#o2)
* [自定义构造方法](#o3)
* [自定义类工厂方法](#o4)
* [类的本质](#o5)

> OC 中的构造方法实际上并不是传统意义上的构造方法

# <h2 id="o1">new & init</h2>
## new
- new 做了三件事情
    1. 开辟存储空间
    2. 初始化所有的属性(实例变量)
    3. 返回对象的地址
- new 相当于是 alloc 和 init 的组合
- **推荐使用 alloc + init 的方式**
    1. 能够统一编码格式能够统一的对代码进行初始化 (使用 new 不方便使用自定义构造方法)

```objectivec
Person *p = [Person new];
// 相当于下面
Person *p = [[Person alloc] init];
/*
alloc 做的事
    1. 开辟存储空间 
    2. 将所有的属性设置为 0 (无论属性是何类型)
    3. 返回当前实例对象的地址
init 做的事
    1. 初始化成员变量，但是默认情况下 init 的实现是什么都没有做
    2. 返回初始化后的实例变量
    
注意: alloc 返回的地址，和 init 返回的地址是同一个地址
*/
Person *p1 = [Person alloc];
Person *p2 = [p1 init];
NSLog(@"%p %p", p1, p2); // 相同
```

## init
- OC 中以 init 开头的方法，称之为构造方法
- 用途: 用于初始化一个对象，让某个对象一创建出来就拥有某些属性和值

```objectivec
@implementation Person

// 重写 init 方法，在 init 方法中初始化实例变量
// 注意: 重写 init 方法必须按苹果规定的格式重写，不这样会引发一些未知的错误
//    1. 先初始化父类，再初始化子类
//    2. 判断父类是否初始化成功，只有父类初始化成功才能继续初始化子类
//    3. 返回当前对象的地址
- (instancetype)init {
    // 1. 初始化父类
    self = [super init]; // 初始化成功返回对应地址，失败返回 nil
    
    // 2. 判断是否初始化成功
    if (self) {
        // 2.1 初始化子类 (如设置实例变量的值)
    }
    
    // 3. 返回地址
    return self;
}

@end

// 使用
Person *p = [[Person alloc] init];
```



# <h2 id="o2">instancetype & id</h2>
- instancetype 和 id 一样都是万能指针
- 如果构造函数返回值是 instancetype，那么将返回值赋值给一个其他的对象会报一个警告；id 类型则不会
- instancetype 在编译时可判断对象的真实类型，id 则不能
- id 可用来定义变量，可以作为返回值，可以作为形参。instancetype 只能作为返回值
- **注意:** 定义构造方法，返回值尽量使用 instancetype，不要使用 id

```objectivec
@implementation Person
- (instancetype)init {
    if (self = [super init]) {}
    return self;
}
@end

// 使用
Person *p1 = [[Person alloc] init];

// 报警告！将 instancetype 赋给其他对象了
NSString *p2 = [[Person alloc] init];
```
```objectivec
@implementation Person
- (id)init {
    if (self = [super init]) {}
    return self;
}
@end

// 使用
// 不会报警告，因为返回值是 id 类型 (新版本 xcode 也会警告了)
NSString *p = [[Person alloc] init];
```



# <h2 id="o3">自定义构造方法</h2>
- 一个类可以有 0 个或者多个自定义构造方法
- 其实就是自定义一个 init 方法
    1. 一定是实例方法
    2. 一定返回 id/instancetype (建议 instancetype)
    3. 方法名一定以 init 开头，之后有参数的 `initWithXXX`，W 一定要大写 (苹果硬性规定)
        - initwithxxx W 小写的话，不认为其是自定义构造方法，其内不能调用 `[super init]`

```objectivec
// Person.h
@interface Person : NSObject

@property int age;

@end

// Person.m
@implementation Person

- (instancetype)init {
    if (self = [super init]) {
        _age = 10;
    }
    return self;
}
// 自定义构造方法
- (instancetype)initWithAge:(int)age {
    if (self = [super init]) {
        _age = age;
    }
    return self;
}

@end

// 使用
Person *p = [[Person alloc] initWithAge:25];
p.age; // 25
```



# <h2 id="o4">自定义类工厂方法</h2>
- 类工厂方法是一种用于分配、初始化实例并返回一个它自己的实例的类方法
- 类工厂方法规范
    1. 一定是类方法
    2. 返回值一般是 instancetype 类型
    3. 方面名以类名开头，首字母小写
    4. 类工厂中创建对象不要使用类名创建，应该使用 self 来创建

```objectivec
+ (instancetype)personWithAge:(int)age {
    // 不能是 [Person alloc]，否则继承时会有问题
    Person *p = [[self alloc] init];
    [p setAge:age];
    return p;
}

// 使用
Person *p = [Person personWithAge:25];
```



# <h2 id="o5">类的本质</h2>
- 类的本质其实也是一个对象 (类对象)
- 程序中第一次使用该类的时候被创建，在整个程序中只有一份
- 此后每次使用都是这个类对象，它在程序运行时一直存在
- 类对象是一种数据结构，存储类的基本信息: 类大小、类名称、类的版本、继承层次、以及消息与函数的映射表等
- 类对象代表类，Class 类型，对象方法属于类对象
- 如果消息的接收者是类名，则类名代表类对象
- 所有类的实例都由类对象生成，类对象会把实例的 isa 的值修改成自己的地址，每个实例的 isa 都指向该实例的类对象

## 获取类对象
```objectivec
// 1. 通过实例对象
Person *p = [[Person alloc] init];
Class c1 = [p class];

// 通过类名获取 (类名其实就是类对象)
Class c2 = [Person class];
```
```objectivec
// 应用场景

// 1. 用于创建实例对象
Person *p1 = [[c1 alloc] init];

// 2. 用于调用类方法
[c1 test];
```

## 类的启动过程
只要程序启动就会将类的代码加载到内存中，放到代码区

```objectivec
@inplementation Person

// load 方法会在当前类被加载到内存时调用，仅会被调用一次
// 如存在继承关系，会先调用父类的 load 方法，再调用子类的 load 方法
+ (void)load {
    NSLog(@"类被加载到内存中了...");
}

// 当前类第一次被使用时会调用 (创建类对象的时候)
// 在整个程序的运行过程中只会被调用一次，无论你使用多少次这个类都只会调用一次
// 用于对某一个类进行一次性的初始化
// 如存在继承关系，会先调用父类的 initialize 方法，再调用子类的 initialize 方法
+ (void)initialize {
    NSLog(@"类第一次被使用啦");
}

@end
```
