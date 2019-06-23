---
layout: title
title: Lua编程7之metatable
date: 2019-06-23 22:47:23
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.如何创建一个空表？

<!--more-->

在 Lua table 中可以访问对应的key来得到value值，但是却无法对两个 table 进行操作。要实现两个table的操作就要使用Lua提供的元表(Metatable)，元表允许我们改变table的行为，每个行为关联了对应的元方法。

# <span style="color:#039BE5;">Lua元表</span>
Lua中每一个值都有metatable。metatable是一个普通的table，定义了一个值在特定情况下的操作。你可以通过修改值的metatable来修改这些操作。

> 在Lua代码中，只能设置table的元表，若要设置其它类型值的元表，则必须通过C代码来完成。

table和userdata可以有各自独立的元表，而其它数据类型的值则共享其类型所属的单一元表。缺省情况下，table在创建时没有元表。

任何table都可以作为任何值的元表，而一组相关的table也可以共享一个通用的元表，此元表将描述了它们共同的行为。一个table甚至可以作为它自己的元表，用于描述其特有的行为。

例如，使用元表我们可以定义Lua如何计算两个table的相加操作a+b。

当Lua试图对两个表进行相加时，先检查两者之一是否有元表，之后检查是否有一个叫<span style="color:red">\_\_add</span>的字段，若找到，则调用对应的值。<span style="color:red">\_\_add</span>等字段其对应的值（往往是一个函数或是table）就是<span style="color:red">** 元方法 ** </span>，前面有两个下划线。

有两个很重要的函数来处理元表：

* <span style="color:red">setmetatable(table,metatable)</span>: 对指定 table 设置元表(metatable)，如果元表(metatable)中存在 \_\_metatable 键值，setmetatable 会失败。
* <span style="color:red">getmetatable(table)</span>: 返回对象的元表(metatable)。

以下实例演示了如何对指定的表设置元表：
```lua
mytable = {}                      -- 普通表 
mymetatable = {}                  -- 元表
setmetatable(mytable,mymetatable) -- 把 mymetatable 设为 mytable 的元表 
```
以上代码也可以直接写成一行：
```lua
mytable = setmetatable({},{})
```
以下为返回对象元表：
```lua
getmetatable(mytable) -- 这会返回mymetatable
```

## <span style="color:#EF7060;">算术类的元方法</span>
在下面的示例代码中，将用table来表示集合，并且有一些函数用来计算集合的并集和交集等。
```lua
Set = {}
local metatable = {} --元表

--根据参数列表中的值创建一个新的集合
function Set.new(l)
    local set = {}
    --将所有由该方法创建的集合的元表都指定到metatable
    setmetatable(set, metatable)
    for _, v in ipairs(l) do
        set[v] = true
    end
    return set
end

--取两个集合并集的函数
function Set.union(a, b)
    local res = Set.new {}
    for k in pairs(a) do
        res[k] = true
    end
    for k in pairs(b) do
        res[k] = true
    end
    return res
end

--取两个集合交集的函数
function Set.intersection(a, b)
    local res = Set.new {}
    for k in pairs(a) do
        res[k] = b[k]
    end
    return res
end

function Set.tostring(set)
    local l = {}
    for e in pairs(set) do
        l[#l + 1] = e
    end
    return "{" .. table.concat(l, ", ") .. "}"
end

function Set.print(s)
    print(Set.tostring(s))
end

--最后将元方法加入到元表中，这样当两个由Set.new方法创建出来的集合进行
--加运算时，将被重定向到Set.union方法，乘法运算将被重定向到Set.intersection
metatable.__add = Set.union
metatable.__mul = Set.intersection

--下面为测试代码
s1 = Set.new {10, 20, 30, 50}
s2 = Set.new {30, 1}
s3 = s1 + s2
Set.print(s3)
Set.print(s3 * s1)

--输出结果为：
--{1, 30, 10, 50, 20}
--{30, 10, 50, 20}
```
在元表中，每种算术操作符都有对应的字段名，对应的操作列表如下：(注意：\_\_是两个下划线)

