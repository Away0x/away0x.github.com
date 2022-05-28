---
layout: post
title: "ObjectiveC - Singleton"
date: 2018-02-16 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

单例可以保证在程序运行过程，一个类只有一个实例，而且该实例易于供外界访问从而方便地控制了实例个数，并节约系统资源

使用场合: 在整个应用程序中，共享一份资源 (这份资源只需要创建初始化一次)

* [ARC环境下实现单例模式](#arc)
* [MRC环境下实现单例模式](#mrc)
* [ARC与MRC通用的单例](#am)

# <h2 id="arc">ARC 环境下实现单例模式</h2>
```objectivec
// 懒加载的单例的写法

@implementation Demo

// 提供全局变量
static Demo *_instance;

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    if (_instance == nil) {
        _instance = [super allocWithZone:zone];
    }
    return _instance;
}
@end
```
```objectivec
// 使用

Demo *d1 = [[Demo alloc]init];
Demo *d2 = [[Demo alloc]init];
Demo *d3 = [Demo new];

NSLog(@"d1:%p d2:%p d3:%p", d1, d2, d3); // 可见地址都一样
```

使用懒加载的写法，如外界在很多子线程中 alloc，那么会有安全隐患问题

## 解决多线程下的隐患
```objectivec
// 方法一: 加互斥锁，解决多线程访问安全问题

@implementation Demo

static Demo *_instance;

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    @synchronized(self) {
       if (_instance == nil) {
            _instance = [super allocWithZone:zone];
        }
    }
    return _instance; 
}
@end
```
```objectivec
// 方法二: GCD 一次性代码，保证整个程序运行中，只执行一次，且线程安全

@implementation Demo

static Demo *_instance;

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [super allocWithZone:zone];
    });
    
    return _instance; 
}
@end
```

## 优化
- 为了外部方便调用，可像外部提供一个类方法，便于使用单例
    - 方法名: `share + 类名` | `default + 类名` | `类名`

```objectivec
@interface Demo : NSObject

+ (instancetype)shareDemo;

@end

@implementation Demo

static Demo *_instance;

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [super allocWithZone:zone];
    });
    
    return _instance; 
}

// 提供类方法，方便外界访问
+ (instancetype)shareDemo {
    // 调用 alloc 时，会调用 allocWithZone
    return [[self aloc]init];
}

// 严谨: 由于下面这两个方法也可获取到对象
// 所以也需重写一下，让他们得到的也是单例对象
- (id)copyWithZone:(NSZone *)zone {
    return _instance;
}
- (id)mutableCopyWithZone:(NSZone *)zone {
    return _instance;
}

@end
```
```objectivec
// 使用

Demo *d1 = [Demo shareDemo];
Demo *d2 = [Demo new];
Demo *d3 = [Demo shareDemo];
Demo *d4 = [d1 copy];
Demo *d5 = [d1 mutableCopy];

NSLog(@"d1:%p d2:%p d3:%p d4:%p d5:%p", d1, d2, d3, d4, d5); // 可见地址都一样
```



# <h2 id="mrc">MRC 环境下实现单例模式</h2>
```objectivec
@implementation Demo

// 实现同 ARC 单例

#pragma mark - MRC中需要覆盖的方法
// 不需要计数器+1
- (id)retain {
    return self;
}

// 不需要，堆区的对象才需要
- (id)autorelease {
    return self;
}

// 不需要
- (oneway void)release {}

// 不需要计数器个数， 直接返回最大无符号整数
- (NSUInteger)retainCount {
    return UINT_MAX;  // 参照常量区字符串的 retainCount
}
@end
```



# <h2 id="am">ARC 与 MRC 通用的单例</h2>
```objectivec
@interface Demo : NSObject

+ (instancetype)shareDemo;

@end

@implementation Demo

static Demo *_instance;

+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [super allocWithZone:zone];
    });
    
    return _instance; 
}

// 提供类方法，方便外界访问
+ (instancetype)shareDemo {
    // 调用 alloc 时，会调用 allocWithZone
    return [[self aloc]init];
}

- (id)copyWithZone:(NSZone *)zone {
    return _instance;
}
- (id)mutableCopyWithZone:(NSZone *)zone {
    return _instance;
}

// 通过宏定义判断是否为 MRC 环境
#pragma mark - MRC 中需要覆盖的方法, ARC与MRC的整合  
#if !__has_feature(objc_arc)

- (id)retain {
    return self;
}

- (id)autorelease {
    return self;
}

- (oneway void)release {}

- (NSUInteger)retainCount {
    return UINT_MAX;
}  

#endif

@end
```
