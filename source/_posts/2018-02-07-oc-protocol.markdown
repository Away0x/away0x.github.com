---
layout: post
title: "ObjectiveC - Protocol"
date: 2018-02-07 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

* [基础](#o1)
* [使用场景](#o2)

# <h2 id="o1">基础</h2>
- Java 中有 interface 接口。而 OC 中 interface 是类的头文件声明，OC 的接口由 protocol 实现
- protocol 可**声明一些必须实现的方法和选择实现的方法**
- protocol 的作用
    1. 用来声明一些方法
- protocol 是由一系列的方法声明组成的
- **类遵守 protocol**
    1. 一个类可以遵守一个或多个 protocol
    2. 任何类只要遵守了 protocol，就相当于拥有了 protocol 的所有方法声明
- **协议和继承的区别**
    1. 继承后默认就有实现，而协议只有声明没有实现
    2. 相同类型的类可以使用继承，但是不同类型的类只能使用协议
    3. 协议可用于存储方法的声明，可以将多个类中共同的方法抽取出来，以后让这些类遵守协议即可
- **协议的注意事项**
    1. 协议只能声明方法，不能声明属性
    2. 父类遵守了某个协议，那么子类也会自动遵守这个协议
    3. OC 中一个类可以遵守一个或多个协议
    4. OC 中的协议可遵守其他协议，只要一个协议遵守了其他协议，那么这个协议中就会自动包含其他协议的声明

## 定义
```objectivec
// 定义 protocol
@protocol 协议名称
// 方法声明列表
@end

// 类遵守 protocol
@interface 类名 : 父类 <协议1, 协议2, ...>
@end
```

## 基协议 NSObject
- NSObject 是一个基类，最根本最基本的类，任何其他类都要继承自它
- 还有个协议也叫 NSObject，它是一个基协议，最根本最基本的协议
- NSObject 协议中声明了很多最基本的方法
    1. description
    2. retain
    3. release
- 建议每个新的协议都要遵守 NSObject 基协议

```objectivec
@protocol DemoProtocol <NSObject>
@end
```

## @required & @optional
- @require: (默认) 这个方法必须实现，不实现会有警告，但不会报错
- @optional: 这个方法不一定要实现

```objectivec
@protocol DemoProtocol <NSObject>
/*
 func0、func1、func2 不实现会有警告
 func3 可以不实现
*/

- (void)func0; // 默认是 required

@required
- (void)func1;
- (void)func2;

@optional
- (void)func3;

@end
```



# <h2 id="o2">使用场景</h2>
## 类型限制
- 通过协议来限制类型
- `数据类型<协议名称> 变量名`

```objectivec
@protocol Animal <NSObject>
- (void)eat;
- (void)run;
@end

// 如果 dog 没有遵守 Animal 协议就会报警告
Dog<Animal> dog = [[Dog alloc]init];
```
```objectivec
@interface Dog : NSObject

// 限制 property 的类型需要遵守 Animal 协议
@property (nonatomic, strong) Puppy<Animal> *puppy1;

// 这个 property 无论是何类都可，但是得遵守 Animal 协议
@property (nonatomic, strong) id<Animal> *son;

@end
```

**注意:** 虽然在对对象进行了类型限定(限定它必须实现某个协议)，但并不意味着这个对象就真正的实现了该方法(不实现只是由警告无报错)。所以在调用对象的协议方法时，应该进行判断

```objectivec
Dog<Animal> dog = [[Dog alloc]init];

// 判断 dog 是否能响应 eat 消息 (即是否有实现这个方法)
if ([dog respondsToSelector:@selector(eat)]) {
    [dog eat];
}
```

## 委托代理
详见类之间通信的笔记