| <center>** 模式 ** </center>  | <center>** 描述 ** </center>  |
| :-| :- |
| __add  | 对应的运算符 '+'.  |
| __sub  | 对应的运算符 '-'.  |
| __mul  | 对应的运算符 '*'.  |
| __div  | 对应的运算符 '/'.  |
| __mod  | 对应的运算符 '%'.  |
| __unm  | 对应的运算符 '-'.  |
| __concat  | 对应的运算符 '..'.  |
| __eq  | 对应的运算符 '=='.  |
| __lt  | 对应的运算符 '<'.  |
| __le  | 对应的运算符 '<='.  |

对于上面的示例代码，我们在算术运算符的两侧均使用了table类型的操作数。那么如果为s1 = s1 + 8，Lua是否还能正常工作呢？答案是肯定的，因为Lua定位元表的步骤为，如果第一个值有元表，且存在\_\_add字段，那么Lua将以这个字段为元方法，否则会再去查看第二个值否是有元表且包含\_\_add字段，如果有则以此字段为元方法。最后，如果两个值均不存在元方法，Lua就引发一个错误。然而对于上例中的Set.union函数，如果执行s1 = s1 + 8将会引发一个错误，因为8不是table对象，不能基于它执行pairs方法调用。为了得到更准确的错误信息，我们需要给Set.union函数做如下的修改，如：
```lua
function Set.union(a,b)
    if getmetatable(a) ~= metatable or getmetatable(b) ~= metatable then
        error("attempt to 'add' a set with a non-set value")
    end
    --后面的代码与上例相同。
    ... ...
end
```
## <span style="color:#EF7060;">关系类的元方法</span>
元表还可以指定关系操作符的含义，元方法分别为\_\_eq(等于)、\_\_lt(小于)和\_\_le(小于等于)，至于另外3个关系操作符，Lua没有提供相关的元方法，可以通过前面3个关系运算符的取反获得。

| <center>** 模式 ** </center>  | <center>** 描述 ** </center>  |
| :-| :- |
| __eq  | 对应的运算符 '=='.  |
| __lt  | 对应的运算符 '<'.  |
| __le  | 对应的运算符 '<='.  |

与算术类的元方法不同，关系类的元方法不能应用于混合的类型。

示例如下：
```lua
Set = {}
local metatable = {}

function Set.new(l)
    local set = {}
    setmetatable(set,metatable)
    for _, v in ipairs(l) do
        set[v] = true
    end
    return set
end

metatable.__le = function(a,b) 
    for k in pairs(a) do
        if not b[k] then return false end
    end
    return true
end
metatable.__lt = function(a,b) return a <= b and not (b <= a) end
metatable.__eq = function(a,b) return a <= b and b <= a end

--下面是测试代码：
s1 = Set.new{2,4}
s2 = Set.new{4,10,2}
print(s1 <= s2) --true
print(s1 < s2)  --true
print(s1 >= s1) --true
print(s1 > s1)  --false
```
## <span style="color:#EF7060;">库定义的元方法</span>
除了上述基于操作符的元方法外，Lua还提供了一些针对框架的元方法，如print函数总是调用tostring来格式化其输出。如果当前对象存在__tostring元方法时，tostring将用该元方法的返回值作为自己的返回值，如：
```lua
Set = {}
local metatable = {}

function Set.new(l)
    local set = {}
    setmetatable(set,metatable)
    for _, v in ipairs(l) do
        set[v] = true
    end
    return set
end

function Set.tostring(set)
    local l = {}
    for e in pairs(set) do
        l[#l + 1] = e
    end
    return "{" .. table.concat(l,", ") .. "}";
end

metatable.__tostring = Set.tostring


--下面是测试代码：
s1 = Set.new{4,5,10}
print(s1) --{5,10,4}
```
函数setmetatable和getmetatable也会用到元表中的一个字段(\_\_metatable)，用于保护元表，如：
```lua
mt.__metatable = "not your business"
s1 = Set.new{}
print(getmetatable(s1))   --此时将打印"not your business"
setmetatable(s1,{})  --此时将输出错误信息："cannot change protected metatable"
```
从上述代码的输出结果即可看出，一旦设置了__metatable字段，getmetatable就会返回这个字段的值，而setmetatable将引发一个错误。


