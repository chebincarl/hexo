---
layout: title
title: 初识Unity编辑器
date: 2019-05-16 11:27:43
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1..meta文件是干嘛用的？如何设置隐藏？
2.如果Unity崩溃未保存场景，如何恢复？
3.空游戏对象有哪些使用场景？
4.Preset是干嘛用的？
5.unitypackage是什么？优点是什么？
6.Unity项目下哪几个文件夹需要加入版本控制？

<!--more-->

# Template 模板

{% asset_img 2.png %}

| <center>** 模板 ** </center>  | <center>** 描述 ** </center>  |
| :-| :- |
| 2D  | 使用Unity内置渲染管道的2D项目。默认使用2D的项目设置，包括图像导入参数，Sprite打包器，场景视图，照明和正交相机。  |
| 3D  | 使用Unity内置渲染管道的3D项目。默认使用3D的项目设置，包括2018的新功能线性颜色空间和渐进式光照贴图。  |
| 3D with Extras  | 与3D模板类似，但包含了后处理，预设和示例内容。  |
| High-Definition RP(Preview)  | 适用于支持Shader Model 5.0（DX11及以上版本）的平台上的高端图形。该模板使用HD RP，一种现代渲染流水线，其中包括高级材质类型和可配置的混合Tile/Cluster延迟/前向照明架构。  |
| Lightweight RP  | 关注主要使用烘焙照明解决方案的项目。该模板使用LW RP，这是一个单pass前向渲染器，每个对象都有光照剔除。使用LW RP将减少项目的绘图调用次数，使其成为低端硬件的理想解决方案。所有灯都以单通道着色，而不是每个像素灯一个pass。  |
| Lightweight VR RP(Preview)  | 开发主要使用烘焙照明解决方案的VR项目，重点关注性能。该模板使用LW RP并需要VR设备来运行。  |

如果选错了，可以后续在菜单栏Edit->Project Settings->Editor中的Default Behavior Mode中修改。

{% asset_img 3.png %}

# 查看工程目录结构

打开刚才创建工程的磁盘目录，你会看到Unity根据你的工程名称自动创建了一个文件夹，对于我来说就是MyFirstUnityProject，文件夹里面有以下内容：

{% asset_img 1.png %}
<center>目录结构</center>

| <center>** 文件夹 ** </center>  | <center>** 内容 ** </center>  |
| :-| :- |
| Assets  | 工程中所有资源的存放目录  |
| Library  | Unity自动生成的中间文件目录  |
| Packages  | Unity的包管理器相关文件存放目录  |
| ProjectSettings  | 工程的相关设置  |
| Temp  | 只有Unity打开这个工程的时候，Temp目录才会出现存放一些临时文件，关闭Unity会该目录会自动消失  |
| sln文件和csproj文件  | Unity自动生成的Visual Studio工程文件，用于代码编写和调试  |

** 小技巧 **

这些文件中，Assets、Packages、ProjectSettings三个文件夹是必须的，也不能删除，其他文件都可以由Unity自动生成。在存档、拷贝工程源文件时，可以只压缩Assets、Packages、ProjectSettings这三个文件夹，可以减小压缩包的体积。

# 资源文件的位置

如果你需要直接访问下载下来的资源文件，你可以在下方路径找到：

Windows: C:\Users\‹accountName›\AppData\Roaming\Unity\Asset Store

下载下来的资源会以发布者的名称作为文件夹，你可以在对应的发布者名称的文件夹下找到对应的资源。

# 其他
人们总是对未知的事物有莫名的恐惧。克服恐惧的过程就是你成长的过程。

就拿Unity来说，英文、编程、每隔几个月的新版本新功能的变化都可能让你恐惧和焦虑。

书读百遍，其义自见。剩下1%看不懂的内容，你见的多了，自然就懂了。


TextMesh Pro（用于渲染字体）、ProBuilder（可以在Unity编辑三维模型）、Cinemachine（强大的相机插件）

Standard Assets 内置标准资源：安装此组件后，可以使用Unity官方自带的资源如地形资源，角色控制器等。如果没安装，可以在Asset Store中找到。

如果少安装了组件也无须担心，下次再次打开vs_community.exe安装文件，可以添加安装新的内容
