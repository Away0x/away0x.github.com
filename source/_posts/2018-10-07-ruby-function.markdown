---
layout: post
title: "Ruby - Function"
date: 2018-10-07 08:00:00 +0800
comments: true
categories: ruby
---

<!-- more -->

方法是由对象定义的与该对象相关的操作。在 ruby 中，对象的所有操作都被封装成方法

* [调用](#call)
* [分类](#category)
* [定义](#define)
* [send & respond_to?](#sr)

# <h2 id="call">调用</h2>
- ruby 中，调用方法被称为 "向对象发送消息 (message)"，调用结果就是 "对象接收 (receive) 了消息"
- 调用方法时，`()` 是可以省略的

```ruby
# 1. 简单的方法调用
对象.方法名(参数1, 参数2, ...)

# 2. 带块的方法调用 (do ~ end 可替换为 { ~ })
#    - do ~ end 时，可以省略参数列表的 ()；{ ~ } 时，只在无参时才可省略
对象.方法名(参数1, 参数2, ...) do |变量1, 变量2, ...|
    块内容 
end

# 3. 运算符形式的方法调用
#    - ruby 中有些方法看起来很像运算符，四则运算等二元运算符，负号等一元运算符
#    - 指定数组、散列的元素下标的 [] 等，实际上都是方法
obj + arg1
obj =~ arg1
-obj
!obj
obj[arg1]
obj[arg1] = arg2
```



# <h2 id="category">分类</h2>
- 帮助文档中方法的标记
    1. 实例方法: `Array#each`
    2. 类方法: `Array.new` 或 `Array::new`

```ruby
# 1. 实例方法
"abc".split(",")

# 2. 类方法
Array.new

# 3. 函数式方法 (没有接收者的)
print "hello"
sleep(10)
```



# <h2 id="define">定义</h2>
```ruby
def 方法名(参数1, 参数2, ...)
    code ...
end
```
```ruby
# 1. 参数默认值 (需放参数列表末尾)
def func(a, b=1, c=2)
    # code ...
end
```
```ruby
# 2. 返回值 (可以省略 return，方法体内部最后一个表达式的结果即为返回值)
# 无指定返回值的函数默认返回 nil
def func()
    return 123
end

def func2()
    123
end

def max(a, b)
    if a > b
        a
    else
        b
    end
end

p max(10, 5) # 10
```
```ruby
# 3. 定义带块的方法
def myloop
    while true
        yield  # 定义块
    end
end

num = 1

# 调用方法时通过块传进来的处理会在 yield 定义的地方执行
# yield 可携带参数
myloop do
    puts "num is #{num}"
    break if num > 10
    num *= 2
end
```
```ruby
# 4. 参数个数不确定的方法
def foo(arg, *args)
    [arg, args]
end

p foo(1)        # [1, []]
p foo(1, 2, 3)  # [1, [2, 3]]

def foo2(a, *b, c)
    [a, b, c]
end

p foo2(1, 2, 3, 4, 5)  # [1, [2, 3, 4], 5]
p foo2(1, 2)           # [1, [], 2]
```
```ruby
# 5. 关键字参数 (可指定默认值)
def 方法名(参数1: 参数1的值, 参数2: 参数2的值, ...)
    code ...
end

def area(x: 0, y: 0, z: 0)
    xy = x * y
    yz = y * z
    zx = z * x
    (xy + yz + zx) * 2
end

p area(x: 2, y: 3, z: 4) # 52
p area(z: 4, y: 3, x: 2) # 52 (参数顺序可变)
p area(x: 2, z: 3)       # 12 (省略 y)

# 不需指定默认值时
def volume(x:, y: 2, z: 4)
    x * y * z
end

p volume(x: 2, y: 3) # 24
p volume(y: 3, z: 4) # ArgumentError
volume(foo: 0)       # unknow keyword

# 搜集关键字参数
def meth(x: 0, y: 0, z: 0, **args)
    [x, y, z, args]
end

p meth(z: 4, y: 3, x: 2)       # [2, 3, 4, {}]
p meth(x: 2, z: 3, v: 4, w: 5) # [2, 0, 3, {:v=>4, :2=>5}]

# 关键字参数与普通参数搭配使用
def func(a, b: 1, c: 2)
    code ...
end
func(1, b: 2, c: 3)

# 用散列传递参数
def area(x: 0, y: 0, z: 0)
    code ...
end

args1 = {x: 2, y: 3, z: 4}
p area(args1)
```
```ruby
# 6. 把数组分解成参数
def foo(a, b, c)
    a + b + c
end

p foo(1, 2, 3)    # 6

args1 = [2, 3]
p foo(1, *args1)  # 6

args2 = [1, 2, 3]
p foo(*args2)     # 6
```
```ruby
# 7. 把散列作为参数传递
#    - 散列作为参数传递给方法时，可省略 {}
def foo(arg)
    arg
end

p foo({"a"=>1, "b"=>2}) # {"a"=>1, "b"=>2}
p foo("a"=>1, "b"=>2)   # {"a"=>1, "b"=>2}
p foo(a: 1, b: 2)       # {:a=>1, :b=>2}

#    - 当作为最后一个参数传递时，也可使用这样的写法
def bar(arg1, arg2)
    [arg1, arg2]
end

p bar(100, {"a"=>1, "b"=>2})  # [100, {"a"=>1, "b"=>2}]
p bar(100, "a"=>1, "b"=>2)    # [100, {"a"=>1, "b"=>2}]
p bar(100, a: 1, b: 2)        # [100, {:a=>1, :b=>2}]
```



# <h2 id="sr">send & respond_to?</h2>
ruby 中所有实例都有该方法，可用于检测当前实例有无该方法

```ruby
a = "hello world"

puts a.respond_to?(:length)
```
