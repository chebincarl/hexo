---
layout: title
title: xLua热更新2之Lua调用C#
date: 2019-06-24 23:03:31
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.如何创建一个空表？

<!--more-->

之前学习了C#调用lua，更多的时候，都是调用<span style="color:red">DoString("require 'byfile'")</span>然后放到lua代码中去处理了，那就意味着会有更多的lua调用C#的情况。

# <span style="color:#039BE5;">Lua调用C</span>
## <span style="color:#EF7060;">创建C#对象</span>

在C#这样new一个对象：
```cs
var newGameObj = new UnityEngine.GameObject();
```
对应到Lua是这样：
```lua
local newGameObj = CS.UnityEngine.GameObject()
```
基本上是类似的，区别是：

* lua里没有new关键字；
* 所有C#相关的都放到CS下，包括构造函数，静态成员属性、方法；

如果有多个构造函数呢？xlua支持重载，比如你要调用GameObject的带一个string参数的构造函数，这么写：
```lua
local newGameObj2 = CS.UnityEngine.GameObject('helloworld')
```
## <span style="color:#EF7060;">访问C#静态属性，方法</span>
** 读静态属性 **

```cs
CS.UnityEngine.Time.deltaTime
```
** 写静态属性 **

```cs
CS.UnityEngine.Time.timeScale = 0.5
```
** 调用静态方法 **

```cs
CS.UnityEngine.GameObject.Find('helloworld')
```
小技巧：如果需要经常访问的类，可以先用局部变量引用后访问，除了减少敲代码的时间，还能提高性能：
```lua
local GameObject = CS.UnityEngine.GameObject
GameObject.Find('helloworld')
```
## <span style="color:#EF7060;">访问C#成员属性，方法</span>
** 读成员属性 **

```cs
testobj.DMF
```
** 写成员属性 **

```cs
testobj.DMF = 1024
```
** 调用成员方法 **

注意：调用成员方法，第一个参数需要传该对象，建议用冒号语法糖，如下
```cs
testobj:DMFunc()
```
静态方法直接用点即可，成员方法需要用冒号语法糖。

** 父类属性，方法 **

xlua支持（通过派生类）访问基类的静态属性，静态方法，（通过派生类实例）访问基类的成员属性，成员方法
```lua
print(DerivedClass.BSF)--读基类静态属性
DerivedClass.BSF = 2048--写基类静态属性
DerivedClass.BSFunc();--基类静态方法
print(testobj.BMF)--读基类成员属性
testobj.BMF = 4096--写基类成员属性
testobj:BMFunc()--基类方法调用
```
** 参数的输入输出属性（out，ref） **

Lua调用测的参数处理规则：C#的普通参数算一个输入形参，ref修饰的算一个输入形参，out不算，然后从左往右对应lua 调用测的实参列表；

Lua调用测的返回值处理规则：C#函数的返回值（如果有的话）算一个返回值，out算一个返回值，ref算一个返回值，然后从左往右对应lua的多返回值。

比如C#的方法如下：
```cs
public double ComplexFunc(Param1 p1, ref int p2, out string p3, Action luafunc, out Action csfunc)
{
    Debug.Log("P1 = {x=" + p1.x + ",y=" + p1.y + "},p2 = " + p2);
    luafunc();
    p2 = p2 * p1.x;
    p3 = "hello " + p1.y;
    csfunc = () =>
    {
        Debug.Log("csharp callback invoked!");
    };
    return 1.23;
}
```
lua中对应的调用方法为：
```lua
local ret, p2, p3, csfunc = testobj:ComplexFunc({x=3, y = 'john'}, 100, function()
   print('i am lua callback')
end)
print('ComplexFunc ret:', ret, p2, p3, csfunc)
```
** 重载方法 **

直接通过不同的参数类型进行重载函数的访问，例如：
```lua
testobj:TestFunc(100)
testobj:TestFunc('hello')
```
将分别访问整数参数的TestFunc和字符串参数的TestFunc。

注意：xlua只一定程度上支持重载函数的调用，因为lua的类型远远不如C#丰富，存在一对多的情况，比如C#的int，float，double都对应于lua的number，上面的例子中TestFunc如果有这些重载参数，第一行将无法区分开来，只能调用到其中一个（生成代码中排前面的那个）

** 操作符 **

支持的操作符有：+，-，\*，/，==，一元-，<，<=， %，[]

比如C#中如下代码：
```cs
public static DerivedClass operator +(DerivedClass a, DerivedClass b)
{
    DerivedClass ret = new DerivedClass();
    ret.DMF = a.DMF + b.DMF;
    return ret;
}
```
lua中支持的操作符如下：
```lua
print('(testobj + testobj2).DMF = ', (testobj + testobj2).DMF)
```
** 参数带默认值的方法 **

和C#调用有默认值参数的函数一样，如果所给的实参少于形参，则会用默认值补上。

C#代码：
```cs
public void DefaultValueFunc(int a = 100, string b = "cccc", string c = null)
{
    UnityEngine.Debug.Log("DefaultValueFunc: a=" + a + ",b=" + b + ",c=" + c);
}
```
lua代码：
```lua
testobj:DefaultValueFunc(1)
testobj:DefaultValueFunc(3, 'hello', 'john')
```
** 可变参数方法 **

对于C#的如下方法：
```cs
void VariableParamsFunc(int a, params string[] strs)
```
可以在lua里头这样调用：
```lua
testobj:VariableParamsFunc(5, 'hello', 'john')
```
** 使用Extension methods **

