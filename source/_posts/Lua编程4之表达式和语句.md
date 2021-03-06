---
layout: title
title: Lua编程4之表达式和语句
date: 2019-06-23 22:12:35
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.如何创建一个空表？

<!--more-->

学习了数据类型和变量之后，就可以开始写代码的表达式，然后进一步写语句了。

# <span style="color:#039BE5;">Lua表达式</span>

## <span style="color:#EF7060;">算术操作符</span>

Lua支持常规算术操作符有：二元的“+”、“-”、“\*”、“/”、“^”(指数)、“%”(取模)，一元的“-”(负号)。所有这些操作符都可用于实数。

然而需要特别说明的是取模操作符(%)，Lua中对该操作符的定义为：
```lua
a % b == a - floor(a / b) * b
```
由此可以推演出x % 1的结果为x的小数部分，而x - x % 1的结果则为x的整数部分。类似的，x - x % 0.01则是x精确到小数点后两位的结果。

## <span style="color:#EF7060;">关系操作符</span>
Lua支持的关系操作符有：>、<、>=、<=、==、\~ =（不等于），所有这些操作符的结果均为true或false。

操作符==用于相等性测试，操作符~ =用于不等性测试。这两个操作符可以应用于任意两个值。如果两个值的类型不同，Lua就认为他们不等。nil值只与其自身相等。

对于table、userdata和函数，Lua是通过引用进行比较的。也就是说，只有当他们引用同一个对象时，才视为相等。如：
```lua
a = {}
a.x = 1
a.y = 0
b = {}
b.x = 1
b.y = 1
c = a
```
其结果是a == c，但a ~ = b。
对于字符串的比较，Lua是按照字符次序比较的。

## <span style="color:#EF7060;">逻辑操作符</span>
Lua支持的逻辑操作符有：and、or和not。

与条件控制语句一样，所有的逻辑操作符都将false和nil视为假，其他的结果均为真。

和其他大多数语言一样，Lua中的and和or都使用“短路原则”。

