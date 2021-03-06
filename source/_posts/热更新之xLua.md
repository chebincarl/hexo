---
layout: title
title: 热更新之xLua
date: 2019-06-22 09:36:12
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.Lua有哪两种作用？
2.如何安装xLua？

<!--more-->

Unity热更新有两大流派，C#派和lua派，那lua派是啥样的呢？

lua是一门历史悠久的脚本语言，从端游那个年代就被广泛应用在游戏开发中，所以到了现在的手游时代，有很多团队也让lua技术再次发展了起来。

# <span style="color:#039BE5;">Lua语言</span>

Lua 是一个小巧的脚本语言，于1993年开发。

Lua设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。Lua由标准C编写而成，几乎在所有操作系统和平台上都可以编译运行。Lua并没有提供强大的库，因为它的定位就是作为嵌入的脚本语言，所以Lua不适合开发独立应用程序。Lua有一个同时进行的JIT项目，提供在特定平台上的即时编译功能。

Lua脚本可以很容易的被<span style="color:red">C/C++代码调用，也可以反过来调用C/C++的函数</span>，这使得Lua在应用程序中可以被广泛应用，特别是在端游阶段，游戏几乎都是使用C/C++开发的，lua给游戏开发带来了极大的便利性。

不仅仅作为扩展脚本，lua也可以作为普通的<span style="color:red">配置文件</span>，代替XML、JSON、ini等文件格式，并且很容易理解和维护。Lua由标准C编写而成，代码简洁优美，几乎在所有操作系统和平台上都可以编译运行。一个完整的Lua解释器不过200k，在目前所有脚本引擎中，Lua的速度是最快的。这一切都决定了Lua是作为嵌入式脚本的最佳选择。

Lua之所以在游戏开发中这么受欢迎，也是因为它的优点：(1)语言优美、轻巧 (2)性能优良、速度快 (3)可扩展性强。

# <span style="color:#039BE5;">Unity与Lua</span>

Unity中的Lua也一直在发展，目前Unity中比较流行的几个Lua框架有：

* xLua
* uLua
* sLua

其中xLua是腾讯团队开发维护的一个框架，广泛应用在腾讯系的手游中，也是目前最受认可以及使用比较多的一个框架。

相对于我们之前学过的ILRuntime呢，lua也有一些优势和弱势。

Lua的优势：
* 技术更为成熟
* 成功的商业案例更多

Lua的劣势：
* 需要开发团队掌握Lua
* 可能需要维护C#和lua两套代码

后面我们会以目前最为主流的xLua来进行学习。

# <span style="color:#039BE5;">xLua<span>

xLua为Unity、.Net、Mono等C#环境增加Lua脚本编程的能力，借助xLua，这些Lua代码可以方便的和C#相互调用。

** <span style="color:red;">xLua的突破<span> **

xLua在功能、性能、易用性都有不少突破，这几方面分别最具代表性的是：

* 可以运行时把C#实现（方法，操作符，属性，事件等等）替换成lua实现，也就是热补丁功能；
* 出色的GC优化，自定义struct，枚举在Lua和C#间传递无C# gc alloc；
* 编辑器下无需生成代码，开发更轻量；


## <span style="color:#00ACC1;">下载</span>

下载地址为：[https://github.com/Tencent/xLua/releases](https://github.com/Tencent/xLua/releases)

可以选择最新的zip下载：

{% asset_img 1.png %}

上面图中可以看到xLua有几个不同的版本，有什么区别呢？

xLua有两个版本，分别集成了lua5.3和luajit，一个项目只能选择其一。这两个版本C#代码是一样的，不同的是Plugins部分。

&emsp;&emsp;xlua：此版本集成的是lua5.3，lua5.3的特性更丰富些，比如支持原生64位整数，支持苹果bitcode，支持utf8等。出现问题因为是纯c代码，也好定位。比起luajit，lua对安装包的影响也更小。

&emsp;&emsp;xlua_luajit：此版本集成的是luajit，luajit胜在性能，如果其jit不出问题的话，可以比lua高一个数量级。目前luajit作者不打算维护luajit，在找人接替其维护，后续发展不太明朗。

&emsp;&emsp;xlua_general：用于通用的.Net或Mono环境，可以在非Unity环境中使用。

目前lua53版本使用较多，所以xLua工程Plugins目录下默认配套是lua53版本，我们后面学习中也会使用这个版本。

## <span style="color:#00ACC1;">安装</span>

安装非常简单，新建一个Unity工程。

打开zip包，你会看到一个Assets目录，这目录就对应Unity工程的Assets目录，保持这目录结构放到你的Unity工程。

{% asset_img 2.png %}
<center><font color="gray">拖到空白处覆盖现在的Assets目录</font></center>
