---
layout: title
title: 'C#基础'
date: 2019-07-15 22:59:20
tags:
---
C#基础

<!--more-->

.net的本质其实是一个运行在操作系统上的**软件**。一般所说的.net指的是.net Framework框架。

安装路径：C:\Windows\Microsoft.NET

mono是一个开源跨平台的.net运行环境。

C#通过.net平台开发的应用程序，能不能运行在某个计算机上面，取决于该计算机是否安装了对应版本的.net Framework框架。

基于.net framework3.0开发的程序如果只使用了.net framework2.0的功能，也可以运行在.net framework2.0环境下。

Unity是基于.net framework3.5的。

{% asset_img 1.png %}

# VS

一个解决方案可以有多个项目

{% asset_img 2.png %}

{% asset_img 3.png %}

启动项目是粗体，可以右键“设为启动项目”，运行的时候就执行此项目。

解决方案（.sln）文件其实是一个文本文件，这个文件记录着解决方案管理的项目（项目名称与路径）
项目文件(.csproj)，也是一个文本文件，这个记录了项目里面的资源

注意：在VS里面，项目移除可以重新加载，项目里的资源被移除则是永久删除

# .Net编译原理

Main函数是程序的入口，也是程序的出口。

Net平台的重要组成部分

一、 重要组成部分：

{% asset_img 5.png %}    

1、FCL: 框架类库

我们在编程的时候，经常会用到一些功能，微软事先把这些功能写好（比如像控制台输出信息（例如Console.WriteLine()），产生一个随机数），封装在一个一个的类里面，然后把这些类放在.Net平台中，这些类的集合，我们就叫做框架类库（可以看做是功能的集合）。我们可以用各自的语法来调用
      
2、CLR：公共语言运行时

它是.Net程序运行的必备环境
如果.Net程序要在系统上运行，那么系统上面必须安装CLR
注意：CLR是无法单独安装的，它集成在.Net框架里面，将.Net框架安装在系统上，系统里面就有了CLR和FCL
      
二、如何获取.Net程序的可运行文件

这个文件能不能在电脑上运行，取决于电脑是否安装了对应的.Net框架
        
Net程序的编译原理  
     
1、C#源代码通过C#编译器，编译成程序集，程序集里面包含微软中间语言（MSIL）     

2、运行程序集的时候，CLR中的即时编译器（JIT,Just In Time）会将微软中间语言转换为本地平台对应的二进制指令，然后交给CPU去运行

{% asset_img 4.png %}    

# 托管代码和非托管代码

{% asset_img 6.png %}    

从 .NET Framework 4 开始，所有 .NET Framework 版本都是就地更新的，因此，在系统中只能存在一个 4.x 版本。 此外，某些版本的 Windows 操作系统上预装了特定版本的 .NET Framework。 这表示：
如果在计算机上已安装了更高的 4.x 版本，则无法安装以前的 4.x 版本。
如果操作系统预安装了特定的 .NET Framework 版本，则无法在同一计算机上安装以前的 4.x 版本。
如果你安装更高版本，则无需先卸载以前的版本。

.NET Framework 是软件开发平台（框架），C#是.NET框架中支持的其中一种语言。
C#语言的运行载体是.NET框架。

C#源文件要经过C#编译器编译成托管程序集元数据，就是exe程序或者dll库。

然后由.NET Framework编译器编译成机器代码，被操作系统识别。


C#是跟随.NET的开发工具Visual Studio一起发布的，在安装Visual Studio时是把.NET Framework框架一并安装的，所以C#的版本是跟随.NET Framework的，换句话说C#语言本身没有单独版本之说，和.NET框架版本同步。

在同一台机器上可以同时存在不同版本的.NET Framework，互不干扰，正常运作。

微软产品的特性：高版本兼容低版本，向下兼容。


c# 跟.net Framework不是一个概念。另外我们通常说的版本都是指.net framework的版本。

不过c#语言本身也有版本之分


http://blog.sina.com.cn/s/blog_4eece9300101ryvq.html


像PHP一样，PHP7比5是语法上有提升，然后解释器不一样。C#的版本是和VS绑定的。VS就是个解释器，也会识别新语法。

C# 8.0
尚在预览版本
C# 7.3
2018 年 5 月
随 Visual Studio 2017 v15.7 发布
C# 7.2
2017 年 11 月
随 Visual Studio 2017 v15.5 发布
C# 7.1
2017 年 8 月
随 Visual Studio 2017 v15.3 发布
C# 7.0
2017 年 3 月
随 Visual Studio 2017 和 .NET Framework 4.7 发布
C# 6.0
2015 年 7 月
随 Visual Studio 2015 和 .NET Framework 4.6 发布
C# 5.0
2012 年 8 月
随 Visual Studio 2012 和 .NET Framework 4.5 发布
C# 4.0
2010 年 4 月
随 Visual Studio 2010 和 .NET Framework 4.0 发布
C# 3.0
2007 年 11 月
随 Visual Studio 2008 和 .NET Framework 3.5 发布
C# 2.0
2005 年 11 月
随 Visual Studio 2005 和 .NET Framework 3.0 发布
C# 1.2
2003 年 4 月
随 Visual Studio 2003 和 .NET Framework 1.1 发布
C# 1.0
2002 年 1 月
随 Visual Studio 2002 和 .NET Framework 1.0 发布



VS
菜单-》项目-》最后一项  更改.net Framework版本