## <span style="color:#EF7060;">table访问的元方法</span>
算术类和关系类运算符的元方法都为各种错误情况定义了行为，它们不会改变语言的常规行为。但是Lua还提供了一种可以改变table行为的方法。有两种可以改变的table行为：查询table及修改table中不存在的字段。

** \_\_index 元方法 **

这是 metatable 最常用的键（key）。

当你通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）中的\_\_index键。如果\_\_index包含一个表，Lua会在表中查找相应的键。

例如：
```lua
other = { foo = 3 } 
t = setmetatable({}, { __index = other }) 
print(t.foo) -- 3
print(t.bar) -- nil
```
如果__index包含一个函数的话，Lua就会调用那个函数，table和键会作为参数传递给函数。

\_\_index 元方法查看表中元素是否存在，如果不存在，返回结果为 nil；如果存在则由 \_\_index 返回结果。
```lua
mytable = setmetatable({key1 = "value1"}, {
  __index = function(mytable, key)
    if key == "key2" then
      return "metatablevalue"
    else
      return nil
    end
  end
})

print(mytable.key1,mytable.key2)
```
实例输出结果为：
```lua
value1    metatablevalue
```
实例解析：

* mytable 表赋值为 {key1 = "value1"}。
* mytable 设置了元表，元方法为 \_\_index。
* 在mytable表中查找 key1，如果找到，返回该元素，找不到则继续。
* 在mytable表中查找 key2，如果找到，返回 metatablevalue，找不到则继续。
* 判断元表有没有\_\_index方法，如果\_\_index方法是一个函数，则调用该函数。
* 元方法中查看是否传入 "key2" 键的参数（mytable.key2已设置），如果传入 "key2" 参数返回 "metatablevalue"，否则返回 mytable 对应的键值。

我们还可以将以上代码简单写成：
```lua
mytable = setmetatable({key1 = "value1"}, { __index = { key2 = "metatablevalue" } })
print(mytable.key1,mytable.key2)
```
如果想在访问table时禁用__index元方法，可以通过函数rawget(table,key)完成。通过该方法并不会加速table的访问效率。

** 总结 **

Lua查找一个表元素时的规则，其实就是如下3个步骤:

* 在表中查找，如果找到，返回该元素，找不到则下一步
* 判断该表是否有元表，如果没有元表，返回nil，有元表则下一步
* 判断元表有没有\_\_index方法，如果\_\_index方法为nil，则返回nil；如果\_\_index方法是一个表，则重复1、2、3；如果\_\_index方法是一个函数，则返回该函数的返回值。

** \_\_newindex 元方法 **
\_\_newindex 元方法用来对表更新，\_\_index则用来对表访问 。

当你给表的一个缺少的索引赋值，解释器就会查找__newindex元方法：如果存在则调用这个函数而不进行赋值操作。

以下实例演示了 \_\_newindex 元方法的应用：
```lua
mymetatable = {}
mytable = setmetatable({key1 = "value1"}, { __newindex = mymetatable })

print(mytable.key1)

mytable.newkey = "新值2"
print(mytable.newkey,mymetatable.newkey)

mytable.key1 = "新值1"
print(mytable.key1,mymetatable.key1)
```
以上实例执行输出结果为：
```lua
value1
nil    新值2
新值1    nil
```
以上实例中表设置了元方法\_\_newindex，在对新索引键（newkey）赋值时（mytable.newkey = "新值2"），会调用元方法，而不进行赋值。而如果对已存在的索引键（key1），则会进行赋值，而不调用元方法 \_\_newindex。

以下实例使用了 rawset 函数来更新表：

> <span style="color:red"> ** rawset (table, index, value) ** </span>
将table[index]的值直接设为value，不调用__newindex元方法。

```lua
mytable =
    setmetatable(
    {key1 = "value1"},
    {
        __newindex = function(mytable, key, value)
            rawset(mytable, key, '"' .. value .. '"')
        end
    }
)

mytable.key1 = "new value"
mytable.key2 = 4

print(mytable.key1, mytable.key2)
```
以上实例执行输出结果为：
```lua
new value    "4"
```

# <span style="color:#039BE5;">总结</span>

从本文中可以知道元表可以很好的简化我们的代码功能，所以了解 Lua 的元表，可以让我们写出更加简单优秀的 Lua 代码。