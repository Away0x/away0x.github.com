---
layout: post
title: "Ruby - Variable, Operator"
date: 2018-10-05 08:00:00 +0800
comments: true
categories: ruby
---

<!-- more -->

* [Variable](#variable)
* [Operator](#operator)

# <h2 id="variable">Variable</h2>
Ruby 中有六种类型的变量

1. **局部变量** (local variable)
    - 以英文小写字母或 `_` 开头
2. **全局变量** (global variable)
    - 以 `$` 开头
    - 作用域在整个程序中，即使运用在不同文件里
3. **实例变量** (instance variable)
    - 以 `@` 开头
4. **类变量** (class variable)
    - 以 `@@` 开头
5. **伪变量** (pseudo variable)
    - 伪变量是 ruby 预先定义好的代表某特定值的特殊变量，因此即使在程序中给伪变量赋值，它的值也不会改变
    - `nil、true、false、self` 等都是伪变量
    - 它们表面上虽然看着像变量，但实际行为又与变量有差别，因此称为伪变量
6. **预定义变量**

```ruby
a = "hello"
a.object_id # 70235952968400
# 内容变了，内存地址不变
a.replace("hello2")
a.object_id # 70238952968400

# 重新赋值，重新开辟内存
a = "hello"
a.object_id # 70238952968400
```

全局变量

```ruby
# a.rb

$x = 0  # 定义了一个全局变量
x = 0   # 定义了属于自身的局部变量

require_relative "b"

p $x  # 全局变量 $x 在 b.rb 中被修改了
p x   # 0，a.rb 的 x 和 b.rb 的 x 不是同一个，所以不会被修改
```
```ruby
# b.rb

$x = 1  # 修改了全局变量
x = 1   # 定义了属于自身的局部变量
```

## 常量
- 常量以大写英文字母开头
- ruby 的运行版本 (RUBY_VERSION)、运行平台 (RUBY_PLATFORM)、命令行参数数组 (ARGV) 等，都是 ruby 预定义好的常量
- 已赋值的常量重赋值会报警告

```ruby
TEST = 1
TEST = 2  # 报警告
```



# <h2 id="operator">Operator</h2>
## 算数运算符
```ruby
+ - * / % **

# ** 是指数运算
```

## 赋值运算符
```ruby
=
+= -= *= /= %= **=
&&= ||= ^= &= |= <<= >>=

# ruby 无 ++ --
```
```ruby
# 并行赋值
a, b, c = 10, 20, 30

# 交换变量值
a, b = b, c

# 获取数组的值
ary = [1, 2]
a, b = ary
p a # 1
p b # 2

c, = ary
p c # 1
```

## 逻辑运算符
- 表达式执行顺序从左到右
- 如逻辑表达式的真假已经可以确定，则不会再判断剩余的表达式 (短路)
- 最后一个表达式的值为整体逻辑表达式的值

```ruby
|| or
&& and
! not
```
```ruby
# 变量默认值

item = item || 1  # 在 item 为 false 或 nil 时，才赋值 1

item ||= 1  # 效果同上
```
```ruby
# 安全运算符 (ruby2.3.0+ 支持)

item = ary && ary.first  # ary 安全才使用 ary.first

item = ary&.first  # 效果同上 (ary 为 nil 时，表达式返回 nil)
```

## 比较运算符
```ruby
== != > < >= <=
```
```ruby
<=>
=begin
联合比较运算符 <=>

如果第一个操作数等于第二个操作数则返回 0
如果第一个操作数大于第二个操作数则返回 1
如果第一个操作数小于第二个操作数则返回 -1
=end
```
```ruby
# 如果接收器和参数具有相同的类型和相等的值，则返回 true

.eql?

# 1 == 1.0 返回 true，但是 1.eql?(1.0) 返回 false
```
```ruby
# 如果接收器和参数具有相同的对象 id，则返回 true

equal?

=begin
如果 aObj 是 bObj 的副本，那么 aObj == bObj 返回 true
a.equal?bObj 返回 false
但是 a.equal?aObj 返回 true
=end
```
```ruby
===

# 1. 用于测试 case 语句的 when 子句内的相等
# 2. **左侧**是数值或字符串时，=== 与 == 的意义是一样的
# 3. === 还可和 =~ 一样来判断正则是否匹配
#    - p (/zz/ === "xyzzy")  # true
# 4. 或者判断右边的对象是否属于左边的类
#    - p ((1..3) === 2)  # true
```

## 条件运算符
```ruby
max = (a > b) ? a : b
```

## 范围运算符
- 用于表示数值范围，返回一个 Range 对象
- `X..Y` (范围是 X 到 Y)
- `X...Y` (范围是 X 到 Y 的前一个元素)
- 对 Range 对象使用 to_a 方法，会返回范围中从开始到结束的值

```ruby
# 生成 1~10 的范围对象
Range.new(1, 10)

# 简化定义
1..10
```
```ruby
p (5..10).to_a  # [5, 6, 7, 8, 9, 10]
p (5...10).to_a # [5, 6, 7, 8, 9]

p ("a".."f").to_a  # ["a", "b", "c", "d", "e", "f"]
p ("a"..."f").to_a  # ["a", "b", "c", "d", "e"]
```
```ruby
# 在 Range 对象内部，可以调用 succ 方法根据
```

## 位运算
```ruby
& | ^ ~ << >>
```

## defined? 运算符
- `defined?` 是一个特殊的运算符，以方法调用的形式来判断传递的表达式是否已定义
- 它返回表达式的描述字符串，如果表达式未定义则返回 nil

## 点运算符与双冒号运算符
- 可通过在方法名称前加上类或模块名称和 `.` 来调用类或模块中的方法
- 可以使用类或模块名称和两个冒号 `::` 来引用类或模块中的常量
    - 如果 `::` 前的表达式为类或模块名称，则返回该类或模块内对应的常量值
    - 如果 `::` 前未没有前缀表达式，则返回主 Object 类中对应的常量值

## 运算符重载
不能重载的运算符: `:: && || .. ... ?: not = and or`

### 重载二元运算符

```ruby
class Point                     
    attr_accessor :x, :y
    
    def initialize(x=0, y=0)
        @x, @y = x, y
    end 
    
    def inspect  # 用 p 方法显示 (x, y)
        "(#{x}, #{y})"
    end 
    
    def +(other)  # x, y 分别进行加法运算
        self.class.new(x + other.x, y + other.y)
    end 
    
    def -(other)  # x, y 分别进行减法运算
        self.class.new(x - other.x, y - other.y)
    end 
end

point0 = Point.new(3, 6)
point1 = Point.new(1, 8)

p point0          # (3, 6)
p point1          # (1, 8)
p point0 + point1 # (4, 14)
p point0 - point1 # (2, -2)
```

### 重载一元运算符
- 一元运算符: `+ - ~ !`
- 对应的方法名为: `+@ -@ ~@ !@`
- 一元运算符都是没有参数的

```ruby
class Point                     
    attr_accessor :x, :y
    def initialize(x=0, y=0)
        @x, @y = x, y
    end 
    def inspect  # 用 p 方法显示 (x, y)
        "(#{x}, #{y})"
    end 
    
    def +@  # 返回自己的副本
        dup
    end
    
    def -@  # 颠倒 x, y 的正负
        self.class.new(-x, -y)
    end
    
    def ~@  # 使坐标翻转 90 度
        self.class.new(-y, x)
    end
end

point = Point.new(3, 6)

p +point  # (3, 6)
p -point  # (-3, -6)
p ~point  # (-6, 3)
```

### 重载下标方法
- 数组散列中 `obj[i]` 和 `obj[i] = x` 这样的方法，称为下标方法
- 对应的方法名为 `[]` 和 `[]=`

```ruby
class Point                     
    attr_accessor :x, :y
    def initialize(x=0, y=0)
        @x, @y = x, y
    end 
    def inspect  # 用 p 方法显示 (x, y)
        "(#{x}, #{y})"
    end 
    
    def [](index)
        case index
        when 0
            x
        when 1
            y
        else
            raise ArgumentError, "out of range #{index}"
        end
    end
    
    def []=(index, val)
        case index
        when 0
            self.x = val
        wthen 1
            self.y = val
        else
            raise ArgumentError, "out of range #{index}"
        end
    end
end

point = Point.new(3, 6)

p point[0]      # 3
p point[1] = 2  # 2
p point[1]      # 2
p point[2]      # 错误 ArgumentError
```
