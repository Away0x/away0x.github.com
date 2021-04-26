---
layout: post
title: "ObjectiveC - Description, Category, Extention"
date: 2018-02-08 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

* [Description](#description)
* [Category](#category)
* [Extention](#extention)

# <h2 id="description">Description</h2>
```objectivec
// 这会调用 A 的 description 方法来输出 A 的描述信息
NSLog(@"%@", A);
```
- description 方法默认返回对象的描述信息 (默认实现是返回类名和对象的内存地址)
- description 方法是基类 NSObject 所带的方法，我们可以重写其来输出对象的信息
- 注意:
    1. description 中尽量不要使用 self 来打印

```objectivec
- (void)description {
    // 打印 self 应用 %p
    NSLog(@"%p", self); // 这样才行
    // NSLog(@"%@", self); // 使用 %@ 打印 self 会造成死循环
}
```
```objectivec
@interface Demo : NSObject

@end

@impletation Demo

- (NSString *)description {
    return [NSString stringWithFormat:@"对象信息"];
}

+ (NSString *)description {
    return [NSString stringWithFormat:@"类对象信息 %@", [Demo class]];
}

@end
```




# <h2 id="category">Category</h2>
- Category 是 OC 特有的语法
- Category 也分为声明和实现
- 一个类可有多个 Category
- 作用
    1. **可以在不修改原来类的基础上，为这个类扩充一些方法**
    2. **一个庞大的类可以分模块开发** (把一个类拆分，归类管理)
    3. 一个庞大的类可以由多个人来编写，更有利于团队合作
- 注意事项
    1. Category 是用于给原有的类添加方法的，只能添加方法，不能添加实例变量
    2. Category 中添加 @property，只会生成 setter/getter 的声明，不会生成实现和私有实例变量
    3. 可以在 Category 中访问原有类 **h 文件**中定义的属性
    4. Category 中有和原有类同名的方法，会覆盖原有类的方法
    5. 如多个 Category 中都有和原有类同名的方法，那么调用该方法时由编译器决定执行哪个，会执行最后一个参与编译的分类中的方法
- 方法的调用顺序
    1. 分类 > 本类 > 父类 > ... > NSObject
- Category 中重写的是原来就有的方法，那么使用其的文件中，不用引入 Category 头文件，就可使用
- Category 中重写的是新方法，那么使用其的文件中，需要引入 Category 头文件，方可使用

```objectivec
// 例如扩充 Person 类

// Category 声明
// Person+Test.h
@interface Person (Test)

- (void)run;

@end

// Category 实现
// Person+Test.m
@implementation Person (Test)

- (void)run {
    NSLog(@"run...");
}

@end

// 使用 Person 类的地方，只要导入了 Person+Test.h
// 就可以使用 Person 新增的实例方法 run 了
```



# <h2 id="extention">Extention</h2>
- Class Extention: 类扩展、匿名 Category
- 可以为某个类扩充一些**私有**的实例变量和方法，外部是无法访问的
    - 保证封装性 (iOS UI 编写中常用)
    - 子类无法获取父类的类扩展
- 对比 Category
    1. 写在**m 文件**中
    2. 不仅可扩充方法，还可扩充实例变量和 @property
    3. 定义格式中不用写分类名

```objectivec
// Person.m

// Class Extention
@interface Person ()

@end

// 类的实现
@inpelmentation Person

@end
```