> 短路原则：逻辑运算时，如果已经计算的表达式已经可以得出最终结果，就不对后面的表达式继续计算。
(表达式1）and (表达式2) 如果表达式1为假，则表达式2不会进行运算，即表达式2“被短路”
(表达式1）or (表达式2) 如果表达式1为真，则表达式2不会进行运算，即表达式2“被短路”

在Lua中有一种惯用写法x = x or v，它等价于：if not x then x = v end。

这里还有一种基于“短路原则”的惯用写法，如：
```lua
x = 10
y = 9
max = (x > y) and x or y
print(max) -- 10
```
这等价于C#语言中max = (x > y) ? x : y。

## <span style="color:#EF7060;">字符串连接</span>

昨天已经提到了字符串连接操作符(..)，这里再给出一些简单的示例。
```lua
print("Hello " .. "World) -- Hello World
print(0 .. 1)  --01
```
即使连接操作符的操作数为数值类型，在执行时Lua仍会将其自动转换为字符串。

## <span style="color:#EF7060;">table构造器</span>
构造器用于构建和初始化table的表达式。这是Lua特有的表达式，也是Lua中最有用、最通用的机制之一。

其中最简单的构造器是空构造器<span style="color:red">{}</span>，用于创建空table。

我们通过构造器还可以初始化数组，如：
```lua
days = {"Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"}
for i = 1,#days do
    print(days[i])
end
--输出结果为
--Sunday
--Monday
--Tuesday
--Wednesday
--Thursday
--Friday
--Saturday
```
从输出结果可以看出，days在构造后会将自动初始化，其中days[1]被初始化为"Sunday"，days[2]为"Monday"，以此类推。

Lua中还提供了另外一种特殊的语法用于初始化记录风格的table。
如：a = { x = 10, y = 20 }，其等价于：a = {}; a.x = 10; a.y = 20

在实际编程时我们也可以将这两种初始化方式组合在一起使用，如：
```lua
polyline = {color = "blue", thickness = 2, npoints = 4, 
    {x = 0, y = 0},
    {x = 10, y = 0},
    {x = -10, y = 1},
    {x = 0, y = 1} }
print(polyline["color"]);
print(polyline[2].x)
print(polyline[4].y)
--输出结果如下：
--blue
--10
--1
```
除了以上两种构造初始化方式之外，Lua还提供另外一种更为通用的方式，如：
```lua
opnames = { ["+"] = "add", ["-"] = "sub", ["*"] = "mul", ["/"] = "div"}
print(opnames["+"])
i = 20; s = "-"
a = { [i + 0] = s, [i + 1] = s .. s, [i + 2] = s..s..s }
print(a[22])
```
对于table的构造器，还有两个需要了解的语法规则，如：
```lua
a = { [1] = "red", [2] = "green", [3] = "blue", }
```
这里需要注意最后一个元素的后面仍然可以保留逗号(,)，这一点类似于C语言中的枚举。
```lua
a = {x = 10, y = 45; "one", "two", "three" }
```
可以看到上面的声明中同时存在逗号,和分号;两种元素分隔符，这种写法在Lua中是允许的。我们通常会将分号;用于分隔不同初始化类型的元素，如上例中分号之前的初始化方式为记录初始化方式，而后面则是数组初始化方式。

# <span style="color:#039BE5;">Lua语句</span>

## <span style="color:#EF7060;">赋值语句</span>
Lua中的赋值语句和其它编程语言基本相同，唯一的差别是Lua支持“多重赋值”，如：a, b = 10, 2 * x，其等价于a = 10; b = 2 * x。

然而需要说明的是，Lua在赋值之前需要先计算等号右边的表达式，在每一个表达式都得到结果之后再进行赋值。因此，我们可以这样写变量交互：x,y = y,x。

如果等号右侧的表达式数量少于左侧变量的数量，Lua会将左侧多出的变量的值置为nil，如果相反，Lua将忽略右侧多出的表达式。

## <span style="color:#EF7060;">局部变量与块</span>
Lua中的局部变量定义语法为：local i = 1，其中local关键字表示该变量为局部变量。和全局变量不同的是，局部变量的作用范围仅限于其所在的程序块。Lua中的程序可以为控制结构的执行体、函数执行体或者是一个程序块，如：

下面的x变量仅在while循环内有效。
```lua
while i <= x do
    local x = i * 2
    print(x)
    i = i + 1
end
```
和其它编程语言一样，如果有可能尽量使用局部变量，以免造成全局环境的变量名污染。同时由于局部变量的有效期更短，这样垃圾收集器可以及时对其进行清理，从而得到更多的可用内存。

## <span style="color:#EF7060;">流程控制</span>
Lua中提供的控制语句和其它大多数开发语言所提供的基本相同，因此这里仅仅是进行简单的列举。然后再给出差异部分的详细介绍。如：

** if then else **
```lua
if a < 0 then
    b = 0
else
    b = 1
end
```
** if elseif else then **
```lua
if a < 0 then 
    b = 0
elseif a == 0 then
    b = 1
else
    b = 2
end
```

** while **

```lua

local i= 1
while a[i] do
    print(a[i])
    i = i + 1
end
```
** repeat **

类似C#中的while…do
```lua
repeat
    line = io.read()
until line ~= "" --直到until的条件为真时结束。
print(line)
```
** for **
```lua
for var=begin,last,step do --如果没有step变量，begin的缺省步长为1。
    i = i + 1
end
```
需要说明的是，for循环开始处的三个变量begin、last和step，如果它们是表达式的返回值，那么该表达式将仅执行一次。再有就是不要在for的循环体内修改变量var的值，否则会导致不可预知的结果。

** foreach **
```lua
for i, v in ipairs(a) do  --ipairs是Lua自带的系统函数，返回遍历数组的迭代器。
    print(v)
end

for k in pairs(t) do      --打印table t中的所有key。
    print(k)
end
```
见如下示例代码：
```lua
days = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday" }
revDays = {}
for k, v in ipairs(days) do
    revDays[v] = k
end

for k in pairs(revDays) do
    print(k .. " = " .. revDays[k])
end

--输出结果为：
--Saturday = 7
--Tuesday = 3
--Wednesday = 4
--Friday = 6
--Sunday = 1
--Thursday = 5
--Monday = 2
```
** break **
和C#语言循环中的break语义完全相同，即跳出最内层循环。
