---
layout: post
title: "Ruby - install, basic"
date: 2018-10-04 08:00:00 +0800
comments: true
categories: ruby
---

<!-- more -->

* [RubyGem](#rubygem)
* [RVM](#rvm)
* [RBENV](#rbenv)
* [命令行选项](#cl)
* [预定义变量常量](#const)
* [基础](#basic)

# <h2 id="rubygem">RubyGem</h2>
RubyGems 是一个统一安装和管理 Ruby 的库、程序的 Ruby 标准工具

在 RubyGems 中， 每个单独的库称为 GEM。通过 RubyGems，我们可以搜索 GEM、显示 GEM 相关的信息、安装 或卸载 GEM、升级旧版本的 GEM，以及查看 GEM 的安装进度一览表，等等

```bash
# 显示 GEM 的安装进度一览表
gem list

# 用于搜索 GEM 文件，没有指定选项时，会搜索远程仓库的 GEM 文件
gem search nokogiri
gem search -l nokogiri # 搜索本地已安装的 GEM

# 安装
gem install nokogiri
# 安装本地的 GEM 文件时，不是指定 GEM 名，而是指定 GEM 文件名
gem install nokogiri-1.6.6.2.GEM

# 把 GEM 更新为最新版本
gem update nokogiri

# 更新 RubyGems 自身
gem update --system

# 查看某个 gem 的源码
bundle show gem_name # 然后根据 path 打开源码
```

选项 | 意义
- | -
build | 根据 gemspec 创建 GEM
cert | 管理、签署 RubyGems 的许可证时使用
check | 检查 GEM
cleanup | 整理已安装的旧版本的 GEM
contents | 显示已安装的 GEM 的内容
dependency | 显示已安装的 GEM 的依赖关系
environment | 显示 RubyGems、Ruby 等相关的环境信息
fetch | 把 GEM 文件下载到本地目录，但不安装
generate_index | 创建 GEM 服务器所需的索引文件
help | 显示 GEM 命令的帮助说明
install | 安装 GEM 至本地仓库
list | 显示 GEM 的一览表
lock | 锁定 GEM 版本，并输出锁定后的 GEM 列表
mirror | 创建 GEM 仓库的镜像
open | 用编辑器编辑已安装的 GEM
outdated | 显示所有需要更新的 GEM 列表
owner | 管理 GEM 所有者的资料
pristine | 从 GEM 缓存中获取已安装的 GEM，并将其恢复为初始状态
push | 向服务器上传 GEM
query | 搜索本地或者远程仓库的 GEM 信息
rdoc | 生成已安装的 GEM 的 RDoc 文件
regenerate_binstubs | 变更用 GEM 安装的命令的 shebang
search | 显示名字包含指定字符串的 GEM
server | 启动 HTTP 服务器，用于管理 GEM 的文档及仓库
sources | 管理搜索 GEM 时所需的 RubyGems 的源以及缓存
specification | 以 yaml 形式显示 GEM 的详细信息
stale | 按最后访问的时间顺序显示 GEM 的一览表
uninstall | 从本地卸载 GEM
unpack | 在本地目录解压已安装的 GEM
update | 更新指定的 GEM(或者全部 GEM)
which | 显示读取 GEM 时引用的类库
wrappers | regenerate_binstubs 的别名
yank | 撤回已上传到服务器的 GEM 文件



# <h2 id="rvm">RVM</h2>
- RVM 是一个命令行工具,可以提供一个便捷的多版本 Ruby 环境的管理和切换
- [安装方法](http://rvm.io/)
- [参考](https://ruby-china.org/wiki/rvm-guide)

```bash
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash -s stable
:'
# 下载命令错误可试试
curl -L https://raw.githubusercontent.com/wayneeseguin/rvm/master/binscripts/rvm-installer | bash -s stable
'
source ~/.bashrc
source ~/.bash_profile

# 修改 RVM 的 Ruby 安装源到 Ruby China 的 Ruby 镜像服务器
echo "ruby_url=https://cache.ruby-china.org/pub/ruby" > ~/.rvm/user/db
```
```bash
# 列出已知的 Ruby 版本
rvm list known

# 安装一个 Ruby 版本
rvm install 2.2.0 --disable-binary

# 切换 Ruby 版本
rvm use 2.2.0

# 如果想设置为默认版本，这样一来以后新打开的控制台默认的 Ruby 就是这个版本
rvm use 2.2.0 --default 

# 查询已经安装的ruby
rvm list

# 卸载一个已安装版本
rvm remove 1.8.7
```

## gemset
gemset 可以理解为是一个独立的虚拟 Gem 环境，每一个 gemset 都是相互独立的

```bash
# 建立 gemset
rvm use 1.8.7
rvm gemset create rails23

# use 可以用来切换语言或者 gemset (前提是他们已经被安装(或者建立)。并可以在 list 命令中看到。)
rvm use 1.8.7
rvm use 1.8.7@rails23

# 列出当前 Ruby 的 gemset
rvm gemset list

# 清空 gemset 中的 Gem
rvm gemset empty 1.8.7@rails23

# 删除一个 gemset
rvm gemset delete rails2-3

# 项目自动加载 gemset
:'
# 项目目录，建立一个 .rvmrc 文件，内容如下
rvm use 1.9.3@rails313

这样，cd 到这个项目的时候，RVM 会帮你加载 Ruby 1.9.3 和 rails313 gemset.
'
```



# <h2 id="rbenv">RBENV</h2>
- 类似 RVM，二者选其一即可 
- [参考](https://ruby-china.org/wiki/rbenv-guide)

```bash
git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
# 用来编译安装 ruby
git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
# 用来管理 gemset, 可选, 因为有 bundler 也没什么必要
git clone git://github.com/jamis/rbenv-gemset.git  ~/.rbenv/plugins/rbenv-gemset
# 通过 gem 命令安装完 gem 后无需手动输入 rbenv rehash 命令, 推荐
git clone git://github.com/sstephenson/rbenv-gem-rehash.git ~/.rbenv/plugins/rbenv-gem-rehash
# 通过 rbenv update 命令来更新 rbenv 以及所有插件, 推荐
git clone git://github.com/rkh/rbenv-update.git ~/.rbenv/plugins/rbenv-update
# 使用 Ruby China 的镜像安装 Ruby, 国内用户推荐
git clone git://github.com/AndorChen/rbenv-china-mirror.git ~/.rbenv/plugins/rbenv-china-mirror
```
```bash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
source ~/.bash_profile
```
```bash
# 安装 ruby
rbenv install --list         # 列出所有 ruby 版本
rbenv install 2.4.0          # 安装 2.4.0
rbenv install jruby-1.7.3    # 安装 jruby-1.7.3

# 列出版本
rbenv versions               # 列出安装的版本
rbenv version                # 列出正在使用的版本

# 设置版本
rbenv global 1.9.3-p392      # 默认使用 1.9.3-p392
rbenv shell 1.9.3-p392       # 当前的 shell 使用 1.9.3-p392, 会设置一个 `RBENV_VERSION` 环境变量
rbenv local jruby-1.7.3      # 当前目录使用 jruby-1.7.3, 会生成一个 `.rbenv-version` 文件

# 其他
rbenv rehash                 # 每当切换 ruby 版本和执行 bundle install 之后必须执行这个命令
rbenv which irb              # 列出 irb 这个命令的完整路径
rbenv whence irb             # 列出包含 irb 这个命令的版本
```

## gemset
```bash
# 创建
# 参数 1 是已安装的 ruby 版本，参数 2 是 gemset 的名字
rbenv gemset create 1.9.3-p392 ruby-china 

# 查看当前 gemset
rbenv gemset active

## 删除
rbenv gemset delete 1.9.3-p392 ruby-china 

# 使用
rbenv gemset init ruby-china

# 列出所有 gemset
rbenv gemset list
```



# <h2 id="cl">命令行选项</h2>
```ruby
ruby -v
```

选项 | 意义
- | -
-0octal | 用八进制指定 IO.gets 等识别的换行符
-a | 指定为自动分割模式 ( 与 -n 或者 -p 选项一起使用时则将 $F 设为 $_.split($;))
-c | 只检查脚本的语法
-Cdirectory | 在脚本执行前，先移动到 directory 目录下
-d、--debug | 使用 debug 模式(将 $DEBUG 设为 true)
-e 'command' | 通过 command 指定一行代码的程序。本选项可指定多个
-Eex[:in]、--encoding=ex[:in] | 指定默认的外部编码(ex)以及默认的内部编码(in)
-Fpattern | 指定 String#split 方法使用的默认分隔符($;)
-i[extension] | 以替换形式编辑 ARGV 文件(指定 extension 时则会生成备份文件)
-Idirectory | 指定追加到 $LOAD_PATH 的目录。本选项可指定多个
-l | 删除 -n 或者 -p 选项中的 $_ 的换行符
-n | 使脚本整体被'while gets();... end'包围(将gets()的结果设定到$_中)
-p | 在 -n 选项的基础上，在每次循环结束时输出 $_
-rlibrary | 在执行脚本前通过 require 引用 library
-s | 要使脚本解析标志(flag)的功能有效('ruby -s script -abc'，则 $abc 为 true)
-S | 从环境变量 PATH 开始搜索可执行的脚本
-Tlevel | 指定不纯度检查模式
-U | 将内部编码的默认值(Encoding.default_internal)设为 UTF-8
-v、--verbose | 显示版本号，冗长模式设定为有效($VERBOSE 设定为 true)
-w | 冗长模式设定为有效
-Wlevel | 指定冗长模式的级别 [0= 不输出警告，1= 只输出重要警告，2= 输出全部警告 (默认值)]
-xdirectory | 忽略执行脚本中 #!ruby 之前的内容
--copyright | 显示版权信息
--enable=feature[, ...] | 使 feature 有效
--disable=feature[, ...] | 使 feature 无效
--external-encoding=encoding | 指定默认的外部编码
--internal-encoding=encoding | 指定默认的内部编码
--version | 显示版本信息
--help | 显示帮助信息

> 上表 --enable、--disable 选项可指定的功能名

功能名 | 意义
- | -
gems | RubyGems 是否有效(默认有效)
rubyopt | 是否引用环境变量 RUBYOPT(默认引用)
did_you_mean | 是否打开拼写检查功能(默认打开)
frozen-string-literal | 是否 freeze 所有字符串字面量(默认否)
all | 上述功能是否全部有效

# <h2 id="const">预定义变量常量</h2>
## 预定义变量
- 预定义变量是指 Ruby 预先定义好的变量，全部都是以 $ 开头的变量，因此可以像全局变 量那样引用

变量名 | 内容
- | -
$! | 最后发生的异常的相关信息
$" | $LOADED_FEATURES 的别名
$$ | 当前执行中的 Ruby 的进程 ID
$& | 最后一次模式匹配后得到的字符串
$' | 最后一次模式匹配中匹配部分之后的字符串
$* | ARGV 的别名
$+ | 最后一次模式匹配中最后一个 () 对应的字符串
$, | Array#join 的默认分割字符串(默认为 nil)
$. | 最后读取的文件的行号
$/ | 输入数据的分隔符(默认为 "\n")
$0 | $PROGRAM_NAME 的别名
$1、$2...... | 最后一次模式匹配中与 () 匹配的字符串(第 n 个 () 对应 $n)
$: | $LOAD_PATH 的别名
$; | String#split 的默认分割字符串(默认为 nil)
$< | ARGF 的别名
$> | print、puts、p 等的默认输出位置(默认为 STDOUT)
$? | 最后执行完毕的子进程的状态
$@ | 最后发生的异常的相关位置信息
$\ | 输出数据的分隔符(默认为 nil)
$_ | gets 方法最后读取的字符串
$` | 最后一次模式匹配中匹配部分之前的字符串
$~ | 最后一次模式匹配相关的信息
$DEBUG | 指定 debug 模式的标识(默认为 nil)
$FILENAME | ARGF 当前在读取的文件名
$LOADED_FEATURES | require 读取的类库名一览表
$LOAD_PATH | 执行 require 读取文件时搜索的目录名数组
$PROGRAM_NAME | 当前执行中的 Ruby 脚本的别名
$SAFE | 安全模式等级(默认 0)
$VERBOSE | 指定冗长模式的标识(默认为 nil)
$stdin | 标准输入(默认为 STDIN)
$stdout | 标准输出(默认为 STDOUT)
$stderr | 标准错误输出(默认为 STDERR)

## 预定义常量
常量名 | 内容
- | -
ARGF | 参数，或者从标准输入得到的虚拟文件对象
ARGV | 命令行参数数组
DATA | 访问 _ _END_ _ 以后数据的文件对象
ENV | 环境变量
RUBY_COPYRIGHT | 版权信息
RUBY_DESCRIPTION | ruby -v 显示的版本信息
RUBY_ENGINE | Ruby 的处理引擎
RUBY_PATCHLEVEL | Ruby 的补丁级别
RUBY_PLATFORM | 运行环境的信息(OS、CPU)
RUBY_RELEASE_DATE | Ruby 的发布日期
RUBY_VERSION | Ruby 的版本
STDERR | 标准错误输出
STDIN | 标准输入
STDOUT | 标准输出

## 伪变量
- 伪变量虽然可以像变量那样引用，但是不能改变其本身的值，对其赋值会产生错误

变量名 | 内容
- | -
self | 默认的接收者
nil、true、false | nil、true、false
`__FILE__` | 执行中的 Ruby 脚本的文件名
`__LINE__` | 执行中的 Ruby 脚本的行编号
`__ENCODING__` | 脚本的编码

## 环境变量
变量名 | 内容
- | -
RUBYLIB | 追加到预定义变量 $LOAD_PATH 中的目录名(各目录间用 : 分隔)
RUBYOPT | 启动 Ruby 时的默认选项(RUBYOPT = "-U -v" 等)
RUBYPATH | -S 选项指定的、解析器启动时脚本的搜索路径
PATH | 外部命令的搜索路径
HOME | DIR.chdir 方法的默认移动位置
LOGDIR | 没有 HOME 时的 DIR.chdir 方法的默认移动位置
LC_ALL、LC_CTYPE、LANG | 决定默认编码的本地信息(平台依赖)
RUBYSHELL、COMSPEC | 执行外部命令时，shell 需要使用的解析器路径(平台依赖)



# <h2 id="basic">基础</h2>
## 注释
```ruby
# 单行注释

=begin
多
行
注
释
=end
```

## 变量
```ruby
a = 123
print a
```

## 控制语句
```ruby
# then 可省略
if expr then
    ...
elseif expr2
    ...
else
    ...
end


while true
    ...
end

# 迭代器 times
3.times do
    ...
end
```

if while 碰到 false 或 nil，认为为假，其余都为真

```ruby
names = ["小林", "林", "高野", "森岗"]
names.each do | name |
    if /林/ =~ name
        puts name
    end
end
# 小林
# 林
```

## 字符串
- 单引号不转义 `'hello.\n'` 输出 `hello.\n`，且不能插值
- 双引号会转义 `"hello.\n"` 输出 `hello 并且有换行`，可插值 `"#{我是变量} aaaa"`

## 数值
```ruby
1    # Fixnum 对象
3.14 # Float 对象

# 数学相关函数
Math.sin
Math.sqrt'
```

## symbol
- 与字符串对象类似，符号也是对象，一般作为名称标签使用，表示方法等的对象的名称
- 符号能实现的功能，大部分字符串也能实现。但在像散列的键这样只是单纯判断 "是否相等" 的处理中，使用符号会比字符串更加有效率

```ruby
sym = :foo
sym2 = :"foo"

# 符号转字符串
sym.to_s
# 字符串转符号
"foo".to_sym
```

## 对象
> ruby 中，一切皆对象。ruby 是强类型的

```ruby
# 数组
names = ["a", "b", "c"]
names[0] # "a"
names[0] = "aa"
names.size # 3

# each 方法返回值为 ["aa", "b", "c"]
names.each do | item |
    puts item
end
```
```ruby
# 散列
# 散列的 key 是符号类型 symbol 或者字符串 在这种只是单纯判断相等的情况下，比字符串性能更好
sym = :foo
sym.to_s     # "foo"
"foo".to_sym # :foo

address1 = { :name => "wt", :id => 1 }
address2 = { name: "wt", id: 1 } # address1 的简写(key 为符号)

address3 = { "name" => "wt", "id" => 1 } # key 为字符串

address1[:name] # "wt"
address1[:id] = 2

# each 方法返回值为 { :name => "wt", :id => 2 }
address1.each do | key, val |
    puts "#{key}: #{val}"
end
```
```ruby
# 正则
# /xxx/ 是正则，=~ 用于匹配正则和字符串
# i 表不区分大小写

/Ruby/ =~ "Ruby"    # 0
/Ruby/ =~ "Diamond" # nil
/Ruby/i =~ "RUBY"   # 0
```

### 常用对象
- 数值对象
    1. `-10、3.1415` 还有表示矩阵、复数、质数、公式的对象
- 字符串对象
- 数组对象、散列对象
- 正则对象
- 时间对象
- 文件对象
- 符号对象
- 范围对象
- 异常对象

### 类
> ruby 中的类 (class) 表示的就是对象的种类

- 对象拥有什么特性等，都是由类决定的
- XX 类的对象，一般也会说成 XX 类的实例 (instance)。所有 ruby 对象其实都是某个类的实例，所以 ruby 中对象和实例的意义几乎是一样的

| 对象 | 类 |
| - | - |
| 数值 | Numeric |
| 字符串 | String |
| 数组 | Array |
| 散列 | Hash |
| 正则表达式 | Regexp |
| 文件 | File |
| 符号 | Symbol |

### nil
- nil 是个特殊的值，表示对象不存在, 在 ruby 中 nil 和 false 都是 "否" 的意思
- **其他一切都为 true (空字符串也是 true)**
- 方法不能返回有意义的值时就会返回 nil
- 从数组或者散列中获取对象时，若指定不存在的索引或者键，则得到的返回值也是 nil

```ruby
a = nil # nil
a.nil?  # true
```

## 方法
ruby 在调用方法时可以省略 `()`

 ```ruby
def foo
    puts 123
end

foo()
foo
```
```ruby
print("hello, ruby.\n")
print "hello, ruby.\n"

print("hello, ", "ruby", ".", "\n")
print "hello, ", "ruby", ".", "\n"
```

## output
`print/puts/p/pp`

- puts 在结尾会自动输出换行符
- 使用 p 方法时，数值结果和字符串结果会以不同的形式输出
- pp 需要 `require "pp` 可以有格式的打印散列、数组

## 引用其他文件
被其他程序引用的程序，称为库

 - require 希望引用的库名
    - 用 require 方法后，Ruby 会搜索参数指定的库，并读取库的所有内容。库内容读取完毕后，程序才会执行 require 方法后面的处理
    - require 方法用于引用已存在的库。只需要指定库名，程序就会在预先定义好的路径下查 找并读取与 Ruby 一起安装的库

```ruby
require "date" # 引入标准库

days = Date.tody - Date.new(1993, 2, 24)
puts(days.to_i) # 8323
```

- require_relative 希望引用的库名
    - require_relative 方法在查找库时，则是根据执行中的程序目录(文件夹)来进行的

 ```ruby
# fun.rb
def foo(a, b)
    puts a, b
end
```
```ruby
# use_fun.rb
require_relative "fun"
foo()
```

## 使用中文
- 在某些 ruby 运行环境中，执行包含中文的脚本时，有可能出错，可在程序首行添加魔法注释类指定编码格式
- 如没指定魔法注释，ruby 会默认使用 UTF-8 编码方式

```ruby
# encoding: GBK
```

此外，使用 p 方法输出中文时也可能会出现乱码，可用 `-E 编码方式` 来指定输出结果的编码方式

```bash
ruby -E UTF-8
irb -E UTF-8
```

## 文件读取
```ruby
# read_text.rb
# 执行命令 ruby read_text.rb test.txt

# 一次读取整个文件内容 1
# 使用 ARGV 这个内置变量可得到命令行传递来的参数数组
filename = ARGV[0]
file = File.open(filename)
text = file.read
print text
file.close

# ---------------------------------------
# 一次读取整个文件内容 2
print File.read(ARGV[0])

# 逐行读
filename = ARGV[0]
file = File.open(filename)
file.each_line do | line |
    print line
end
file.close

# --------------------------------------
# 文件中读取指定模式的内容并输出
# 命令 ruby read_text.rb 模式 文件名
pattern = Regexp.new(ARGV[0])
filename = ARGV[1]

file = File.open(filename)
file.each_line do | line |
    if pattern =~ line
        print line
    end
end
file.close
```
