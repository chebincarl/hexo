---
layout: title
title: xLua热更新3之生成代码
date: 2019-06-24 23:03:45
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.如何创建一个空表？

<!--more-->

之前提到了生成代码的问题，是不是对这个概念很陌生？生成代码从广义上来说就是通过程序来生成代码。在xLua中也是如此，下面来看看在xLua中具体是什么。

# <span style="color:#039BE5;">xLua生成代码</span>
## <span style="color:#EF7060;">什么是生成代码？</span>
生成代码是xLua支持的lua和C#间交互技术之一，这种技术通过生成两者间的适配代码来实现交互，性能较好，是<span style="color:red"> 推荐的方式</span>。xLua可以自动生成配置好的代码，后面会有介绍。

另一种交互技术是反射，这种方式对安装包的影响更少，可以在性能要求不高或者对安装包大小很敏感的场景下使用。

## <span style="color:#EF7060;">应该什么时候生成代码？</span>
开发期不建议生成代码，可以避免很多由于不一致导致的编译失败，以及生成代码本身的编译等待。

发布手机版本前必须执行生成代码，建议做成自动化的流程。

做性能调优，性能测试前必须执行生成代码，因为生成和不生成性能的区别还是很大的。

## <span style="color:#EF7060;">如何生成代码？</span>
生成代码的过程是自动化的，xLua提供了相关的脚本进行生成。

入口是下图的菜单栏：

{% asset_img 1.png %}

但是xLua需要首先知道生成哪些代码，这就涉及到xLua的配置了。

## <span style="color:#EF7060;">xLua的配置</span>
xLua所有的配置都支持三种方式：打标签；静态列表；动态列表。

配置有两必须两建议：

* 列表方式均必须是static的字段/属性
* 列表方式均必须放到一个static类
* 建议不用标签方式
* 建议列表方式配置放Editor目录（如果是Hotfix配置【后面会讲】，而且类位于Assembly-CSharp.dll之外的其它dll，必须放Editor目录）

** 打标签 **

xLua用白名单来指明生成哪些代码，而白名单通过attribute来配置，比如你想从lua调用c#的某个类，希望生成适配代码，你可以为这个类型打一个LuaCallCSharp标签：
```cs
[LuaCallCSharp]
public class A
{
}
```
<span style="color:red">该方式方便，但在il2cpp下会增加不少的代码量，不建议使用。</span>

** 静态列表 **

有时我们无法直接给一个类型打标签，比如系统API，没源码的库，或者实例化的泛化类型，这时你可以在一个静态类里声明一个静态字段，该字段的类型除BlackList和AdditionalProperties之外只要实现了IEnumerable就可以了（这两个例外后面具体会说），然后为这字段加上标签：
```cs
[LuaCallCSharp]
public static List<Type> mymodule_lua_call_cs_list = new List<Type>()
{
    typeof(GameObject),
    typeof(Dictionary<string, int>),
};
```
这个字段需要放到一个<span style="color:red">静态类</span>里头，建议放到<span style="color:red">Editor</span>目录 。

** 动态列表 **

声明一个静态属性，打上相应的[Hotfix]标签即可。
```cs
[Hotfix]
public static List<Type> by_property
{
    get
    {
        return (from type in Assembly.Load("Assembly-CSharp").GetTypes()
                where type.Namespace == "XXXX"
                select type).ToList();
    }
}
```
Getter是代码，你可以实现很多效果，比如按名字空间配置，按程序集配置等等。

这个属性需要放到一个<span style="color:red">静态类</span>里头，建议放到<span style="color:red">Editor</span>目录 。

## <span style="color:#EF7060;">配置Attribute</span>

** XLua.LuaCallCSharp **
```cs
[LuaCallCSharp]
public class A
{
}
```
一个C#类型加了这个配置，xLua会生成这个类型的适配代码（包括构造该类型实例，访问其成员属性、方法，静态属性、方法），否则将会尝试用性能较低的反射方式来访问。

一个类型的扩展方法（Extension Methods）加了这配置，也会生成适配代码并追加到被扩展类型的成员方法上。

xLua只会生成加了该配置的类型，不会自动生成其父类的适配代码，当访问子类对象的父类方法，如果该父类加了LuaCallCSharp配置，则执行父类的适配代码，否则会尝试用反射来访问。

反射访问除了性能不佳之外，在il2cpp下还有可能因为代码剪裁而导致无法访问，后者可以通过下面介绍的ReflectionUse标签来避免。

