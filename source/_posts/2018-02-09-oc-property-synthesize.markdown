---
layout: post
title: "ObjectiveC - Property, Synthesize"
date: 2018-02-09 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

> Xcode4.4之前 @property 用于生成实例变量 getter/setter 的声明，@synthesize 用于生成 getter/setter 的实现

> @property 和 @synthesize 在 Xcode4.4 以前一直都是配合着使用，在 4.4 以后，@property 得到了增强，一行代码编译器就会自动帮我们生成 setter 和  getter 方法的声明和实现，同时在 m 文件中声明一个和属性名一样并且在最前面带有下划线的实例变量 (私有的)

* [Property](#property)
* [Synthesize](#synthesize)

# <h2 id="property">Property</h2>
- @property 是一个编译器指令
- 可以使用 @property 来代替 getter/setter 方法的声明和实现
    - 即只要写 @property 就不用写 getter/setter 方法了
    - 但 @property 生成的 getter/setter 十分简单，想增强，可自己重写
    - 但如**同时重写(只重写其一不会)**了 getter/setter @property 就不会自动生成私有的实例变量

```objectivec
// 使用实例变量 getter/setter
@interface Person : NSObject {
    int _age;
}
// setter
- (void)setAge:(int)age;
// getter
- (int)age;
@end
```
```objectivec
// 使用 property 替代 实例变量 getter/setter
@interface Person : NSObject

// 编译器只要看到 property，就做下面的事情
//    - 编译时生成一个实例变量 _age
//    - 生成 getter/setter 的声明
//    - 生成 getter/setter 的实现
@property int age; // property 名不用写下划线

@end

// 使用
Person *p = [Person new];
[p setAge:11]; // 可使用 setter 了
p.age = 11;
```

## Property 修饰符
- `@property (修饰符1, 修饰符2, ...) 数据类型 变量名;`
- 不写修饰符默认为 `(atomic, assign, readwrite)`

### readwrite, readonly
- readwrite: (默认) 可读可写，生成 getter/setter
- readonly: 只读，只生成 getter
- 也可指定生成的方法的名称

```objectivec
@property (readonly) int age;

@property (getter=isMarried) BOOL married;
// 通常 BOOL 类型的属性的 getter 要以 is 开头
```

### getter, setter
- getter: 可以给生成的 getter 方法起一个名称
- setter: 可以给生成的 setter 方法起一个名称

### atomic, nonatomic
- atomic: 线程安全，为 setter 加锁，默认就是 atomic，需要消耗大量的资源
- nonatomic: 非线程安全，不会为 setter 加锁，适合内存小的移动设备

### retain
自动生成 set 方法内存管理的代码

- 指针的拷贝，使用的是原来的内存空间，对象的索引计数加1
- 此属性只能用于Objective-C对象类型，而不能用于Core Foundation对象。
    - 原因很明显，retain会增加对象的引用计数，而基本数据类型或者 Core Foundation 对象都没有引用计数

```objectivec
- (void)setBook:(Book *)book {
    if (_book != book) {
        [_book release];
        _book = [book retain];
    }
}
```

### assign
不会生成 set 方法内存管理的代码，仅仅只会生成普通的 set 方法

- 直接赋值，索引计数不改变，适用于简单数据类型，例如：NSIngeter、CGFloat、int、char 等
- 修饰对象了类型时，不改变其引用计数
- 所指对象在被释放之后，仍指向那块内存地址，会产生悬垂指针

```objectivec
- (void)setBook:(Book *)book {
    _book = book;
}
```

### weak, strong
- weak: 
    - 不改变被修饰对象的引用计数
    - 所指对象在被释放之后会自动置为 nil
    - `assign 和 weak 的区别`:
        1. `weak / __weak`: 弱指针，不会让引用计数器加一，如果指向的对象被销毁，指针会自动清空
        2. `assign / __unsafe_unretained`: 不会让引用计数器加一，如果指向的对象被销毁，指针不会清空
- strong: 强指针 `__strong`

### copy
对象的拷贝，新申请一块内存空间，并把原始内容复制到那片空间，新对象的索引计数为 1

- 对象进行拷贝，对于不可变对象拷贝为不可变对象会创建新对象
- 此属性只对那些实行了 NSCopying 协议的对象类型有效
- 很多 Objective-C 中的 object 最好使用用 retain，一些特别的 object（例如：string）使用 copy
- 详见内存管理笔记
- 会先判断这个对象是可变的还是不可变的
    - 所以如该对象不可变可使用 strong，避免判断，以提高性能
- **一般就用于 NSString 和 block**

| 源对象类型 | 拷贝方式 | 目标对象类型 | 拷贝类型 |
| - | - | - | - |
| mutable 对象 | copy | 不可变 | 深拷贝 |
| mutable 对象 mutableCopy | 可变 | 深拷贝 |
| immutable 对象 | copy | 不可变 | 浅拷贝 |
| immutable 对象 | mutableCopy | 可变 | 深拷贝 |

```objectivec
@property(copy) NSMutableArray* arr;
// 赋值过来的是 NSMutableArray，copy 之后是 NSArray
// 赋值过来的是 NSArray，copy 之后是 NSArray
// 所以这样定义容易出现一些异常
```

# <h2 id="synthesize">Synthesize</h2>
- @synthesize 是一个编译器指令
- 通常使用 @property 生成了 getter/setter
    - 可用 synthesize 来修改属性的名称

```objectivec
@interface Person : NSObject
@property int age;
// 相当于下面的代码
/*
{
    int _age;
}
- (void)setAge:(int)age;
- (int)age;

// 并且还在 implementation 中生成了对应的实现
- (void)setAge:(int)age {
    _age = age;
}
- (int)age {
    return _age;
}
*/
@end

@implementation Person

// 告诉编译器，需要实现哪个 @property 生成的声明
//   这里需要实现的是 age property
// 告诉 @synthesize，需要将传入的值赋给谁和返回谁的值给调用者
// 如只写 @synthesize age 那么相当于 @synthesize age = age
// 
@synthesize age = _age;
// 相当于下面的代码
/*
- (void)setAge:(int)age {
    _age = age;
}
- (int)age {
    return _age;
}
*/

@synthesize age = $age;
// 相当于下面的代码
/*
- (void)setAge:(int)age {
    $age = age;
}
- (int)age {
    return $age;
}
*/
@end
```

## 使用场景
```objectivec
// 生成了实例变量 _age，并且实现了它的 getter/setter 的声明和实现
@property (nonatomic, assign) int age;

// 当同时重写了 setter/getter 时，系统会报错，原因是找不到 _age 这个变量

// 解决方法一 (在 .h 的文件中声明这个属性)
@interface Person : NSObject {
  int _age;
}
@end;


// 解决方法二 (在 .m 的文件中使用 @synthesize)
@implementation Person

@synthesize age = _age;
// 由于自己重写了 setter/getter 所以得定义下 synthesize
- (int)age {
    return _age;
}
- (void)setAge:(int)age {
    _age = age;
}

@end

/**
@synthesize age = _age 告诉编译器有个叫 _age 的实例变量，它是方法 age 以及 setAge 的实例变量，如果它不存在，就要将它创建出来

通过这个看似像是赋值的一个操作
我们可以在 @synthesize 中定义与变量名不同的 setter 和 getter 的命名
以此来保护变量不会被不恰当的访问
*/
```
