---
layout: post
title: "ObjectiveC - typedef, generic, enum, @class"
date: 2018-02-11 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

* [typedef](#typedef)
* [generic](#generic)
* [enum](#enum)
* [@class](#class)

# <h2 id="typedef">typedef</h2>
```objectivec
// 未使用别名
enum IColor {
    kIColorBlack,
    kIColorWhite
}

@interface Iphone : NSObject {
    enum IColor _color; // 类型名很长
}

@end

@implementation Iphone

@end

// 使用别名
typedef enum {
    kIColorBlack,
    kIColorWhite
} IColor;

@interface Iphone : NSObject {
    IColor _color; // 使用类型别名
}

@end

@implementation Iphone

@end

// 使用
int main(int argc, const char * argv[]) {
    Iphone *p = [Iphone new];
    p->_color = kIColorBlack;
    
    return 0;
}
```



# <h2 id="generic">generic</h2>
> iOS9+

1. 用于限制类型
2. 提高代码规划，减少沟通成本
3. 限制集合里的存储类型，不限制的话都是 id 类型
    - 限制后只能存储对应类型，且由于有了具体类型，取出时可以使用点语法
    - id 类型是不能使用点语法的
4. 泛型仅仅是报警告

```objectivec
// 不用泛型 --------
// 属性
@property (nonatomic, strong) NSMutableArray *arr;

// 使用
[_arr addObject:@"112"]; // 由于定义时没用泛型，所以这里 add 进去的其实是 id 类型
_arr[0].length; // 报错，_arr[0] 里面是 id 类型，id 类型不能使用点语法

// 使用泛型 --------
// 属性
@property (nonatomic, strong) NSMutableArray<NSString *> *arr;

// 使用
[_arr addObject:@"112"]; // add 的是 NSString 类型
_arr[0].length; // 不报错，数组里是 NSString 类型
```
```objectivec
// 参数
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event;
```
```objectivec
// 限定类型
@interface Test<ObjectType> : NSObject
@property (nonatomic, strong) ObjectType language;
@end

// 使用
Java *java = [[Java alloc] init];
iOS *ios = [[iOS alloc] init];

Test<iOS *> *t = [[Test alloc] init];
t.language = ios;  // 泛型限制了，这里只能设置 iOS 类型
t.language = java; // 报警告
```



# <h2 id="enum">enum</h2>
```objectivec
// 第一种写法 (无法设置枚举值类型)
typedef enum {
    DemoTypeTop,
    DemoTypeBottom,
} DemoType;

// 第二种写法 (可设置枚举值类型)
typedef NS_ENUM(NSInteger, DemoType) {
    DemoTypeTop,
    DemoTypeBottom,
};

// 第三种写法 (位移枚举)
// 更强大，使用时可用并运算
typedef NS_OPTIONS(NSInteger, DemoType) {
    DemoTypeTop = 1 << 0,    // 1*2^0 = 1
    DemoTypeBottom = 1 << 1, // 1*2^1 = 2
    DemoTypeLeft = 1 << 2,   // 1*2^2 = 4
    DemoTypeRight = 1 << 3,  // 1*2^3 = 8
};
```

位移枚举可进行位运算

```objectivec
// 按位与 &
//    1&1==1 1&0==0 0&0==0
//    总结: 只有有 0 则为 0
// 按位或 |
//    1|1==1 1|0==1 0|0==0
//    总结: 只有有 1 则为 1

- (void)demo:(DemoType)type {
    if (type & DemoTypeTop) {
        NSLog(@"top --- %zd", type & DemoTypeTop);
    }
    if (type & DemoTypeBottom) {
        NSLog(@"bottom --- %zd", type & DemoTypeBottom);
    }
    if (type & DemoTypeLeft) {
        NSLog(@"left --- %zd", type & DemoTypeLeft);
    }
    if (type & DemoTypeRight) {
        NSLog(@"right --- %zd", type & DemoTypeRight);
    }
}

- (void)test {
    [self demo:DemoTypeTop | DemoTypeRight]; // 打印 top 和 right
    [self demo:0]; // 什么都不会打印
    
    // 小技巧
    // 如是位移枚举，可观察第一个枚举值，如该枚举值不为 0
    // 那么可默认传 0 做参数，如传 0，那么效率最高 (因为什么额外操作都不会做)
}
```

## 枚举中的位运算
- 位移枚举，可以使用并运算 `|`
- 一般情况下，如方法参数是枚举值，那么可用过 `|` 符号，连接多个枚举值

```objectivec
NSDate *now = [NSDate date];
NSCalendar *calendar = [NSCalendar currentCalendar];

// 分别获取 年月日时分秒
// 如方法参数是位移枚举，那么可用过 | 符号，连接多个枚举值
NSCalendarUnit type = NSCalendarUnitYear |
                      NSCalendarUnitMonth |
                      NSCalendarUnitDay |
                      NSCalendarUnitHour |
                      NSCalendarUnitMinute |
                      NSCalendarUnitSecond;
NSDateComponents *cmps = [calendar components:type fromDate:now];
NSLog(@"year = %ld", cmps.year);
NSLog(@"month = %ld", cmps.month);
NSLog(@"day = %ld", cmps.day);
NSLog(@"hour = %ld", cmps.hour);
NSLog(@"minute = %ld", cmps.minute);
NSLog(@"second = %ld", cmps.second);
```



# <h2 id="class">@class</h2>
作用: 可以简单的引用一个类

- 简单使用: `@class Dog;` 仅仅是告诉编译器: Dog 是一个类，并不会包含 Dog 这个类的所有内容

- `@class:` 在 `.h` 文件中使用 `@class` 引用一个类
    - 仅仅只是告诉编译器，`@class`后面的名称是一个类，不会做任何拷贝操作
    - 注意: 编译器不会知道这个类中有哪些属性和方法，所以在 `.m` 中使用这个类时，需要 import 这个类，才能使用
- `#import: `在 `.m` 文件中使用 `#import` 包含这个类的 `.h` 文件
    - 会将整个文件的代码拷贝到 import 所在的位置
    - 只要 import 的文件发生了改变，那么 import 就会重新拷贝一次 (更新操作)
        - 多个类时，编译效率变慢

```objectivec
// Person2.h 中
@class Person1; // 该文件中引入 Person1 类

// Person2.m 中
#import "Person1.h"
```

如果在 A 类中要包含 B类，B 类中又要包含 A 类，则 `.h` 中一定要用 `@class` 避免循环引入

- 总结:
    1. 在 `.h` 中使用 `@class` 引用一个类
    2. 在 `.m` 文件中使用 `#import` 包含这个类的 `.h` 文件

## @class 其他应用场景
对于循环依赖关系来说，比方 A 类引用 B 类，同时 B 类也引用 A 类

```objectivec
// 这种嵌套包含的代码编译会报错
#import "B.h"
@interface A : NSObject {
    B *_b;
}
@end

#import "A.h"
@interface B : NSObject {
    A *_a;
}
@end
```
```objectivec
// 当使用 @class 在两个类相互声明，就不会出现编译报错

@class B
@interface A : NSObject {
    B *_b;
}
@end

@class A
@interface B : NSObject {
    A *_a;
}
@end
```

## @class 和 #import 的区别
- 作用上的区别
    - import 会包含引用类的所有信息(内容)，包括引用类的变量和方法
    - @class 仅仅是告诉编译器有这么个类，具体这个类有什么信息，完全不知
- 效率上的区别
    - 如果有上百个头文件都 `#import` 了同一个文件，或者这些文件依次被 `#import`，那么一旦最开始的头文件有所改动，后面引用到这个文件的所有类都需要重新编译一遍，编译效率非常低
    - 相对来讲，使用 `@class` 就不会出现这个问题了