在C#里定义了，lua里就能直接使用。

C#代码：
```cs
[LuaCallCSharp]
public static class DerivedClassExtensions
{
    public static int GetSomeData(this DerivedClass obj)
    {
        Debug.Log("GetSomeData ret = " + obj.DMF);
        return obj.DMF;
    }
    public static int GetSomeBaseData(this BaseClass obj)
    {
        Debug.Log("GetSomeBaseData ret = " + obj.BMF);
        return obj.BMF;
    }
    public static void GenericMethodOfString(this DerivedClass obj)
    {
        obj.GenericMethod<string>();
    }
}
```
lua代码：
```lua
print(testobj:GetSomeData()) 
print(testobj:GetSomeBaseData()) --访问基类的Extension methods
testobj:GenericMethodOfString()  --通过Extension methods实现访问泛化方法
```
** 泛型（模版）方法 **

不直接支持，可以通过Extension methods功能进行封装后调用。如上面代码中<span style="color:red">GenericMethodOfString</span>方法所示。

** 枚举类型 **

枚举值就像枚举类型下的静态属性一样。
```lua
testobj:EnumTestFunc(CS.Tutorial.TestEnum.E1)
```
上面的EnumTestFunc函数参数是Tutorial.TestEnum类型的。

枚举类支持__CastFrom方法，可以实现从一个整数或者字符串到枚举值的转换，例如：
```cs
CS.Tutorial.TestEnum.__CastFrom(1)
CS.Tutorial.TestEnum.__CastFrom('E1')
```
** delegate使用（调用，+，-） **

C#的delegate调用：和调用普通lua函数一样，注意此处调用是用点.

+操作符：对应C#的+操作符，把两个调用串成一个调用链，右操作数可以是同类型的C# delegate或者是lua函数。

-操作符：和+相反，把一个delegate从调用链中移除。

> Ps：delegate属性可以用一个luafunction来赋值。

```lua
testobj.TestDelegate('hello') --直接调用
local function lua_delegate(str)
    print('TestDelegate in lua:', str)
end
testobj.TestDelegate = lua_delegate + testobj.TestDelegate --combine，这里演示的是C#delegate作为右值，左值也支持
testobj.TestDelegate('hello')
testobj.TestDelegate = testobj.TestDelegate - lua_delegate --remove
testobj.TestDelegate('hello')
```
** event **

比如testobj里头有个事件定义是这样：<span style="color:red">public event Action TestEvent;</span>

增加事件回调
```lua
testobj:TestEvent('+', lua_event_callback)
```
移除事件回调
```lua
testobj:TestEvent('-', lua_event_callback)
```
** 64位整数支持 **

Lua53版本64位整数（long，ulong）映射到原生的64未整数，而luajit版本，相当于lua5.1的标准，本身不支持64位，xlua做了个64位支持的扩展库，C#的long和ulong都将映射到userdata：

* 支持在lua里头进行64位的运算，比较，打印
* 支持和lua number的运算，比较
* 要注意的是，在64扩展库中，实际上只有int64，ulong也会先强转成long再传递到lua，而对ulong的一些运算、比较，xLua采取和java一样的支持方式，提供一组API，详情请看API文档：[https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/XLua_API.md#%E6%97%A0%E7%AC%A6%E5%8F%B764%E4%BD%8D%E6%94%AF%E6%8C%81](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/XLua_API.md#%E6%97%A0%E7%AC%A6%E5%8F%B764%E4%BD%8D%E6%94%AF%E6%8C%81)。

> Lua53 vs Luajit 的区别我们在218学习过。

** C#复杂类型和table的自动转换 **

对于一个有无参构造函数的C#复杂类型，在lua侧可以直接用一个table来代替，该table对应复杂类型的public字段有相应字段即可，支持函数参数传递，属性赋值等，例如： C#下B结构体（也支持class）定义如下：
```cs
public struct A
{
    public int a;
}

public struct B
{
    public A b;
    public double c;
}
```
某个类有成员函数如下：
```cs
void Foo(B b)
```
在lua可以这么调用
```lua
obj:Foo({b = {a = 100}, c = 200})
```
** 获取类型（相当于C#的typeof） **

比如要获取UnityEngine.ParticleSystem类的Type信息，可以这样
```lua
typeof(CS.UnityEngine.ParticleSystem)
```
** 强制转换 **

lua没类型，所以不会有强类型语言的强制转换。

但是某些情况下，比如：告诉xlua要用指定的生成代码去调用一个对象，有的时候第三方库对外暴露的是一个interface或者抽象类，实现类是隐藏的，这样我们无法对实现类进行代码生成。该实现类将会被xlua识别为未生成代码而用反射来访问，如果这个调用是很频繁的话还是很影响性能的，这时我们就可以把这个interface或者抽象类加到生成代码，然后指定用该生成代码来访问：
```lua
cast(calc, typeof(CS.Tutorial.Calc))
```
上面就是指定用CS.Tutorial.Calc的生成代码来访问calc对象。

# <span style="color:#039BE5;">总结</span>

使用lua访问C#代码的变化不大，需要注意里面几点：

* 如果需要经常访问的类，可以先用局部变量引用后访问，除了减少敲代码的时间，还能提高性能
* xlua只一定程度上支持重载函数的调用，因为lua的类型远远不如C#丰富，存在一对多的情况，比如C#的int，float，double都对应于lua的number，无法区分
* 委托调用的时候需要用<span style="color:red">.</span>