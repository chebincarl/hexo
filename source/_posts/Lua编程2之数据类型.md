---
layout: title
title: Lua编程2之数据类型
date: 2019-06-23 16:09:27
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.nil 的“删除”作用怎么理解？如何删除table里的一个值？
2.怎么比较一个变量是否为nil？
3.数字零和空字符串为真吗？假有哪些？
4.如何表示块字符串？如何避免误解析的发生？
5.显式转换函数有哪些？
6.如何计算字符串的长度？
7.如何构建数组？第一个索引是0吗？table会固定长度吗？
8.a = {} a[1000] = 1 和table.maxn()之间有什么关系？
9.function和int类型一样怎么理解？C#中的delegate，class和int一样怎么理解？
10.lua中如何进行字符串连接？
11.Lua将nil作为界定数据结尾的标志会导致什么问题？


<!--more-->

# <span style="color:#039BE5;">数据类型</span>

Lua是一种动态类型的语言，变量本身没有类型，只有值拥有类型。Lua语言本身没有提供类型定义的语法，每个值都“携带”了它自身的类型信息。

在Lua中有8种基础类型，分别是：nil、boolean、number、string、userdata、function、thread和table。

| <center>** 数据类型 ** </center>  | <center>** 描述 ** </center>  |
| :-| :- |
| nil  | 这个最简单，只有值nil属于该类型，表示一个无效值（在条件表达式中相当于false）。  |
| boolean  | 包含两个值：false和true。  |
| number  | 表示双精度类型的实浮点数  |
| string  | 字符串由一对双引号或单引号来表示  |
| function  | 由 C 或 Lua 编写的函数  |
| userdata  | 表示任意存储在变量中的C数据结构  |
| thread  | 表示执行的独立线路，用于执行协同程序  |
| table  | <span style="color:red">Lua 中的表（table）其实是一个“关联数组”（associative arrays），数组的索引可以是数字或者是字符串。</span>在 Lua 里，table 的创建是通过“构造表达式”来完成，最简单构造表达式是{}，用来创建一个空表。  |


我们可以通过type函数获得变量的类型信息，该类型信息将以字符串的形式返回。如：
```lua
print(type("hello world")) --string
print(type(10.4)) --number
print(type(print)) --function
print(type(true)) --boolean
print(type(nil)) --nil
print(type(type(X))) --string
```
## <span style="color:#EF7060;">nil（空）</span>
nil是一种类型，它只有一个值nil，它的主要功能是区别其他任何值。就像之前所说的，<span style="color:red">一个全局变量在第一次赋值前的默认值的默认值就是nil，将nil赋予一个全局变量等同于删除它</span>。Lua将nil用于表示一种“无效值”的情况，类似C#中的null。

例如打印一个没有赋值的变量，便会输出一个 nil 值：
```lua
print(type(a)) -- nil
```
对于全局变量和 table，nil 还有一个“删除”作用，给全局变量或者 table 表里的变量赋一个 nil 值，等同于把它们删掉，如下面代码：
```lua
tab1 = { key1 = "val1", key2 = "val2", "val3" }
for k, v in pairs(tab1) do
    print(k .. " - " .. v)
end

tab1.key1 = nil
for k, v in pairs(tab1) do
    print(k .. " - " .. v)
end
```
nil 作比较时应该加上双引号 ""：
```lua
type(X) --nil
type(X)==nil --false
type(X)=="nil" --true
```
<span style="color:red">type(X)==nil</span> 结果为 false 的原因是因为 <span style="color:red">type(type(X))==string</span>。

## <span style="color:#EF7060;">boolean（布尔）</span>
boolean 类型只有两个可选值：true（真） 和 false（假），Lua 把 false 和 nil 看作是“假”，其他的都为“真”，如数字零和空字符串也为真，这点需要注意。
```lua
print(type(true))
print(type(false))
print(type(nil))

if false or nil then
    print("至少有一个是 true")
else
    print("false 和 nil 都为 false!")
end
```
以上代码执行结果如下：
```lua
boolean
boolean
nil
false 和 nil 都为 false!
```
## <span style="color:#EF7060;">number（数字）</span>
<span style="color:red">Lua 默认只有一种 number 类型 -- double（双精度）类型（默认类型可以修改 luaconf.h 里的定义），以下几种写法都被看作是 number 类型</span>：
```lua
print(type(2))
print(type(2.2))
print(type(0.2))
print(type(2e+1))
print(type(0.2e-1))
print(type(7.8263692594256e-06))
```
<span style="color:red">Lua中没有专门的类型表示整数。</span>

## <span style="color:#EF7060;">string（字符串）</span>
字符串由一对双引号或单引号来表示。
```lua
string1 = "this is string1"
string2 = 'this is string2'
```
Lua支持和C语言类似的字符转义序列，见下表：

| <center>** 转义符 ** </center>  | <center>** 描述 ** </center>  |
| :-| :- |
| \a  | 响铃  |
| \b  | 退格  |
| \n  | 换行  |
| \r  | 回车  |
| \t  | 水平Tab  |
| \  | 反斜杠  |
| \"  | 双引号  |
| \'  | 单引号  |

<span style="color:red">也可以用 2 个方括号 "[[]]" 来表示"一块"字符串，这个时候会禁用里面的转义字符</span>。
```lua
html = [[
<html>
<head></head>
<body>
    <a href="http://www.runoob.com/">菜鸟教程</a>
</body>
</html>
]]
print(html)
```
如果两个方括号中包含这样的内容：<span style="color:red">a = b[c[i]]</span>，这样将会导致Lua的误解析，因此在这种情况下，我们可以将其改为<span style="color:red">[===[ </span>和<span style="color:red"> ]===]</span>的形式，从而避免了误解析的发生。

