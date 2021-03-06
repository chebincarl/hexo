---
layout: title
title: xLua热更新1之C#调用Lua
date: 2019-06-24 23:03:00
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.如何创建一个空表？

<!--more-->

前面学习了lua编程，使用xLua热更新的时候，很多代码都是使用lua来编写的。但是到目前为止我们学习的内容都还很孤立，C#代码是C#代码，lua代码是lua代码，那么他们之间怎么联系在一起呢？

# <span style="color:#039BE5;">C#访问Lua</span>

有时候，需要在C#中访问Lua中的数据或者函数，那怎么做呢？

下面例子中用到的lua代码如下：
```lua
string script = @"
    a = 1
    b = 'hello world'
    c = true
    d = {
       f1 = 12, f2 = 34, 
       1, 2, 3,
       add = function(self, a, b) 
          print('d.add called')
          return a + b 
       end
    }
    function e()
        print('i am e')
    end
    function f(a, b)
        print('a', a, 'b', b)
        return 1, {f1 = 1024}
    end

    function ret_e()
        print('ret_e called')
        return e
    end
";
```

## <span style="color:#EF7060;">访问全局变量</span>

LuaEnv.Global中有个模版Get方法，可指定返回的类型。获取一个全局基本数据类型使用这个方法即可。
```lua
luaenv = new LuaEnv();
luaenv.DoString(script);
Debug.Log("_G.a = " + luaenv.Global.Get<int>("a"));
Debug.Log("_G.b = " + luaenv.Global.Get<string>("b"));
Debug.Log("_G.c = " + luaenv.Global.Get<bool>("c"));
```
那如果访问非基本数据类型，比如table怎么办呢？

## <span style="color:#EF7060;">访问table</span>

访问table需要将lua table转换成C#中的数据结构，有几种方式：

* 映射到普通class或struct
* 【推荐】映射到一个interface
* 更轻量级的by value方式：映射到Dictionary<>，List<>
* 另外一种by ref方式：映射到LuaTable类

** 1.映射到class或struct **

定义一个class或struct，有对应于table的字段的public属性，而且有无参数构造函数即可。

代码如下：
```cs
public class DClass
{
    public int f1;
    public int f2;
}

DClass d = luaenv.Global.Get<DClass>("d");//映射到有对应字段的class，by value
Debug.Log("_G.d = {f1=" + d.f1 + ", f2=" + d.f2 + "}");
```
table的属性可以多于或者少于class的属性。可以嵌套其它复杂类型。

** <span style="color:red">要注意的是，这个过程是值拷贝，如果class比较复杂代价会比较大。而且修改class的字段值不会同步到table，反过来也不会。</span> **

可以通过把类型加到GCOptimize生成降低这个功能的开销。

** GCOptimize **

一个C#纯值类型（注：指的是一个只包含值类型的struct，可以嵌套其它只包含值类型的struct）或者C#枚举值加上了这个配置，xLua会为该类型生成gc优化代码，效果是该值类型在lua和c#间传递不产生（C#）gc alloc，该类型的数组访问也不产生gc。

除枚举之外，包含无参构造函数的复杂类型，都会生成lua table到该类型，以及生成该类型的一维数组的转换代码，这将会优化这个转换的性能，包括更少的gc alloc。

示例代码如下：
```cs
[GCOptimize]
public struct DClass
{
    public int f1;
    public int f2;
}
```
** 2. 映射到interface【推荐】 **

这种方式依赖于生成代码（如果没生成代码会抛InvalidCastException异常），代码生成器会生成这个interface的实例，如果get一个属性，生成代码会get对应的table字段，如果set属性也会设置对应的字段。甚至可以通过interface的方法访问lua的函数。
```cs
[CSharpCallLua] // 最简单的一种添加到生成代码的方式
public interface ItfD
{
    int f1 { get; set; }
    int f2 { get; set; }
    int add(int a, int b);
}

ItfD d3 = luaenv.Global.Get<ItfD>("d"); //映射到interface实例，引用传递，这个要求interface加到生成列表，否则会返回null，建议使用该用法
d3.f2 = 1000;
Debug.Log("_G.d = {f1=" + d3.f1 + ", f2=" + d3.f2 + "}");
Debug.Log("_G.d:add(1, 2)=" + d3.add(1, 2));
```
** 什么是生成代码？ **

