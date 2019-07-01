---
layout: title
title: Lua编程8之面向对象
date: 2019-06-23 23:18:25
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.如何创建一个空表？

<!--more-->

之前学习了table，Lua中table很重要的一个功能就是实现面向对象的架构。因为lua本身并不是面向对象的语言，但是通过table可以实现。

# <span style="color:#039BE5;">Lua 面向对象</span>

面向对象编程（Object Oriented Programming，OOP）是一种非常流行的计算机编程架构。

以下几种编程语言都支持面向对象编程：C# C++ Java Objective-C Ruby

## <span style="color:#EF7060;">面向对象特征</span>
1）** <span style="color:red">封装</span> ** ：指能够把一个实体的信息、功能、响应都装入一个单独的对象中的特性。

2）** <span style="color:red">继承</span> ** ：继承的方法允许在不改动原程序的基础上对其进行扩充，这样使得原功能得以保存，而新功能也得以扩展。这有利于减少重复编码，提高软件的开发效率。

3）** <span style="color:red">多态</span> ** ：同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果。在运行时，可以通过指向基类的指针，来调用实现派生类中的方法。

4）** <span style="color:red">抽象</span> ** ：抽象(Abstraction)是简化复杂的现实问题的途径，它可以为具体问题找到最恰当的类定义，并且可以在最恰当的继承级别解释问题。

## <span style="color:#EF7060;">Lua 中面向对象</span>
我们知道，对象由属性和方法组成。LUA中最基本的结构是table，所以需要用table来描述对象的属性。

lua中的function可以用来表示方法。那么LUA中的类可以通过table + function模拟出来。

Lua中的表不仅在某种意义上是一种对象。像对象一样，表也有状态（成员变量）；也有与对象的值独立的本性，特别是拥有两个不同值的对象（table）代表两个不同的对象；一个对象在不同的时候也可以有不同的值，但他始终是一个对象；与对象类似，表的生命周期与其由什么创建、在哪创建没有关系。

但是如果直接使用table仍然会存在大量的问题，见如下代码：
```lua
Account = {balance = 0}
function Account.withdraw(v)
    Account.balance = Account.balance - v
end
--下面是测试调用函数
Account.withdraw(100.00)
```
在上面的withdraw函数内部依赖了全局变量Account，一旦该变量发生改变，将会导致withdraw不再能正常的工作，如：
```lua
a = Account; Account = nil
a.withdraw(100.00)  --将会导致访问空nil的错误。
```
这种行为明显的违反了面向对象封装性和实例独立性。要解决这一问题，我们需要给withdraw函数在添加一个参数self，等价于C#中的this，见如下修改：
```lua
function Account.withdraw(self, v)
    self.balance = self.balance - v
end
--下面是基于修改后代码的调用：
a1 = Account
Account = nil
a1.withdraw(a1, 100.00) --正常工作
```
针对上述问题，Lua提供了一种更为便利的语法，即将点(.)替换为冒号(:)，这样可以在定义和调用时均隐藏self参数，如:
```lua
function Account:withdraw(v)
    self.balance = self.balance - v
end
--调用代码可改为：
a:withdraw(100.00)
```
** <span style="color:red">一个简单实例</span> **
以下简单的类包含了三个属性： area, length 和 breadth，printArea方法用于打印计算结果：
```lua
-- Meta class
Rectangle = {area = 0, length = 0, breadth = 0}

-- 派生类的方法 new
function Rectangle:new (o,length,breadth)
    o = o or {}
    setmetatable(o, self)
    self.__index = self
    self.length = length or 0
    self.breadth = breadth or 0
    self.area = length*breadth;
    return o
end

-- 派生类的方法 printArea
function Rectangle:printArea ()
    print("矩形面积为 ",self.area)
end
```
** 创建对象 **

创建对象是为类的实例分配内存的过程。每个类都有属于自己的内存并共享公共数据。
```lua
r = Rectangle:new(nil,10,20)
```
内存在对象初始化时分配。

** 访问属性 **
我们可以使用点号(.)来访问类的属性：
```lua
print(r.length)
```
** 访问成员函数 **
我们可以使用冒号 : 来访问类的成员函数：
```lua
r:printArea()
```
** <span style="color:red">完整实例</span> **
以下我们演示了 Lua 面向对象的完整实例：
```lua
-- Meta class
Shape = {area = 0}

-- 基础类方法 new
function Shape:new (o,side)
    o = o or {}
    setmetatable(o, self)
    self.__index = self
    side = side or 0
    self.area = side*side;
    return o
end

-- 基础类方法 printArea
function Shape:printArea ()
    print("面积为 ",self.area)
end

-- 创建对象
myshape = Shape:new(nil,10)

myshape:printArea()
```
执行以上程序，输出结果为：
```
面积为     100
```
** <span style="color:red">Lua 继承</span> **
继承是指一个对象直接使用另一对象的属性和方法。可用于扩展基础类的属性和方法。

以下演示了一个简单的继承实例：
```lua
 -- Meta class
Shape = {area = 0}

-- 基础类方法 new
function Shape:new (o,side)
    o = o or {}
    setmetatable(o, self)
    self.__index = self
    side = side or 0
    self.area = side*side;
    return o
end

-- 基础类方法 printArea
function Shape:printArea ()
    print("面积为 ",self.area)
end
```
接下来的实例，Square 对象继承了 Shape 类:
```lua
Square = Shape:new()

-- 派生类的new方法
function Square:new (o,side)
    o = o or Shape:new(o,side)
    setmetatable(o, self)
    self.__index = self
    return o
end
```
** 完整实例 **
以下实例我们继承了一个简单的类，来扩展派生类的方法，派生类中保留了继承类的成员变量和方法：
```lua
 -- Meta class
Shape = {area = 0}

-- 基础类方法 new
function Shape:new (o,side)
    o = o or {}
    setmetatable(o, self)
    self.__index = self
    side = side or 0
    self.area = side*side;
    return o
end

-- 基础类方法 printArea
function Shape:printArea ()
    print("面积为 ",self.area)
end

-- 创建对象
myshape = Shape:new(nil,10)
myshape:printArea()

Square = Shape:new()

-- 派生类方法 new
function Square:new (o,side)
    o = o or Shape:new(o,side)
    setmetatable(o, self)
    self.__index = self
    return o
end

-- 派生类方法 printArea
function Square:printArea ()
    print("正方形面积为 ",self.area)
end

-- 创建对象
mysquare = Square:new(nil,10)
mysquare:printArea()

Rectangle = Shape:new()
-- 派生类方法 new
function Rectangle:new (o,length,breadth)
    o = o or Shape:new(o)
    setmetatable(o, self)
    self.__index = self
    self.area = length * breadth
    return o
end

-- 派生类方法 printArea
function Rectangle:printArea ()
    print("矩形面积为 ",self.area)
end

-- 创建对象
myrectangle = Rectangle:new(nil,10,20)
myrectangle:printArea()
```
执行以上代码，输出结果为：
```
面积为     100
正方形面积为     100
矩形面积为     200
```
# <span style="color:#039BE5;">总结</span>

刚刚学习了如何在lua中通过table模拟面向对象编程。Lua中的继承虽然可以通过metetable模拟出来，但是不推荐用，只模拟最基本的对象大部分时间够用了。

