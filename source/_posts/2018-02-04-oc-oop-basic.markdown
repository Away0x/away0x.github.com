---
layout: post
title: "ObjectiveC - 面向对象基础"
date: 2018-02-03 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

* [OC 多文件开发](#o1)
* [isa 指针](#o2)
* [实例变量(成员变量)](#o3)
* [方法](#o4)
* [私有变量和私有方法](#o5)
* [self](#o6)
* [NSObject](#o7)
* [例子](#o8)

* OC 中定义一个类分为声明和实现
    * 类可不声明只实现 (但不建议这样写)
* 类名首字母应大写
* OC 的类本质上就是一个结构体

```objectivec
// 参考 C 语言的结构体的使用过程
struct Person {
    int age;
    char *name;
};
struct Person sp;
struct Person *sip = &sp; // 结构体指针

(*sip).age = 25;
(*sip).name = "wutong";
// 或者 (不能用 . 语法直接访问实例变量)
sip->age = 26;
sip->name = "wt";
```

* 所有的对象没有实例化之前都是 nil
* OC 不支持方法重载但支持重写


# <h2 id="o1">OC 多文件开发</h2>
* 多文件开发中，文件要使用谁，就导入谁的 h 文件即可
    * 不能导入 m 文件，否则会报错
* 通常把不同的类放到不同的文件中，每个类的声明和实现分开
    * 声明写在 h 文件中
    * 实现写在 m 文件中
    * 类名是什么，文件名就是什么

## @interface, @implementation
* @interface 写在 h 文件中，用于声明
* @implementation 写在 m 文件中，用于实现具体逻辑



# <h2 id="o2">isa 指针</h2>
* 每个对象都包含一个 isa 指针，这个指针指向当前对象所属的类
* `[p eat]` 表示给 p 所指向的对象发送一条 eat 消息，调用对象的 eat 方法
    * 此时对象会顺着内部 isa 指针找到存储于类中的方法执行
* isa 是对象中的隐藏指针，指向这个对象的类
* 通过 isa 我们可在运行时知道当前对象是属于哪个类的



# <h2 id="o3">实例变量(成员变量)</h2>
* 其写在 @interface 的大括号中
* 默认权限是受保护的，想外部访问需加上 @public
* 编写 OC 实例变量时，建议**私有实例变量**命名前加上 `_`
* 实例变量不能离开类，不能在定义的同时初始化
* 实例变量只能通过对象来访问
* 实例变量不要以 new 开头，否则有可能导致未知错误

## 实例变量修饰符
实例变量修饰符作用域: 从出现的位置，一直到下一个修饰符出现为止

1. @public
    1. 可以在其他类中访问
    2. 可以在本类中访问
    3. 可在子类中访问父类 public 实例变量
2. @private
    1. 不可在其他类中访问
    2. 可以在本类中访问
    3. 不可在子类中访问父类 private 实例变量
3. @protected **(没添加修饰符的都默认被该修饰符修饰)**
    1. 不可在其他类中访问
    2. 可以在本类中访问
    3. 可在子类中访问父类 protected 实例变量
4. @package
    1. 介于 public 和 private 之间
    2. 如在其他包中访问那么就是 private
    3. 如在当前包中访问那么就是 public
    4. 可在子类中访问父类 package 实例变量

```objectivec
@interface Iphone : NSObject {
    int _year; // protected 不公开，外部想访问需提供 setter/getter
@public        // public 公开，外部可直接访问
    float f;
}
@end
```

## 全局变量、局部变量和实例变量的区别
### 全局变量
* 全局变量依托于文件
* 全局变量可先定义再初始化，或定义同时初始化
* 存储于**静态区**
    * 程序一启动就会分配存储空间，直到程序结束才会释放

### 局部变量
* 局部变量依托于函数或代码块
* 局部变量可先定义再初始化，或定义同时初始化
* 存储于**栈**
    * 栈中的数据，系统会自动给我们释放

### 实例变量
* 实例变量依托于类
* 实例变量不能在定义的同时初始化
* 存储于**堆** (当前对象对应的堆的存储空间中)
    * 存储在堆中的数据，不会被自动释放，只能程序员手动释放



# <h2 id="o4">方法</h2>
* C 语言函数，声明在 h 文件中，实现在 c 文件中
* OC 方法，声明在 @interface 中，实现在 @implementation 中
* OC 方法声明写在 @interface 的大括号下面，而不能写在其中
* OC 方法支持重载
* OC 中的方法，如没有形参不需要写 ()，这是因为 OC 方法中的 () 有其他用处，是用于扩住数据类型的
* 有参方法的冒号和外部参数名也是方法名的一部分
* **方法可以没有声明只有实现**
* **方法如声明了没实现，编译不会报错，运行时会报错**
    * 错误: `unrecognized selector send to class/instance` (发送了一个不能识别的消息)
* 方法不要以 new 开头，否则有可能导致未知错误
* OC 方法分类方法和实例方法

## 实例方法 (减号方法)
* 只能通过实例调用
* 方法中可以直接使用实例变量
8 方法中可调用其他实例方法(通过 self 调用)以及类方法(通过类调用)

## 类方法 (加号方法)
* 只能通过类名调用
* 方法中不能直接使用实例变量
* 不用每次使用方法都要创建对象开辟存储空间
* 调用类方法的效率会比调用实例方法高
* 方法中可调用其他类方法或者实例方法
    * 类方法中调用实例方法，需要实例化类，通过实例调用
    * 类方法中调用其他类方法
        1. 通过类调用
        2. 通过 self 调用
* 一般用于定义工具方法



# <h2 id="o5">私有变量和私有方法</h2>
## 私有变量
* 实例变量即可在 interface 中定义也可在 implementation 中定义
* implementation 中定义的实例变量在其他类中无法访问 (即使其是用 public 修饰的)
* implementation 中定义的实例变量只能在本类中访问
* implementation 中定义的实例变量不能和 interface 中定义的实例变量同名
* 因为在其他文件中通常都只是包含头文件而不会包含实现文件，所以在 m 文件中声明的实例变量是 private 的，这种情况下使用 public 也是徒劳的

```objectivec
// Demo.m
@implementation Demo
{
    int _age; // 外界访问不到，即使是 public
}
@end
```

## 私有方法
* 私有方法: 只有实现没有声明的方法
* 原则上: 私有方法只能在本类中才能调用
    * 注意: OC 中没有真正的私有方法
    * 因为 OC 的方法调用是通过消息机制，所以可通过 selector 访问到私有方法

```objectivec
// p 的 test 是个私有方法，没在 interface 中声明
[p performSelector:@selector(test)]; // 但仍可通过这种方式调用到
```



# <h2 id="o6">self</h2>
1. self 在类方法中，那么 self 就代表那个类
2. self 在实例方法中，那么 self 就代表调用当前实例方法的那个实例
3. 可通过 self 调用实例变量 `self->_age`
3. **注意点:**
    1. self 会自动区分类方法和实例方法，如果在类方法中使用 self 调用实例方法，那么会直接报错
    2. 不能再实例方法或类方法中利用 self 调用当前 self 所在的方法，会造成死循环
4. 使用场景:
    1. 可用于在实例方法之间的相互调用
    2. 可用于在类方法之间的相互调用
    3. 可用于区分成员变量和局部变量同名的情况 `self->实例变量`



# <h2 id="o7">NSObject</h2>
提供了创建对象实例的能力(new 方法)，所以需继承它



# <h2 id="o8">例子</h2>
```objectivec
// 类声明
@interface Iphone : NSObject {
    // 实例变量定义:
    //     1. 默认情况下，OC 对象中的实例变量，调用时是不能直接访问的
    //     2. 所以需要公开才可访问 @public 才可通过一个指向结构体的指针访问到
    //     3. @public 下面的实例变量 cpu、cpu2 都会被公开
@public
    float _cpu;
    float _cpu2;
}

// 类方法声明
+ (void)do;
+ (void)do:(int)number;

// 实例方法声明
- (void)about;    // 无参无返回值
- (char *)about2; // 无参有返回值

// 有参函数声明
//    - 参数的数据类型前面必须加上一个 ":" 号
//        - 注意: 当前这个方法名称为 "printInt:" 名称中是有冒号的
//        - 方法名为 "printInt:"
- (int)printInt:(int)number; // 有一参有返回值

//    - 定义多参方法 (无外部参数名)
//        - 方法名为 "printInt::"
- (int)printInt:(int)number :(char *)content;

//    - 定义多参方法 (有外部参数名)
//        - 方法名为 "printMessageWithNumber:andContent:"
//        - 方法名像自然语言一样流畅，建议这样写
- (void)printMessageWithNumber:(int)number andContent:(char *)content;

@end
```
```objectivec
// 类实现
@implementation Iphone

// 类方法的实现
+ (void)do {
    // 类方法中调用实例方法 (不可直接调用)
    Iphone *p = [Iphone new];
    [p about];
    
    // 类方法中调用其他类方法
    // 1. 通过类
    [Iphone do2];
    // 2. 通过 self
    [self do2];
}
+ (void)do:(int)number {
    
}

// 实例方法的实现
- (void)about {
    // 实例方法中可访问成员变量
    // 1. 直接访问
    NSLog(@"%f", _cpu);
    // 2. 通过 self 访问
    NSLog(@"%f", self->_cpu);
    
    // 调用类方法
    [Iphone do];
    
    // 调用其他实例方法
    [self about2];
}
- (char *)about2 {
    return "啦啦啦";
}
- (int)printInt:(int)number {
    return number + 1;
}
- (int)printInt:(int)number :(char *)content {
    NSLog(@"%s", content);
}
- (void)printMessageWithNumber:(int)number andContent:(char *)content {
    NSLog(@"%d %s", number, content);
}

@end
```
```objectivec
// 类调用
int main(int argc, const char * argv[]) {
    // 1. 实例化
    // OC 中要想创建类实例，需给类发送消息 new (这个类需继承 NSObject)
    //    1. new 创建出来的对象存储在堆中，堆中的数据不会自动释放 (栈才会)
    // 实例化会做 3 件事情
    //    1. 为 Iphone 类创建出来的对象分配存储空间 (在堆内存中开辟空间)
    //    2. 初始化 Iphone 类创建出来的对象中的实例变量
    //    3. 返回 Iphone 类创建出来的对象对应的地址
    //        - 返回的地址是类的第 0 个属性的地址
    //        - 并不是自己创建的那个，而是系统自动添加的名为 isa 的属性 (继承于 NSObject)
    //        - 其实系统会在堆内存中开辟一块空间存储这个类，称其为类对象
    //          类对象中存储了这个类的方法列表，isa 就指向这个类对象，
    //          所以可通过其调用类的属性和方法
    // OC 的类本质上就是一个结构体，所以 p 这个指针其实就是指向了一个结构体
    Iphone *p = [Iphone new];
    
    // 2. 访问类的公开实例变量 (不能用 . 语法直接访问实例变量)
    p->_cpu = 3.5; // 通过指针访问
    p->_cpu2 = 3.6;
    NSLog(@"%f %f", p->_cpu, p->_cpu2);
    
    // 3. 调用方法 (通过指针发送消息)
    //    - 消息机制 (调用: 发送消息: [类/实例 方法名])
    //    1. 调用类方法
    [Iphone do];
    //    2. 调用实例方法
    //      先在栈内存中找到 p 这个局部变量，其存储了实例对象的地址
    //      然后通过这个地址在堆内存中找到实例对象的存储空间，并得到里面的 isa 指针
    //      再通过 isa 里存放的地址，找到 Iphone 类对象的存储空间
    //      再在存储空间的方法列表中找是否有名为 about 的方法，如有则执行
    [p about];
    char *content = [p about2];
    int num = [p printInt:123]; // 调用有参数的方法
    int num2 = [p printInt:123 :"lalala"]; // 无外部参数名
    [p printMessageWithNumber:123 andContent:"lalala"]; // 有外部参数名
    
    return 0;
}
```