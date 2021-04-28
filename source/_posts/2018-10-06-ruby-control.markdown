---
layout: post
title: "Ruby - Control Flow"
date: 2018-10-06 08:00:00 +0800
comments: true
categories: ruby
---

<!-- more -->

* [if](#if)
* [unless](#unless)
* [case](#case)
* [times](#times)
* [for](#for)
* [while](#while)
* [until](#until)
* [each](#each)
* [loop](#loop)
* [break](#break)
* [next](#next)
* [redo](#redo)

# <h2 id="if">if</h2>
```ruby
# 可以省略 then

if expr then
    code ...
elsif expr2 then
    code ...
else expr2
    code ...
end

# if 表达式 (ruby 的 if 有返回值)
puts "a > b" if a > b
```



# <h2 id="unless">unless</h2>
```ruby
# unless 和 if 相反，条件为假时执行

unless expr
    条件1
else
    条件2
    
# 与上等价
if expr
    条件2
else 
    条件1

# unless 也有返回值，因此也可写成表达式形式
a = 2 unless defined?(b)
```



# <h2 id="case">case</h2>
- case 语句在判断与 when 指定的值是否相等时，实际上是用 `===` 来判断的
    - `===` 作用详见运算符笔记

```ruby
# 可以省略 then

case 比较对象
when 值1 then
    code...
when 值2 then
    code...
else
    code...
end
```
- 由于 case 内部是 `===` 所以不止可比较简单类型

```ruby
case a
when 1
    1
when /hello/
    "hello world"
when Array
    []
else
    'ok'
end
```



# <h2 id="times">times</h2>
```ruby
# do ~ end
7.times do
    puts "lalala"
end

# { ~ }
7.times {
    puts "lalala"
}

# 获取 index
7.times do |i|
    puts "第 #{i} 次的循环"
end
```



# <h2 id="for">for</h2>
```ruby
# do 可省略

for i in 1..5 do
    puts i
end

names = ["A", "B", "C"]
for name in names
    puts name
end
```



# <h2 id="while">while</h2
```ruby
# do 可省略

i = 1
while i < 3 do
    puts i
    i += 1
end
```



# <h2 id="until">until</h2>
- until 是不满足条件才执行循环处理，满足则退出，和 while 相反

```ruby
# do 可省略

sum = 0
i = 1
until sum >= 50
    sum += i
    i += 1
end
```



# <h2 id="each">each</h2>
- 可将对象集合中的对象逐个取出，类似 for...in
- ruby 内部 for 实际是用 each 实现的

```ruby
# do ~ end 可换成 { ~ }

names = ["A", "B", "C"]
names.each do |name|
    puts name
end
```



# <h2 id="loop">loop</h2>
- 没用终止的循环，不断执行

```ruby
loop do
    print "lalala"
end
```



# <h2 id="break">break</h2>
终止程序，跳出循环



# <h2 id="next">next</h2>
跳到下一次循环



# <h2 id="redo">redo</h2>
在相同的条件下重复刚才的处理
