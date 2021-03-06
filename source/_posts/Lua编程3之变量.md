---
layout: title
title: Lua编程3之变量
date: 2019-06-23 18:07:10
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.a, b, c = 0 a和b和c的值分别是多少？
2.多值赋值的两种常用场景是什么？
3.局部变量的作用域是什么？使用局部变量的两个好处是什么？
4.全局变量_G是什么类型？有什么作用？
5.全局环境存在什么问题？

<!--more-->

# <span style="color:#039BE5;">Lua变量</span>

变量在使用前，必须在代码中进行声明，即创建该变量。

编译程序执行代码之前编译器需要知道如何给语句变量开辟存储区，用于存储变量的值。

Lua变量有三种类型：全局变量、局部变量、表中的域。

Lua 中的变量<span style="color:red">全是全局变量</span>，那怕是语句块或是函数里，除非用local显式声明为局部变量。

局部变量的作用域为从声明位置开始到所在语句块结束。

变量的默认值均为nil。
```lua
-- test.lua 文件脚本
a = 5               -- 全局变量
local b = 5         -- 局部变量

function joke()
    c = 5           -- 全局变量
    local d = 6     -- 局部变量
end

joke()
print(c,d)          --> 5 nil

do 
    local a = 6     -- 局部变量
    b = 6           -- 对局部变量重新赋值
    print(a,b);     --> 6 6
end

print(a,b)      --> 5 6
```
执行以上实例输出结果为：
```lua
5    nil
6    6
5    6
```

## <span style="color:#EF7060;">赋值语句</span>
赋值是改变一个变量的值和改变表域的最基本的方法。
```lua
a = "hello" .. "world"
t.n = t.n + 1
```
Lua可以对多个变量同时赋值，变量列表和值列表的各个元素用逗号分开，赋值语句右边的值会依次赋给左边的变量。
```lua
a, b = 10, 2*x -- 等价于a=10; b=2*x
```
遇到赋值语句Lua会<span style="color:red">先计算右边所有的值</span>然后再执行赋值操作，所以我们可以这样进行交换变量的值：
```lua
x, y = y, x                     -- swap 'x' for 'y'
a[i], a[j] = a[j], a[i]         -- swap 'a[i]' for 'a[j]'
```
当变量个数和值的个数不一致时，Lua会一直以变量个数为基础采取以下策略：
```
a. 变量个数 > 值的个数             按变量个数补足nil
b. 变量个数 < 值的个数             多余的值会被忽略
```
例如：
```lua
a, b, c = 0, 1
print(a,b,c)             --> 0   1   nil

a, b = a+1, b+1, b+2     -- value of b+2 is ignored
print(a,b)               --> 1   2

a, b, c = 0
print(a,b,c)             --> 0   nil   nil
```
上面最后一个例子是一个常见的错误情况，注意：如果要对多个变量赋值必须依次对每个变量赋值。
```lua
a, b, c = 0, 0, 0
print(a,b,c)             --> 0   0   0
```
多值赋值经常用来交换变量，或将函数调用返回给变量：
```lua
a, b = f()
```
f()返回两个值，第一个赋给a，第二个赋给b。

应该尽可能的使用局部变量，有两个好处：

* 避免命名冲突。
* 访问局部变量的速度比全局变量更快。

## <span style="color:#EF7060;">索引</span>
对 table 的索引使用方括号 []。Lua 也提供了 . 操作。
```lua
t[i]
t.i                 -- 当索引为字符串类型时的一种简化写法
gettable_event(t,i) -- 采用索引访问本质上是一个类似这样的函数调用
```
例如：
```lua
books = {}
books["key"] = "大话Unity"
print(books["key"]) -- 大话Unity
print(books.key) -- 大话Unity
```
## <span style="color:#EF7060;">全局变量</span>

Lua将其所有的全局变量保存在一个普通的table中，这个table被称为“环境”。它被保存在全局变量_G中。

Lua中的全局变量不需要声明就可以使用。尽管很方便，但是一旦出现笔误就会造成难以发现的错误。

## <span style="color:#EF7060;">非全局环境</span>
全局环境存在一个问题，即修改它将影响到程序的所有部分。Lua 5为此做了一些改进，新的特征可以支持每个函数拥有自己独立的全局环境，而由该函数创建的闭包函数将继承该函数的全局变量表。这里我们可以通过setfenv函数来改变一个函数的环境，该函数接受两个参数，一个是函数名，另一个是新的环境table。第一个参数除了函数名本身，还可以指定为一个数字，以表示当前函数调用栈中的层数。数字1表示当前函数，2表示它的调用函数，以此类推。

见如下代码：
```lua
a = 1
setfenv(1,{})
print(a)

--输出结果为：
--[[
LuaException: byfile.lua:3: attempt to call a nil value (global 'print')
stack traceback:
    byfile.lua:3: in main chunk
    [C]: in function 'require'
    [string "chunk"]:1: in main chunk
--]]
```
为什么得到这样的结果呢？因为print和变量a一样，都是全局表中的字段，而新的全局表是空的，所以print调用将会报错。

为了解决这个副作用，我们可以让原有的全局表_G作为新全局表的内部表，在访问已有全局变量时，可以直接转到_G中的字段，而对于新的全局字段，则保留在新的全局表中。这样即便是函数中的误修改，也不会影响到其他用到全局变量_G的地方。见如下代码：

> 下面代码中用到了元表的概念，可以后面学过以后再回来看

```lua
a = 1
local newgt = {}  --新环境表
setmetatable(newgt,{__index = _G})
setfenv(1,newgt)
print(a)  --输出1

a = 10
print(a)  --输出10
print(_G.a) --输出1
_G.a = 20
print(a)  --输出10
```
最后给出的示例是函数环境变量的继承性。见如下代码：
```lua
function factory()
    return function() return a end
end
a = 3
f1 = factory()
f2 = factory()
print(f1())  --输出3
print(f2())  --输出3

setfenv(f1,{a = 10})
print(f1())  --输出10
print(f2())  --输出3
```

# <span style="color:#039BE5;">总结</span>

在lua中要尽量少用全局变量，多使用局部变量，需要记住的是：** <span style="color:red">变量默认都是全局变量，局部变量需要加local关键字。</span> **