## <span style="color:#EF7060;">XLua.ReflectionUse</span>
```cs
[ReflectionUse]
public class A
{
}
```
一个C#类型类型加了这个配置，xLua会生成link.xml阻止il2cpp的代码剪裁。

对于扩展方法，必须加上LuaCallCSharp或者ReflectionUse才可以被访问到。

建议所有要在Lua访问的类型，要么加LuaCallCSharp，要么加上ReflectionUse，这才能够保证在各平台都能正常运行。

## <span style="color:#EF7060;">XLua.BlackList</span>
如果你不要生成一个类型的一些成员的适配代码，你可以通过这个配置来实现。

标签方式比较简单，对应的成员上加就可以了。

由于考虑到有可能需要把重载函数的其中一个重载列入黑名单，配置方式比较复杂，类型是List<list>，对于每个成员，在第一层List有一个条目，第二层List是个string的列表，第一个string是类型的全路径名，第二个string是成员名，如果成员是一个方法，还需要从第三个string开始，把其参数的类型全路径全列出来。

例如下面是对GameObject的一个属性以及FileInfo的一个方法列入黑名单：
```cs
[BlackList]
public static List<List<string>> BlackList = new List<List<string>>()  {
    new List<string>(){"UnityEngine.GameObject", "networkView"},
    new List<string>(){"System.IO.FileInfo", "GetAccessControl", "System.Security.AccessControl.AccessControlSections"},
};
```
## <span style="color:#EF7060;">XLua.DoNotGen</span>

指明一个类里的部分函数、字段、属性不生成代码，通过反射访问。

只能标注Dictionary<type, list>的field或者property。key指明的是生效的类，value是一个列表，配置的是不生成代码的函数、字段、属性的名字。

和ReflectionUse的区别是：

* ReflectionUse指明的是整个类；
* 当第一次访问一个函数（字段、属性）时，ReflectionUse会把整个类都wrap，而DoNotGen只wrap该函数（字段、属性），换句话DoNotGen更lazy一些；

和BlackList的区别是：

* BlackList配了里面指定的类型就不能在lua中用了；
* BlackList能指明某重载函数，DoNotGen不能。

## <span style="color:#EF7060;">XLua.CSharpCallLua</span>
如果希望把一个lua函数适配到一个C# delegate（一类是C#侧各种回调：UI事件，delegate参数，比如List:ForEach；另外一类场景是通过LuaTable的Get函数指明一个lua函数绑定到一个delegate）。或者把一个lua table适配到一个C# interface，该delegate或者interface需要加上该配置。

## <span style="color:#EF7060;">XLua.GCOptimize</span>
一个C#纯值类型（注：指的是一个只包含值类型的struct，可以嵌套其它只包含值类型的struct）或者C#枚举值加上了这个配置。xLua会为该类型生成gc优化代码，效果是该值类型在lua和c#间传递不产生（C#）gc alloc，该类型的数组访问也不产生gc。

除枚举之外，包含无参构造函数的复杂类型，都会生成lua table到该类型，以及改类型的一维数组的转换代码，这将会优化这个转换的性能，包括更少的gc alloc。

## <span style="color:#EF7060;">XLua.AdditionalProperties</span>
这个是GCOptimize的扩展配置，有的时候，一些struct喜欢把field写成私有的，通过property来访问field，这时就需要用到该配置（默认情况下GCOptimize只对public的field打解包）。

标签方式比较简单，配置方式复杂一点，要求是Dictionary<type, list>类型，Dictionary的Key是要生效的类型，Value是属性名列表。

## <span style="color:#EF7060;">下面是生成期配置，必须放到Editor目录下</span>
** CSObjectWrapEditor.GenPath **

```cs
[CSObjectWrapEditor.GenPath]
public static string Path = Application.streamingAssetsPath;
```
配置生成代码的放置路径，类型是string。如果不配置默认生成在"Assets/XLua/Gen/"下。

** CSObjectWrapEditor.GenCodeMenu **

```cs
[CSObjectWrapEditor.GenCodeMenu]
public static void Method(){
}
```
该配置用于生成引擎的二次开发，一个无参数函数加了这个标签，在执行"XLua/Generate Code"菜单时会触发这个函数的调用。

# 总结

xLua也给出了一份常用的xLua的配置，可以在这个基础上根据自己的项目做修改。地址是：[https://github.com/Tencent/xLua/blob/master/Assets/XLua/Editor/ExampleConfig.cs](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Editor/ExampleConfig.cs)