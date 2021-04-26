---
layout: post
title: "ObjectiveC - 面向对象基础(二)"
date: 2018-02-05 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

* [封装](#o1)
* [继承](#o2)
* [多态](#o3)

# <h2 id="o1">封装</h2>
* 屏蔽内部实现的细节，仅对外提供公有的方法/接口
* 优点: 保证数据的安全性，将变化隔离

## 实例变量的封装
### 未封装的实例变量

```objectivec
@interface Demo : NSObject {
@public
    int _age;
}
@end

@impletation Demo
@end
```
```objectivec
Demo *demo = [Demo new];
demo->_age = 25;
int age = demo->age;
```

### 使用 getter/setter 封装实例变量
* 二者写法格式是固定的
* **setter**
    1. 作用: 设置实例变量的值
    2. 一定是实例方法且无返回值
    3. 一定有参数，参数类型和需设置的实例变量类型相同，且名称是实例变量的名称去掉下划线
    4. 命名一定是 "set + 实例变量" (实例变量名去掉开头下划线，且首字母大写) 
* **getter**
    1. 获取实例变量的值
    2. 一定是实例方法且有返回值
    3. 返回值和获取的实例变量的类型一致
    4. 一定无参数
    5. 命名是获取的实例变量的名称去掉下划线
* 只读属性: 私有实例变量只提供了 getter
* 只写属性: 私有实例变量只提供了 setter
* 可读可写属性: 私有实例变量只提供了 getter、setter

```objectivec
@interface Demo : NSObject {
    int _age;
}
// setter 声明
- (void)setAge:(int)age;

// getter 声明
- (int)age;

@end

@impletation Demo

// setter 实现
- (void)setAge:(int)age {
    _age = age; 
}

// getter 实现
- (int)age {
    return _age;
}

@end
```
```objectivec
Demo *demo = [Demo new];
[demo setAge:25];     // setter
int age = [demo age]; // getter
```

## 点语法
* 如果实例属性提供了 setter、getter 方法，那么访问属性又多了一种访问方式: **点语法**
* 点语法的本质是调用了 setter/getter (C++ 中的点可以直接访问实例变量)
* 点语法是一个编译器特性，实际并不是 OC 的语法
    * 会在程序翻译成二进制时，将点语法自动转换为 setter/getter
* 点语法的注意点:
    * 点语法一般用于给实例变量赋值，如果不是给实例变量赋值一般情况不建议使用
    * 因为其是编译器特性，会将 `a.b` 翻译成 `[a b]` 所以也可以调用一些实例方法，但不要使用
    * `请将点语法当成是便捷版的实例变量 getter/setter 的使用方式，其他情况下请不要使用`

```objectivec
Demo *demo = [Demo new];
demo.age = 25; // 相当于 setter "[demo setAge:25];"
int age = demo.age; // 相当于 getter "int age = [demo age];"
```

# <h2 id="o2">继承</h2>
* OC 是单继承
* A 类继承了 B 类，那么 B 类就拥有了 A 类所有属性和方法

```objectivec
// Demo 继承了 NSObject
@interface Demo : NSObject
@end

@impletation Demo
@end
```

* 如子类中有和父类同名的方法，称之为 **方法重写**
* 继承中的方法调用顺序: 自己有就调用自己的，自己无就调用父类的，以此类推，逐级往上
    * 如一直找到 NSObject 类都没有找到，那么就会报错
* 继承的优点:
    1. 提高了代码的复用性
    2. 让类与类之间产生了关系，这才有了多态


## super
* super 和 self 指向的是相同的消息接收者
* 作用:
    1. 直接调用父类中的某个方法
    2. super 在实例方法中，会调用父类的实例方法
    3. super 在类方法中，会调用父类的类方法
* 使用场景:
    1. 子类重写父类的方法时，想保留父类的一些行为



# <h2 id="o3">多态</h2>
* 多态: 事物的多种表现形态
* OC 不同于传统程序设计语言，它可以在运行时加入新的数据类型和新的程序模块: 动态类型识别、动态绑定、动态加载
* 多态的条件:
    1. 有继承关系
    2. 子类重写父类方法
    3. 父类指针指向子类对象
* 表现: 当父类指针指向不同对象的时候，通过父类指针调用被重写的方法时，会执行指正所指向
* 优点:
    1. 提高了代码的拓展性
* 注意点:
    1. 如父类指针指向子类对象，如需调用**子类特有**的方法，必须先强制类型转换为子类才能调用


```objectivec
// 实现多态
// Animal 是父类，子类有 Cat 和 Dog，分别重写了父类的 eat 方法
// 那么实例化时，可这样:
Animal *animal = nil;

// 实例化 Dog
animal = [Dog new];
[animal eat]; // Dog 的 eat
// 实例化 Cat
animal = [Cat new];
[animal eat]; // Cat 的 eat
```

## 动态类型
* 在编译时编译器只会检查当前类型中有没有需要调用的方法
* 运行时，才会去判断对象的真实类型

```objectivec
Animal *a = [Dog new];

// 1
// 首先由于 a 是 Animal 类型，所以编译时会在去 Animal 里找是否已 eat 方法，无, 会编译失败
// 然后在运行时，系统判断出了 a 的真实类型是 Dog 类型
// 因此这里调用的是 Dog 类的 eat 方法，而不是 Animal 类
[a eat];

// 2
// 如果想调用子类特有的方法，必须强制类型转换为子类才能调用
// 如子类 Dog 特有方法 dogEat，Animal 类没有这个方法
[a dogEat]; // 报错，因为编译阶段发现 Animal 中没有这个方法

Dog *dog = (Dog *)a;
[dog dogEat]; // 正确
```

## id 类型
id 类型: 通用对象指针类型，弱类型，编译时不进行类型检查

## 应用场景
```objectivec
@interface Person: NSObject
+ (void)food:(Animal *)animal;
@end

@implementation Person
+ (void)food:(Animal *)animal {
    // 运行时会动态监测 animal 的真实类型，从而调用对应类型的方法
    // 这样就不用创建针对于不同类型的 food 方法啦
    // 如果真实类型没有 eat 方法，这会找到 Animal 类，调用其的 eat 方法
    // 注意这里 Animal 类得有 eat 方法，因为编译时会做检测
    // 运行时才会确定真实类型
    [animal eat];
}
@end

// 使用
// Cat Dog 继承与 Animal
// Animal 有 eat 方法
// Cat Dog 都各自重写了父类的 eat 方法
Dog *d = [Dog new];
Cat *c = [Cat new];

[Person food:d]; // 调用的是 Dog 的 eat 方法
[Person food:c]; // 调用的是 Cat 的 eat 方法
```

## 注意点
如果存在多态，父类是可以访问子类特有的方法。否则不能直接调用

```objectivec
// 子类特有方法 bark
[dog bark];

Animal *animal = [Dog new];
[(Dog *)animal bark]; // 把父类的指针，强制类型转换
```