xLua支持的lua和C#间交互技术之一，这种技术通过生成两者间的适配代码来实现交互，性能较好，是推荐的方式。

另一种交互技术是反射，这种方式对安装包的影响更少，可以在性能要求不高或者对安装包大小很敏感的场景下使用。

xLua相对于其他lua解决方案的一个很大优势就是：** <span style="color:red">编辑器下无需生成代码，开发更轻量。</span> **

应该什么时候生成代码？如何生成代码呢？这个是一个比较大的话题，我们后面再详细讨论。

** 3.映射到Dictionary<>，List<> **

不想定义class或者interface的话，可以考虑用这个，前提table下key和value的类型都是一致的。这种也是值拷贝的方式。
```cs
Dictionary<string, double> d1 = luaenv.Global.Get<Dictionary<string, double>>("d");//映射到Dictionary<string, double>，by value
Debug.Log("_G.d = {f1=" + d1["f1"] + ", f2=" + d1["f2"] + "}, d.Count=" + d1.Count);

List<double> d2 = luaenv.Global.Get<List<double>>("d"); //映射到List<double>，by value
Debug.Log("_G.d.len = " + d2.Count);
```

** 4.映射到LuaTable类 **

这种方式好处是不需要生成代码，但也有一些问题，比如慢，比方式2要慢一个数量级，比如没有类型检查。这种方式是引用传递。
```cs
LuaTable d4 = luaenv.Global.Get<LuaTable>("d");//映射到LuaTable，by ref
Debug.Log("_G.d = {f1=" + d4.Get<int>("f1") + ", f2=" + d4.Get<int>("f2") + "}");
```
## <span style="color:#EF7060;">访问全局function</span>

访问全局的function仍然是用Get方法，不同的是类型映射。

** 1.映射到delegate【建议】 **

这种是建议的方式，性能好很多，而且类型安全。缺点是要生成代码（如果没生成代码会抛InvalidCastException异常）。

delegate要怎样声明呢？ 对于function的每个参数就声明一个输入类型的参数。 多返回值要怎么处理？从左往右映射到c#的输出参数，输出参数包括返回值，out参数，ref参数。

参数、返回值类型支持哪些呢？都支持，各种复杂类型，out，ref修饰的，甚至可以返回另外一个delegate。

delegate的使用就更简单了，直接像个函数那样用就可以了。
```cs
Action e = luaenv.Global.Get<Action>("e");//映射到一个delgate，要求delegate加到生成列表，否则返回null，建议用法
e();

FDelegate f = luaenv.Global.Get<FDelegate>("f");
DClass d_ret;
int f_ret = f(100, "John", out d_ret);//lua的多返回值映射：从左往右映射到c#的输出参数，输出参数包括返回值，out参数，ref参数
Debug.Log("ret.d = {f1=" + d_ret.f1 + ", f2=" + d_ret.f2 + "}, ret=" + f_ret);

GetE ret_e = luaenv.Global.Get<GetE>("ret_e");//delegate可以返回更复杂的类型，甚至是另外一个delegate
e = ret_e();
e();
```
** 2.映射到LuaFunction **

这种方式的优缺点刚好和第一种相反。 使用也简单，LuaFunction上有个变参的Call函数，可以传任意类型，任意个数的参数，返回值是object的数组，对应于lua的多返回值。
```lua
LuaFunction d_e = luaenv.Global.Get<LuaFunction>("e");
d_e.Call();
```

# <span style="color:#039BE5;">总结</span>

访问lua全局数据时，特别是table以及function，代价比较大，建议尽量少做，比如在初始化时把要调用的lua function获取一次（映射到delegate）后，保存下来，后续直接调用该delegate即可，table也类似。