<span style="color:red">在对一个数字字符串上进行算术操作时，Lua 会尝试将这个数字字符串转成一个数字，这和C#中的加号会进行字符串拼接不同</span>:
```lua
print("2" + 6) --8.0
print("2" + "6") --8.0
print("2 + 6") -- 2 + 6
print("-2e2" * "6") -- -1200.0
print("error" + 1)
--[[
stdin:1: attempt to perform arithmetic on a string value
stack traceback:
    stdin:1: in main chunk
    [C]: in ?
 --]]
 ```
以上代码中"error" + 1执行报错了，** 字符串连接应该使用的是 .. **，如：
```lua
print("a" .. 'b') --ab
print(157 .. 428) --157428
```
尽管Lua提供了这种自动转换的功能，为了避免一些不可预测的行为发生，特别是因为Lua版本升级而导致的行为不一致现象。鉴于此，还是应该尽可能使用显式的转换，如字符串转数字的函数tonumber()，或者是数字转字符串的函数tostring()。对于前者，如果函数参数不能转换为数字，该函数返回nil。如：
```lua
line = "150.56"
n = tonumber(line)
if n == nil then
    error(line .. " is not a valid number")
else
    print(n * 2)
end
```
<span style="color:red">使用 # 来计算字符串的长度，放在字符串前面</span>，如下实例：
```lua
len = "hello world"
print(#len) --11
print(#"hello world") --11
```
## <span style="color:#EF7060;">table（表）</span>
在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。也可以在表里添加一些数据，直接初始化表:
```lua
-- 创建一个空的 table
local tbl1 = {}

-- 直接初始表
local tbl2 = {"apple", "pear", "orange", "grape"}
```
Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以为任意类型(nil除外)。类似C#中的Dictionary的Key-Value结构。
```lua
a = {}
a["key"] = "value"
key = 10
a[key] = 22
a[key] = a[key] + 11
for k, v in pairs(a) do
    print(k .. " : " .. v)
end
```
脚本执行结果为：
```lua
key : value
10 : 33
```
不同于其他语言的数组把 0 作为数组的初始索引，在 Lua 里表的默认初始索引一般以 1 开始。
```lua
local tbl = {"apple", "pear", "orange", "grape"}
for key, val in pairs(tbl) do
    print("Key", key)
end
```
脚本执行结果为：
```lua
Key    1
Key    2
Key    3
Key    4
```
table 不会固定长度大小，有新数据添加时 table 长度会自动增长，没初始的 table 都是 nil。
```lua
a3 = {}
for i = 1, 10 do
    a3[i] = i
end
a3["key"] = "val"
print(a3["key"])
print(a3["none"])
```
脚本执行结果为：
```lua
val
nil
```
在Lua中还提供了另外一种方法用于访问table中的值，见如下示例：
```lua
a.x = 10      --等同于a["x"] = 10
print(a.x)    --等同于print(a["x"])
print(a.y)    --等同于print(a["y"])
```
由于数组实际上仍为一个table，所以对于数组大小的计算需要留意某些特殊的场景，如：
```lua
    a = {}
    a[1000] = 1
```
在上面的示例中，数组a中索引值为1--999的元素的值均为nil。<span style="color:red">而Lua则将nil作为界定数据结尾的标志。</span>当一个数组含有“空隙”时，即中间含有nil值，长度操作符#会认为这些nil元素就是结尾标志。当然这肯定不是我们想要的结果。因此对于这些含有“空隙”的数组，我们可以通过函数table.maxn()返回table的最大正数索引值。如：
```lua
a = {}
a[1000] = 1
print(table.maxn(a))    -- 输出1000
```
## <span style="color:#EF7060;">function（函数）</span>
在Lua中，函数可以存储在变量中，可以通过参数传递其它函数，还可以作为其它函数的返回值。这种特性使语言具有了极大的灵活性。
```lua
function factorial1(n)
    if n == 0 then
        return 1
    else
        return n * factorial1(n - 1)
    end
end
print(factorial1(5))
factorial2 = factorial1
print(factorial2(5))
```
脚本执行结果为：
```lua
120
120
```
function 可以以匿名函数（anonymous function）的方式通过参数传递:
```lua
function testFun(tab,fun)
    for k ,v in pairs(tab) do
        print(fun(k,v));
    end
end


tab={key1="val1",key2="val2"};
testFun(tab,
function(key,val)--匿名函数
    return key.."="..val;
end
);
```
脚本执行结果为：
```lua
key1 = val1
key2 = val2
```
## <span style="color:#EF7060;">thread（线程）</span>
在 Lua 里，thread代表了单独线程的执行，并且用来实现lua里的协程（coroutine），类似Unity中的协程。它跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。

Lua的线程和操作系统的线程并不相关。Lua的协程支持所有的操作系统，即使那些不支持线程的操作系统也支持协程。

## <span style="color:#EF7060;">userdata（自定义类型）</span>
userdata 是一种用户自定义数据，提供了将任意外部数据（通常是 struct 和 指针）存储在lua变量中的能力。一个userdata代表了一块内存数据。

有两种userdata，一种是full userdata，是lua管理的一个对象，拥有一块内存区域；还有一种叫light userdata，是一个指针。

在Lua中除了赋值和比较，没有其他针对userdata预先定义的操作，通常使用metatable对full userdata定义操作。userdata的值无法在lua中创建或者修改，只能通过C语言API，这也保证了和宿主程序数据的一致性。