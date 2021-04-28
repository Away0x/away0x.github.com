---
layout: post
title: "ObjectiveC - KVO, KVC, Delegate, Notification, Block"
date: 2018-02-17 08:00:00 +0800
comments: true
categories: objectivec
---

<!-- more -->

* [KVO](#kvo)
* [KVC](#kvc)
* [Delegate](#delegate)
* [Notification](#notification)
    * [带参数通知](#notification-params)
    * [一对多通知接收](#notification-om)
    * [通知的多对一关系](#notification-mo)
    * [多线程中使用通知](#notification-t)
    * [系统通知](#notification-d)
    * [键盘通知](#notification-k)
* [Block](#block)
* [通知、KVO、委托代理的区别](#diff)

# <h2 id="kvo">KVO</h2>
通过 KVC 的方式修改被观察者的属性时，主动通知观察者

- KVO: 监听对象属性的变化
- 使用步骤
    1. 注册
    2. 观察
    3. 移除观察
- KVO 的性能问题
    - 如果一个类用 KVO 监听，系统会为其生成一个子类

```objectivec
// ---------------- KVOClass ----------------------
@interface KVOClass : NSObject
@property(nonatomic,strong)Person *p1;
-(void)testKVO;
@end

@implementation KVOClass
-(void)testKVO{
// 1 注册
    self.p1 = [[Person alloc]init];
    [self.p1 setAge:15];
    // Observer: 观察者
    // KeyPath: 要监听的属性
    // options: 方法中要拿到的属性值
    [self.p1 addObserver:self
        forKeyPath:@"age"
        options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld
        context:nil];
    [self.p1 setAge:16];
}
// 3 移除
-(void)dealloc{
    [self.p1 removeObserver:self forKeyPath:@"age"];
}
// 2 观察 KVO 的监听方法
-(void)observeValueForKeyPath:(NSString *)keyPath
    ofObject:(id)object
    change:(NSDictionary<NSString *,id> *)change
    context:(void *)context{
    
    NSLog(@"我的年龄变化了");
    NSLog(@"之前的年龄 %d", [change[NSKeyValueChangeOldKey] intValue]);
    NSLog(@"现在的年龄 %d", [change[NSKeyValueChangeNewKey] intValue]);
}
@end

// ---------------- Person ----------------------
@interface Person : NSObject
@property(nonatomic,assign)int age;
@end

@implementation Person
@end

// ---------------- 执行文件 ----------------------
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        KVOClass *kvo = [[KVOClass alloc]init];
        [kvo testKVO];
    }
    return 0;
}
```



# <h2 id="kvc">KVC</h2>
简单来说， 是存取类属性， 通过字符串来访问对象属性

- KVC (Key Value Coding) 键值编码
- Java/C# 中可通过反射读取对象的属性
- 动态设置
    - setValue (属性值) forKey (属性名)
    - setValue (属性值) forKeyPath (属性路径)
    - setValuesForKeysWithDictionary
- 动态读取
    - valueForKey (属性名)
    - valueForKeyPath (属性名)
    - dictionaryWithValuesForKeys
- KVC 可设置类的私有成员变量

```objectivec
// ---------------- Person ----------------------
@interface Person : NSObject
@property(nonatomic,assign) int age;
@property(nonatomic,copy) NSString* name;
@end

@implementation Person
@end

// ---------------- 执行文件 ----------------------
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *p1 = [[Person alloc]init];
        [p1 setValue:@"henry" forKeyPath:@"name"]; // 可 forKeyPath:@"a.b"
        [p1 setValue:@"18" forKeyPath:@"age"];
        // 批量设置属性
        //   - p1 必须得有 key 对应的属性才行
        [p1 setValuesForKeysWithDictionary:@{
            @"name": @"wt",
            @"age": @26
        }];
        
        Person *p2 = [[Person alloc]init];
        [p2 setValue:@"robin" forKeyPath:@"name"];
        [p2 setValue:@"17" forKeyPath:@"age"];
        
        NSLog(@"p1=%@ p2=%@",p1,[p2 valueForKeyPath:@"name"]);
        // 批量获取值
        NSLog(@"%@", [p1 dictionaryWithValuesForKeys:@[@"name", @"age"]);
    }
    return 0;
}
```

## 作用
1. 字典转模型 ,简化代码量
2. 修改系统的只读变量: 例如自定义tabBar的时候,由于tabBar是只读属性,只能用KVC赋值
3. 可以任意修改一个对象的属性和变量(包括私有变量)
4. 可以通过运算符层次查找对象的属性

```objectivec
keyPathTeacher *t = [Teacher alloc ] init];
t.chiild = [Child alloc ] init];
t.child.book = [Book alloc] init];

NSLog(@"%@",ValueForKeyPath(@"child.book"));
```



# <h2 id="delegate">Delegate</h2>
委托代理也可通过 block 实现

- 适用场合:
    1. 当对象 A 发生了一些行为，想告知对象 B (让对象 B 成为对象 A 的代理对象)
    2. 对象 B 想监听对象 A 的一些行为(让对象 B 成为对象 A 的代理对象)
    3. 当对象 A 无法处理某些行为的时候，想让对象 B 帮忙处理(让对象 B 成为对象 A 的代理对象)
- 协议规范
    1. 一般情况下，当前协议属于谁，就将协议定义在谁的头文件上
- 步骤：
    1. 定义协议 `类名 + Delegate` (需遵守 NSObject 基协议)
    2. 委托 `委托类，需实现上面定义的代理协议`
    3. 代理方法的调用

```objectivec
// AClass
// ---------------- 头文件 ----------------------
// 声明协议
@protocol BuyTicketDelegate;

@interface AClass: NSObject
@property(nonatomic, weak) id<BuyTicketDelegate> delegete; // delegete 用于接受委托
- (void)myBuyTicket;
@end

// 1. 定义协议
@protocol BuyTicketDelegate <NSObject>

@optional // 表下面的方法不一定要实现
- (void) buyTicket;

@end

// ---------------- 实现文件 ----------------------
@implementation AClass

- (void)myBuyTicket {
    // 判断代理中是否有要求的方法
    if ([self.delegete respondsToSelector:@selector(buyTicket)]) {
        [self.delegete buyTicket];
    }
}

@end
```
```objectivec
// BClass
// ---------------- 头文件 ----------------------
@interface BClass : NSObject<BuyTicketDelegate>
- (void)testDelegate;
@end

// ---------------- 实现文件 ----------------------
@implementation BClass

- (void)testDelegate {
    AClass *classa = [[AClass alloc] init];
    classa.delegete = self; // 设置代理，委托 self
    [classa myBuyTicket];
}
- (void) buyTicket {
    NSLog(@"我是 BClass 中的方法");
}

@end
```
```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        BClass *classb = [[BClass alloc] init];
        [classb testDelegate]; // 打印 "我是 BClass 中的方法"
    }
}
```



# <h2 id="notification">Notification</h2>
- 每一个应用程序都有一个通知中心(NSNotificationCenter)实例，专门负责协助不同对象之间的消息通信
- 任何一个对象都可以向通知中心发布通知(NSNotification)，描述自己在做什么。其他感兴趣的对象(Observer)可以申请在某个特定通知发布时或在某个特定的对象发布通知时，收到这个通知
- 通知基本原理
    -  角色：通知中心、发起人、接收者
    -  步骤：
        1. 接受者向通知中心注册
        2. 定义一些通知接收的方法
        3. 发起人向通知中心发通知
- 一个完整的通知需要包含 3 个属性
    1. `- (NSString *)name;` (通知的名称)
    2. `- (id)object;` (通知发布者)
    3. `- (NSDictionary *)userInfo` (一些额外的信息，通知发布者传递给通知接受者的信息内容)
- 监听通知需在发布通知之前

```objectivec
// 1. A 对象发布通知
//    - 定义通知对象
//        - object 为 nil 时，为发布匿名通知 (不指定发布者)
NSNotification *note = [NSNotification notificationWithName:@"通知名称"
    object:A userInfo:@{@"title": @"通知内容xxx"}];
//    - 发送
[[NSNotificationCenter defaultCenter]postNotification:note];
/**
* 或者也可 (简化发送的方法)
[[NSNotificationCenter defaultCenter] 
    postNotificationName:@"通知名称"
    object:A
    userInfo:@{@"title": @"通知内容xxx"}];
*/

// 2. B 对象订阅通知
//     - addObserver: 通知接收者
//     - selector: 通知接收到时执行的函数
//         - doAction: 参数为通知对象 note
//     - name: 通知名称，nil 时为接收任何 object 的通知
//     - object: 通知发布者, nil 时为接收任何 name 名的通知
//         - name 和 object 都为 nil 时，为接收任何通知
[[NSNotificationCenter defaultCenter]addObserver:B
    selector:@selector(doAction:)
    name:@"通知名称"
    object:A];
    
// 3. 通知的移除 (取消订阅)
[[NSNotificationCenter defaultCenter]removeObserver:B
    name:@"通知名称"
    object:A];
// 或
[[NSNotificationCenter defaultCenter]removeObserver:B];
```
```objectivec
// AClass
// ---------------- 头文件 ----------------------

// ---------------- 实现文件 ----------------------
@implementation AClass
- (void) init {
    self = [super init];
    if (self) {
        // 3. 发起人向通知中心发送通知
        [[NSNotificationCenter defaultCenter] 
            postNotificationName:@"TESTNOTIF" object:nil];
    }
    return self;
}
@end
```
```swift
// BClass
// ---------------- 头文件 ----------------------
@interface BClass : NSObject<BuyTicketDelegate>
- (void)testNotification;
@end
// ---------------- 实现文件 ----------------------
@implementation BClass
- (void)testNotification {
    // 1. 接受者向通知中心注册
    // 参数1：谁在接收这个方法，参数2：响应方法，参数3：通知名称，参数4通知发布者
    // self 指当前的这个类
    [[NSNotificationCenter defaultCenter] 
        addObserver:self selector:@selector(testAction) name:@"TESTNOTIF" object:nil];
    
    AClass *classa = [[AClass alloc] init]; // A 类的构造方法中发广播
}
// 2. 定义一些通知接收的方法
- (void)testAction {
    NSLog(@"接收到通知了");
}
// 广播销毁的方法
- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:@"TESTNOTIF" object:nil];
}
@end
```
```swift
// A 类中发送通知，B 类中接收
// main
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        BClass *classb = [[BClass alloc] init];
        [classb testNotification]; // "接收到通知了"
    }
}
```

## <h3 id="notification-params">带参数通知</h3>
```objectivec
// AClass
// ---------------- 头文件 ----------------------

// ---------------- 实现文件 ----------------------
@implementation AClass
- (void) init {
    self = [super init];
    if (self) {
        // 3. 发起人向通知中心发送通知
        // 如通知需要携带数据，可以：
        NSMutableDictionary *dic = [[NSMutableDictionary alloc] init];
        [dic setValue:@"123" forKey:@"keya"];
        
        // 参数1：通知名，参数2：object，参数3：dict(字典)
        [[NSNotificationCenter defaultCenter] 
            postNotificationName:@"TESTNOTIF" object:
            [NSNumber numberWithInt:5] userInfo:dic];
    }
    return self;
}
@end
```
```objectivec
// BClass
// ---------------- 头文件 ----------------------
@interface BClass : NSObject<BuyTicketDelegate>
- (void)testNotification;
@end
// ---------------- 实现文件 ----------------------
@implementation BClass
- (void)testNotification {
    // 1. 接受者向通知中心注册
    // 参数1：谁在接收这个方法，参数2：响应方法，参数3：通知名称，参数4：通知发布者
    // self 指当前的这个类
    [[NSNotificationCenter defaultCenter] 
        addObserver:self selector:@selector(testAction:) name:@"TESTNOTIF" object:nil];
    
    AClass *classa = [[AClass alloc] init]; // A 类的构造方法中发通知
}
// 2. 定义一些通知接收的方法(拿到参数)
- (void)testAction:(NSNotification*)notif {
    NSNumber *num = notif.object;
    NSDictionary *dic = notif.userInfo;
    NSLog(@"%@ %@", num, dic);
}
// 通知销毁的方法
- (void)dealloc {
    [[NSNotificationCenter defaultCenter] removeObserver:self name:@"TESTNOTIF" object:nil];
}
@end
```
```objectivec
// A 类中发送通知，B 类中接收
// main
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        BClass *classb = [[BClass alloc] init];
        [classb testNotification]; // "接收到广播了"
    }
}
```

## <h3 id="notification-om">一对多通知接收(一个发送，多个接收)</h3>
```objectivec
// Person 发出通知，Student 和 Teacher 会接收到

// ---------------- Person ----------------------
@interface Person : NSObject
-(void)testNotification;
@end

@interface Person(){
    Student *s1;
    Teacher *t1;
}
@end
@implementation Person
-(void)testNotification{
    // [Student new];
    // [Teacher new]; // 这种方式定义的对象 (通知还未来得及发送，对象就已经被释放掉了)
    s1 = [[Student alloc]init];
    t1 = [[Teacher alloc]init];
    [[NSNotificationCenter defaultCenter]postNotificationName:@"TESTNOTIF" object:nil];
}
@end

// ---------------- Student ----------------------
@implementation Student
-(id)init{
    self = [super init];
    if (self) {
        [[NSNotificationCenter defaultCenter]
            addObserver:self
            selector:@selector(notifAction)
            name:@"TESTNOTIF"
            object:nil];
    }
    return self;
}
-(void)notifAction{
    NSLog(@"student 接收");
}
-(void)dealloc{
    [[NSNotificationCenter defaultCenter]removeObserver:self name:@"TESTNOTIF" object:nil];
}
@end

// ---------------- Teacher ----------------------
@implementation Teacher
-(id)init{
    self = [super init];
    if (self) {
        [[NSNotificationCenter defaultCenter]
            addObserver:self
            selector:@selector(notifAction)
            name:@"TESTNOTIF"
            object:nil];
    }
    return self;
}
-(void)notifAction{
    NSLog(@"teacher 接收");
}
-(void)dealloc{
    [[NSNotificationCenter defaultCenter]removeObserver:self name:@"TESTNOTIF" object:nil];
}
@end

// ---------------- 使用文件 ----------------------
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *p1 = [[Person alloc]init];
        [p1 testNotification];
    }
    return 0;
}
```

## <h3 id="notification-mo">通知的多对一关系(多个发送，一个接收)</h3>
```objectivec
// Student 和 Teacher 发出通知，Person 会接收到

// ---------------- Person ----------------------
@interface Person : NSObject
//@property
-(void)testNotification;
@end

@interface Person(){
    Student *s1;
    Teacher *t1;
}
@end
@implementation Person
-(void)testNotification{
    
    [[NSNotificationCenter defaultCenter]addObserver:self selector:@selector(notifAction:) name:@"TESTNOTIF" object:nil];
    s1 = [[Student alloc]init];
    t1 = [[Teacher alloc]init];
}
-(void)notifAction:(NSNotification*)notif{
    NSDictionary *dic = notif.userInfo;
    NSString *string = [dic valueForKey:@"info"];
    NSLog(@"%@",string);
}
-(void)dealloc{
    [[NSNotificationCenter defaultCenter]removeObserver:self name:@"TESTNOTIF" object:nil];
}
@end

// ---------------- Student ----------------------
@implementation Student
-(id)init{
    self = [super init];
    if (self) {
        NSMutableDictionary *dic = [[NSMutableDictionary alloc]init];
        [dic setValue:@"我来自Student" forKey:@"info"];
        [[NSNotificationCenter defaultCenter]postNotificationName:@"TESTNOTIF" object:nil userInfo:dic];
    }
    return self;
}
@end

// ---------------- Teacher ----------------------
@implementation Teacher
-(id)init{
    self = [super init];
    if (self) {
        NSMutableDictionary *dic = [[NSMutableDictionary alloc]init];
        [dic setValue:@"我来自Teacher" forKey:@"info"];
        [[NSNotificationCenter defaultCenter]postNotificationName:@"TESTNOTIF" object:nil userInfo:dic];
    }
    return self;
}
@end

// ---------------- 使用文件 ----------------------
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *p1 = [[Person alloc]init];
        [p1 testNotification];
    }
    return 0;
}
```

## <h3 id="notification-t">多线程中使用通知</h3>
```objectivec
// 监听通知
// Name: 通知名字
// object: 谁发出的通知
// queue: 决定 block 在哪个线程执行，nil 表示在发布通知的线程中执行
// usingBlock: 只要监听到通知，就会执行这个 block
id observe = [[NSNotificationCenter defaultCenter]
    addObserverFromName:@"note"
    object:nil
    queue:nil
    usingBlock:^(NSNotification * __NONNULL note) {
        // 只要监听到通知，就会调用
        NSLog(@"%@", [NSTread currentThread]);
    }];
    

// 发出通知
[[NSNotificationCenter defaultCenter]postNotificationName:@"note" object:nil];

// 移除通知
[[NSNotificationCenter defaultCenter]removeObserver:observe];
```

### 通知在多线程中的注意点
- 接收通知的代码在哪个线程执行是由发通知的的线程决定的
    - 可通过 addObserverFromName 的 queue 参数指定

```objectivec
// 异步线程监听通知，主线程发通知

// 监听通知在异步中
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    // 异步任务
    [[NSNotificationCenter defaultCenter]
        addObserver:self
        selector:@selector(reciveNote) // reciveNote 这个方法是在主线程中调用的(因为发通知在主线程)
        name:@"note"
        objeect:nil];
});

// 发出通知
[[NSNotificationCenter defaultCenter]postNotificationName:@"note" object:nil];

// 发现这段代码中的异步任务中监听不到通知
// 这是由于异步任务执行的顺序不确定，有可能监听在发通知之后

// 当发通知的代码改为如点击事件触发，保证一定在监听之后执行，则异步线程中监听通知成功
```
```objectivec
// 当发通知在异步线程中，主线程监听通知

// 由于监听到通知触发的回调执行在的线程由发通知的线程决定
// 因此该回调执行在异步线程中
// 如果改回调中需要修改 UI，则需在主线程中修改

// 执行在异步线程，但更新 UI 的代码执行在主线程总
- (void)reciveNote {
    
    // ....
    
    // 回到主线程更新 UI
    dispatch_sync(dispatch_get_main_queue(), ^{
        // 修改 UI
    });
}
```
```objectivec
// queue 参数指定 block 在指定线程执行
// 无论发通知是在哪个线程中都没关系

id observe = [[NSNotificationCenter defaultCenter]
    addObserverFromName:@"note"
    object:nil
    queue:[NSOperationQueue mainQueue]
    usingBlock:^(NSNotification * __NONNULL note) {
        // 只要监听到通知，就会调用
        NSLog(@"%@", [NSTread currentThread]);
    }];
```

## <h3 id="notification-d">系统通知 UIDevice</h3>
- UIDevice 类提供了一个单例对象，它代表着设备，通过它可以获得一些设备相关的信息，比如:
    - 电池电量值 batteryLevel
    - 设备类型 model (iPod、iPhone ...)
    - 设备系统 systemVersion
- 可通过 `[UIDevice currentDevice` 获取这个单例对象
- UIDevice 对象会不间断的发布一些通知，下面是通知名称
    1. `UIDeviceOrientationDidChangeNotification` 设备旋转
    2. `UIDeviceBatteryStateDidChangeNotification` 电池状态改变
    3. `UIDeviceBatteryLevelDidChangeNotification` 电池电量改变
    4. `UIDeviceProximityStateDidChangeNotification` 近距离传感器 (比如设备贴近使用者的脸部)

```objectivec
// 系统版本
double version [[UIDevice currentDevice].systemVersion.doubleValue;

if (version >= 9.0) {
    // do something   
}
```

## <h3 id="notification-k">键盘通知</h3>
键盘状态改变时，系统会发出一些特定的通知

1. `UIKeyboardWillShowNotification` 键盘即将显示
2. `UIKeyboardDidShowNotification` 键盘显示完毕
3. `UIKeyboardWillHideNotification` 键盘即将隐藏
4. `UIKeyboardDidHideNotification` 键盘隐藏完毕
5. `UIKeyboardWillChangeFrameNotification` 键盘的位置尺寸即将发生改变
6. `UIKeyboardDidChangeFrameNotification` 键盘的位置尺寸改变完毕



# <h2 id="block">Block</h2>
- block: 代码块
- 本质上是函数指针(代码块的内存地址)
- 常用语传值，即把 block 的地址传到要调用 block 的地方

## 实现
```objectivec
// 1. 声明 block (返回值 + Block名 + 传递参数)
typedef int (^MyBlock)(int);

int main(int argc, const char * argv[]) {
    @autoreleasepook {
        int a = 5;
        __block int aa = 5;
        // 2. 实现 block
        MyBlock b1 = ^(int b) {
            NSLog(@"%d", b);
            // block 中使用外部变量，声明时，需加 __block
            return b + 1;
        };
        // 3. 调用 block
        int newb = b1(10); // 11
    }
    return 0;
}
```

## 类之间的通信为什么要用 block
```objectivec
@interface AClass : NSObject
- (void)testBlock;
@end

@implementation AClass
- (void)testBlock {
    // A 类中调 B 类的方法
    BClass *classb = [[BClass alloc] init];
    [classb testB]; 
}
@end
```
```objectivec
@interface BClass : NSObject
- (void)testB;
@end

@implementation BClass
- (void)testB {
    NSLog(@"我是B");
    // 如果 B 类想使用 A 类的方法，不可实例化 A 类(会交叉引用)，得使用 block
    // AClass *classa = [[AClass alloc] init];
}
@end
```
```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepook {
        AClass *classa = [[AClass alloc] init];
    }
    return 0;
}
```

## 类之间通信中使用 block
其实就是 A 传递一个函数给 B，从而实现在 B 中调 A 的方法 

```objectivec
@interface AClass : NSObject
- (void)testBlock;
@end

@implementation AClass
- (void)testBlock {
    // 声明 block
    MyBlock b1 = ^(NSString *str1) {
        NSLog(@"AClass:%@", str1);
    };
    // 调用 B 类
    BClass *classb = [[BClass alloc] init];
    [classb testB:b1 str1:@"AClass"];
    
    // ---------------
    // 快速传递 block(类似 js 的回调)
    BClass *classb = [[BClass alloc] init];
    [classb testB:^(NSString *str1) {
        NSLog(@"AClass:%@", str1);
    } str1:@"AClass"];
}
@end
```
```objectivec
typedef void (^MyBlock)(NSString*); // 声明 block
@interface BClass : NSObject
- (void)testB:(MyBlock)block str1:(NSString)str1;
@end

@implementation BClass
- (void)testB:(MyBlock)block str1:(NSString)str1 {
    NSLog(@"%@", str1);
    block(@"我是从B类来的");
}
@end
```
```objectivec
int main(int argc, const char * argv[]) {
    @autoreleasepook {
        AClass *classa = [[AClass alloc] init];
    }
    return 0;
}
```



# <h2 id="diff">通知、KVO、委托代理的区别</h2>
- 三者都是 iOS 中监听事件的方式
- **通知**
    - 任何对象之间都可以传递消息
    - 使用范围
        1. 1 个对象可以发通知给多个对象
        2. 1 个对象可以接受多个对象发出的通知
    - 要求: 必须得保证通知的名字在发出和监听时是一致的
- **KVO**
    - 仅能监听对象属性的变化 (灵活度不如通知和代理)
- **代理**
    - 1 个对象只能设置一个代理 (假设这个对象只有一个代理属性)
    - 1 个对象能成为多个对象的代理
- 如何选择
    1. 代理比通知规范
    2. 建议使用代理多于通知，能使用代理尽量使用代理
