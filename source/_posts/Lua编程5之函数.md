---
layout: title
title: Lua编程5之函数
date: 2019-06-23 22:22:14
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.编程中最重要的是什么？
2.描述一下函数的两种用途。
3.如何指定为局部函数？
4.函数可以返回多个值吗？如何返回？
5.foo = function(x) return 2 * x end 如何调用？
6.Lua中实参和形参的数量不一致如何处理？
7.可变参数是什么意思？如何使用？动手实现print("平均值为",average(10,5,3,4,5,6))

<!--more-->

编程中最重要的就是如何提高<span style="color:red">代码的复用</span>，最基础的方法就是提取函数。今天就来看看Lua中如何编写函数。

# <span style="color:#039BE5;">Lua函数</span>
在Lua中，函数是对语句和表达式进行抽象的主要方法。既可以用来处理一些特殊的工作，也可以用来计算一些值。

Lua 提供了许多的内建函数，你可以很方便的在程序中调用它们，如print()函数可以将传入的参数打印在控制台上。

Lua 函数主要有两种用途：

* 完成指定的任务，这种情况下函数作为调用语句使用；（例如print）
* 计算并返回值，这种情况下函数作为赋值语句的表达式使用。（例如add()）

## <span style="color:#EF7060;">函数定义</span>
Lua 编程语言函数定义格式如下：
```lua
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end
```
解析：

<span style="color:blue">optional_function_scope</span>: 该参数是可选的，指定函数是全局函数还是局部函数，未设置该参数默认为全局函数，如果你需要设置函数为局部函数需要使用关键字 local。

<span style="color:blue">function_name</span>: 指定函数名称。

<span style="color:blue">argument1, argument2, argument3…, argumentn</span>: 函数参数，多个参数以逗号隔开，函数也可以不带参数。

<span style="color:blue">function_body</span>: 函数体，函数中需要执行的代码语句块。

<span style="color:blue">result_params_comma_separated</span>: 函数返回值，Lua语言函数可以返回多个值，每个值以逗号隔开。类似C#中的元组。

在声明Lua函数时，可以直接给出所谓的函数名，如：
```lua
function foo(x) return 2 * x end
```
我们同样可以使用下面这种更为简化的方式声明Lua中的函数，类似C#中的匿名方法，如：
```lua
foo = function(x) return 2 * x end
```
** 实例 **
以下实例定义了函数 max()，参数为 num1, num2，用于比较两值的大小，并返回最大值：
```lua
--[[ 函数返回两个值的最大值 --]]
function max(num1, num2)

   if (num1 > num2) then
      result = num1;
   else
      result = num2;
   end

   return result; 
end
-- 调用函数
print("两值比较最大值为 ",max(10,4))
print("两值比较最大值为 ",max(5,6))
```
以上代码执行结果为：
```
两值比较最大值为     10
两值比较最大值为     6
```
Lua 中我们可以将函数作为参数传递给函数，如下实例：
```lua
myprint = function(param)
   print("这是打印函数 -   ##",param,"##")
end

function add(num1,num2,functionPrint)
   result = num1 + num2
   -- 调用传递的函数参数
   functionPrint(result)
end
myprint(10)
-- myprint 函数作为参数传递
add(2,5,myprint)
```
以上代码执行结果为：
```
这是打印函数 -   ##    10    ##
这是打印函数 -   ##    7    ##
```
## <span style="color:#EF7060;">函数调用</span>
在Lua中函数的调用方式和C#语言基本相同，如：print("Hello World")和a = add(x, y)。

唯一的差别是，如果函数只有一个参数，并且该参数的类型为字符串常量或table的构造器，那么圆括号可以省略，如<span style="color:red">print "Hello World"和f {x = 20, y = 20}</span>。

Lua为面对对象式的调用也提供了一种特殊的语法--冒号操作符。表达式<span style="color:red">o.foo(o,x)</span>的另一种写法是<span style="color:red">o:foo(x)</span>。冒号操作符使调用o.foo时将o隐含的作为函数的第一个参数。

需要说明的是，** <span style="color:red">Lua中实参和形参的数量可以不一致</span> ** ，一旦出现这种情况，Lua的处理规则等同于多重赋值，即实参多于形参，多出的部分被忽略，如果相反，没有被初始化的形参的缺省值为nil。

## <span style="color:#EF7060;">多返回值</span>
Lua函数可以返回多个结果值，比如string.find，其返回匹配串"开始和结束的下标"（如果不存在匹配串返回nil）。
```lua
s, e = string.find("hello world", "wo") 
print(s, e) -- 7    8
```
Lua函数中，在return后列出要返回的值的列表即可返回多值，如：
```lua
function maximum (a)
    local mi = 1             -- 最大值索引
    local m = a[mi]          -- 最大值
    for i,val in ipairs(a) do
       if val > m then
           mi = i
           m = val
       end
    end
    return m, mi
end

print(maximum({8,10,23,12,5}))
```
以上代码执行结果为：
```
23    3
```
## <span style="color:#EF7060;">可变参数</span>
Lua 函数可以接受可变数目的参数，和 C 语言类似，在函数参数列表中使用三点 … 表示函数有可变的参数。
```lua
function add(...)  
local s = 0  
  for i, v in ipairs{...} do   --> {...} 表示一个由所有变长参数构成的数组  
    s = s + v  
  end  
  return s  
end  
print(add(3,4,5,6,7))  --->25
```
我们可以将可变参数赋值给一个变量。

例如，我们计算几个数的平均值：
```lua
function average(...)
   result = 0
   local arg={...}    --> arg 为一个表，局部变量
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. #arg .. " 个数")
   return result/#arg
end

print("平均值为",average(10,5,3,4,5,6))
```
以上代码执行结果为：
```
总共传入 6 个数
平均值为    5.5
```
我们也可以通过 select("#",…) 来获取可变参数的数量:
```lua
function average(...)
   result = 0
   local arg={...}
   for i,v in ipairs(arg) do
      result = result + v
   end
   print("总共传入 " .. select("#",...) .. " 个数")
   return result/select("#",...)
end

print("平均值为",average(10,5,3,4,5,6))
```
以上代码执行结果为：
```lua
总共传入 6 个数
平均值为    5.5
```
有时候我们可能需要几个固定参数加上可变参数，固定参数必须放在变长参数之前:
```lua
function fwrite(fmt, ...)  ---> 固定的参数fmt
    return io.write(string.format(fmt, ...))     
end

fwrite("hongiu\n")       --->fmt = "hongiu", 没有变长参数。  
fwrite("%d%d\n", 1, 2)   --->fmt = "%d%d", 变长参数为 1 和 2
```
输出结果为：
```
hongiu
12
```
通常在遍历变长参数的时候只需要使用<span style="color:red">{…}，然而变长参数可能会包含一些 nil，那么就可以用 select 函数来访问变长参数了：<span style="color:red">select('#', …) 或者<span style="color:red">select(n, …)

* <span style="color:red">select('#', …)</span> 返回可变参数的长度
* <span style="color:red">select(n, …)</span> 用于访问 n 到 select('#',…) 的参数

调用select时，必须传入一个固定实参selector(选择开关)和一系列变长参数。如果selector为数字n,那么select返回它的第n个可变实参，否则只能为字符串"#",这样select会返回变长参数的总数。例子代码：
```lua
do  
    function foo(...)  
        for i = 1, select('#', ...) do  -->获取参数总数
            local arg = select(i, ...); -->读取参数
            print("arg", arg);  
        end  
    end  

    foo(1, 2, 3, 4);  
end
```
输出结果为：
```
arg    1
arg    2
arg    3
arg    4
```