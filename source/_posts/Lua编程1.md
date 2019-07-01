---
layout: title
title: Lua编程1
date: 2019-06-23 15:35:12
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.LuaEnv luaenv = new LuaEnv();是什么意思？
2.luaenv.DoString("CS.UnityEngine.Debug.Log('hello world')");是什么意思？
3.luaenv.DoString("require 'byfile'");是什么意思？
4.luaenv.dispose是什么意思？
5.以下划线(\_)开头，后面紧随多个大写字母（\_VERSION）的变量有什么含义？
6.C#中使用哪个命名空间？
7.建议的加载Lua脚本方式是？
8.直接访问未初始化的全局变量会报错吗？

<!--more-->

由于这节课我们会用到xLua的Example和Tutorial，在开始之前需要你将xLua的整个Github源码下载或者Clone下来。[https://github.com/Tencent/xLua](https://github.com/Tencent/xLua)

下载以后，使用Unity打开下载的源码，xLua的示例代码在Assets\XLua\Examples中，xLua的教程在Assets\XLua\Tutorial中。

# <span style="color:#039BE5;">Hello World</span>
学习一切新事物从Hello World入手。那么xLua的Hello World是什么呢？

该例子位于Assets\XLua\Examples\01_Helloworld。
```cs
using UnityEngine;
using XLua;

public class Helloworld : MonoBehaviour 
{
    void Start () 
    {
        LuaEnv luaenv = new LuaEnv();
        luaenv.DoString("CS.UnityEngine.Debug.Log('hello world')");
        luaenv.Dispose();
    }
}
```
上面的代码，就是执行了一句lua代码<span style="color:red">CS.UnityEngine.Debug.Log('hello world')</span>，来打印出hello world。

{% asset_img 1.png %}
<center><font color="gray">执行后Console窗口</font></center>

我们来逐行分析一下Start中的代码：
第一行<span style="color:red">LuaEnv luaenv = new LuaEnv();</span>，创建了一个lua环境，类似一个运行时环境，用来解析、执行lua代码。

第2行<span style="color:red">luaenv.DoString();</span>是用于执行lua代码字符串。

第3行<span style="color:red">luaenv.Dispose();</span>用于释放这个lua环境。

到这呢，我们已经学会了整个环境如何搭建以及如何执行lua代码了。但是通过字符串执行代码总归还是不够方便，我们下面再看看如何加载文件并执行文件中的lua代码。

# <span style="color:#039BE5;">从文件加载Lua文件</span>
从文件加载Lua代码文件，用lua的<span style="color:red">require</span>函数即可，比如：

完整代码见XLua\Tutorial\LoadLuaScript\ByFile

{% asset_img 2.png %}

```cs
using UnityEngine;
using System.Collections;
using XLua;

namespace Tutorial
{
    public class ByFile : MonoBehaviour
    {
        LuaEnv luaenv = null;
        // Use this for initialization
        void Start()
        {
            luaenv = new LuaEnv();
            luaenv.DoString("require 'byfile'");
        }

        // Update is called once per frame
        void Update()
        {
            if (luaenv != null)
            {
                luaenv.Tick();
            }
        }

        void OnDestroy()
        {
            luaenv.Dispose();
        }
    }
}
```
lua中的require有些像C#中的using。实际上是调一个个的加载器loader，来加载一个一个模块/代码文件。<span style="color:red">与using不同的是，require加载进来后会将代码执行一次</span>。

<span style="color:red">xlua除了lua原生的loader外，还添加了从Resource加载的loader</span>，所以上面的代码可以加载Resources目录下的byfile.lua.txt文件并执行。

需要注意的是因为<span style="color:red">Resource只支持有限的后缀</span>，放Resources下的lua文件得加上txt后缀。

建议的加载Lua脚本方式是：整个程序就一个DoString("require 'main'")，然后在main.lua加载其它脚本（类似lua脚本的命令行执行：lua main.lua）。

此外很多时候热更新肯定是需要从服务器上下载lua代码执行，这个后面再说。


> 下面开始学习lua的语法。后面学习中，如果没有特殊说明，代码会写在上面的byfile.lua.txt文件中，并使用XLua\Tutorial\LoadLuaScript\ByFile场景运行测试。

# <span style="color:#039BE5;">Lua编程</span>
Lua是一门非常简洁的语言，也没有过多复杂的语法，学习起来很容易上手。

## <span style="color:#00ACC1;">代码基础</span>
### <span style="color:#EF7060;">语句</span>
Lua的多条语句之间并不要求任何分隔符，如C#语言的分号(;)，其中换行符也同样不能起到语句分隔的作用。因此下面的写法均是合法的。如：
```lua
a = 1
b = a * 2

a = 1;
b = a * 2;

a = 1; b = a * 2;
a = 1  b = a * 2
```

但是为了代码的可读性，通常我们一行只写一句代码。

### <span style="color:#EF7060;">标识符</span>
Lua 标示符用于定义变量，函数或开发者定义的项。标示符以一个字母 A 到 Z 或 a 到 z 或下划线 _ 开头，后加上0个或多个字母、下划线或数字（0到9）。

在Lua中还有一个特殊的规则，即以下划线(\_)开头，后面紧随多个大写字母(\_VERSION)，这些变量一般被Lua保留并用于特殊用途，因此我们在声明变量时需要尽量避免这样的声明方式，以免给后期的维护带来不必要的麻烦。

Lua是大小写敏感的，因此在 Lua 中 Gold与 gold 是两个不同的标示符。此外对于一些Lua保留关键字的使用要特别小心，如and。但是And和AND则不是Lua的保留字。

### <span style="color:#EF7060;">关键字</span>
以下列出了 Lua 的保留关键字。保留关键字不能作为常量或变量或其他用户自定义标示符：

and  break  do else  elseif end   false   for  function   if   in   local   nil   not   or   repeat   return   then   true   until  while 


### <span style="color:#EF7060;">注释</span>
Lua中的注释分为两种，一种是单行注释，如：
```lua
--This is a single line comment.
```
另外一种是多行注释，如：
```lua
--[[
  This is a multi-lines comment.
--]]
```
### <span style="color:#EF7060;">全局变量</span>
<span style="color:red">在默认情况下，变量总是认为是全局的。</span>

在Lua中全局变量不需要声明，直接赋值即可。如果直接访问未初始化的全局变量，Lua也不会报错，直接返回nil。如果不想再使用该全局变量，可直接将其置为nil。如：
```lua
print(b) -- nil
b = 10
print(b) -- 10
b = nil
print(b) -- nil
```
这样变量b就好像从没被使用过一样。换句话说, 当且仅当一个变量不等于nil时，这个变量即存在。

