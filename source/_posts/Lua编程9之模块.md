---
layout: title
title: Lua编程9之模块
date: 2019-06-23 23:28:00
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.如何创建一个空表？

<!--more-->

用Lua写代码不可能把所有代码都写一个文件里，不同文件之间如何引用呢？Lua中也有模块的机制，类似C#中封装的命名空间。

# <span style="color:#039BE5;">Lua 模块</span>
模块类似于一个封装库，从 Lua 5.1 开始，Lua 加入了标准的模块管理机制，可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。

Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。以下为创建自定义模块 module.lua，文件代码格式如下：
```lua
-- 文件名为 module.lua
-- 定义一个名为 module 的模块
module = {}

-- 定义一个常量
module.constant = "这是一个常量"

-- 定义一个函数
function module.func1()
    io.write("这是一个公有函数！\n")
end

local function func2()
    print("这是一个私有函数！")
end

function module.func3()
    func2()
end

return module
```
由上可知，模块的结构就是一个 table 的结构，因此可以像操作调用 table 里的元素那样来操作调用模块里的常量或函数。

上面的 func2 声明为程序块的局部变量，即表示一个私有函数，因此是不能从外部访问模块里的这个私有函数，必须通过模块里的公有函数来调用.

## <span style="color:#EF7060;">require 函数</span>
Lua提供了一个名为require的函数用来加载模块。要加载一个模块，只需要简单地调用就可以了。例如：
```lua
require("<模块名>")
```
或者
```lua
require "<模块名>"
```
执行 require 后会返回一个由模块常量或函数组成的 table，并且还会定义一个包含该 table 的全局变量。
```lua
-- test_module.lua 文件
-- module 模块为上文提到到 module.lua
require("module")

print(module.constant)

module.func3()
```
以上代码执行结果为：
```
这是一个常量
这是一个私有函数！
```
或者给加载的模块定义一个别名变量，方便调用：
```lua
-- test_module2.lua 文件
-- module 模块为上文提到到 module.lua
-- 别名变量 m
local m = require("module")

print(m.constant)

m.func3()
```
以上代码执行结果为：
```
这是一个常量
这是一个私有函数！
```
** <span style="color:red">标准Lua中的加载机制</span> **

对于自定义的模块，模块文件不是放在哪个文件目录都行，函数 require 有它自己的文件路径加载策略，它会尝试从 Lua 文件或 C 程序库中加载模块。

require 用于搜索 Lua 文件的路径是存放在全局变量 package.path 中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

在lua中可以使用<span style="color:red">print (package.path)</span>来查看当前的搜索路径。

我的打印出来是这样的：
```lua
C:\Program Files\Unity\Hub\Editor\2018.3.6f1\Editor\lua\?.lua;
C:\Program Files\Unity\Hub\Editor\2018.3.6f1\Editor\lua\?\init.lua;
C:\Program Files\Unity\Hub\Editor\2018.3.6f1\Editor\?.lua;
C:\Program Files\Unity\Hub\Editor\2018.3.6f1\Editor\?\init.lua;
C:\Program Files\Unity\Hub\Editor\2018.3.6f1\Editor\..\share\lua\5.3\?.lua;
C:\Program Files\Unity\Hub\Editor\2018.3.6f1\Editor\..\share\lua\5.3\?\init.lua;
.\?.lua;
.\?\init.lua
```
那么调用 require("module") 时就会尝试从搜索路径中从前到后的顺序去搜索lua文件。

如果找过目标文件，则会调用 package.loadfile 来加载模块。否则，就会去找 C 程序库。

C 程序库搜索的文件路径是从全局变量 package.cpath 获取，而这个变量则是通过环境变量 LUA_CPATH 来初始。

搜索的策略跟上面的一样，只不过现在换成搜索的是 so 或 dll 类型的文件。如果找得到，那么 require 就会通过 package.loadlib 来加载它。

## <span style="color:#EF7060;">xLua中的模块加载</span>
xLua中require实际上是调一个个的loader去加载，有一个成功就不再往下尝试，全失败则报文件找不到。

在xLua中，模块加载在Unity有些因地制宜的loader。目前xLua除了原生的loader外，还添加了从Resource和StreamingAssets（从StreamingAssets加载已被标记弃用，不建议使用）加载的loader，需要注意的是因为Resource只支持有限的后缀，放Resources下的lua文件得加上txt后缀，比如文件名需要命名成<span style="color:red">module.lua.txt</span>。

可以从xLua的LuaEnv.cs的第108行看到：
```lua
AddSearcher(StaticLuaCallbacks.LoadFromResource, 4);
AddSearcher(StaticLuaCallbacks.LoadFromStreamingAssetsPath, -1);
```
建议的加载Lua脚本方式是：整个程序就一个DoString("require 'main'")，然后在main.lua中require加载其它脚本。

要是我的Lua文件是下载回来的，或者某个自定义的文件格式里头解压出来，或者需要解密等等，怎么办？
xLua的自定义Loader可以满足这些需求。

** <span style="color:red">自定义Loader</span> **

在xLua加自定义loader是很简单的，只涉及到一个接口：
```lua
public delegate byte[] CustomLoader(ref string filepath);
public void LuaEnv.AddLoader(CustomLoader loader)
```
通过AddLoader可以注册个回调，该回调参数是字符串，lua代码里头调用require时，参数将会透传给回调，回调中就可以根据这个参数去加载指定文件，如果需要支持调试，需要把filepath修改为真实路径传出。

该回调返回值是一个byte数组，如果为空表示该loader找不到，否则则为lua文件的内容。 有了这个就简单了，所有你自己的流程都可以处理。比如：
```lua
luaenv.AddLoader((ref string filename) => {
     if (filename == "InMemory")
     {
         string script = "return {ccc = 9999}";
         return System.Text.Encoding.UTF8.GetBytes(script);
     }
     return null;
 });
luaenv.DoString("print('InMemory.ccc=', require('InMemory').ccc)");
```
完整示例见XLua\Tutorial\LoadLuaScript\Loader