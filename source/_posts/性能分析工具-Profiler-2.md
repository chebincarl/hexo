---
layout: title
title: 性能分析工具-Profiler-2
date: 2019-04-10 09:22:16
categories: Unity
tags: Unity5.X游戏开发指南
---
本章涵盖
* 渲染优化
* 内存优化

<!--more-->

# 渲染优化

渲染主要和显卡上搭载GPU运算有关。如果在Profiler的CPU栏下显示Gfx.WaitForPresent，那么就表示GPU每帧渲染需要的时间过长，CPU需要等GPU。

## 渲染参数

和渲染有关的最重要的几个参数可以在Game窗口下查看。在Game窗口中点击“Stats”按钮弹出面板，如下图所示。

其中，FPS和毫秒数代表的是在Unity编辑器下运行的效率，所以仅供参考，真正重要的是下列参数。
* Tris：Trianglie，三角形的数量，渲染的基础指标。图中为17.1K，也就是当前画面一共渲染了约17100个三角形。
* Verts：Vertices，模型顶点的数量，渲染的基础指标。图中为10.7K，也就是当前画面一共渲染了约10700个顶点。
* SetPass calls：SetPass的调用次数。
* Batches：合并后的Drawcall次数。
* Saved by batching：被合并的Drawcall次数。
* Shadow casters：阴影投射图的数量。

## 优化

优化主要分为4个方面：模型优化、材质优化、光照优化和Draw Call合并。

** 1.模型优化 **

三角形和顶点数代表了基本场景的几何负担。在保证效果的情况下，它们当然是越少越好。主要是在3D建模工具中要将视角并不可见的面裁掉，如果是固定视角的场景则非常简单，将摄像机视角的反面删除即可；如果是自由视角，通常也存在死角，如房屋内部、地底等。

** 2.材质优化 **

材质方面首先使用尽量小的可以接受的贴图尺寸。在通过代码操作材质的时候，请尽量使用renderer.sharedMaterial或者renderer.sharedMaterials。而不要使用renderer.materia或者renderer.materials，因为对后两者的每一次改动都会创建一个新的材质。

这里首先介绍一个概念：像素填充率。像素填充率是指图形处理单元在每秒内所渲染的像素数量。当一个像素点渲染的是不透明材质，那么只有一次运算；而如果该像素点是由很多透明的物体像素点叠加显示的话，就是多次运算，会加大GPU的负担。

材质优化也包括Shader渲染器的选择。Shader渲染器本书并不涉及，这里主要说一下原则。
* 首先尽量少使用Standard Shader，因为它的参数比较多，运算相对也就比较多。
* 一般可以不接受光照的物体就用不参与光照的Shader。选中一个材质，在Inspector窗口的Shader栏展开下拉菜单，其中Unlit栏里的都是不参与光照的Shader。
* 对于不涉及颜色变化的物体，请尽量用没有颜色参数的Shader。

** 3.光照优化 **

请尽量使用烘焙，预先烘焙好场景的Lightmap。如果是移动平台开发，要控制光的数量以及谨慎使用产生实时阴影的光。

** 4.Draw Call合并 **

下面就重点介绍下Draw Call合并。首先，我们介绍下Draw Call的概念。

Unity（或者说基本上所有的图形引擎）生成一帧画面的处理过程大致可以这样简化描述：引肇首先经过简单的可见性测试，确定摄像机可以看到的物体，然后把这些物体的顶点（包括本地位置、法线、 UV等）、索引（顶点如何组成三角形）、变换（就是物体的位置、旋转、缩放以及摄像机位置等），相关光源、纹理、渲染方式（由材质/Shader决定）等数据准备好，然后通知图形API\-\-或者就简单地看作是通知GPU\-\-开始绘制，GPU基于这些数据经过一系列运算在屏幕上画出成千上万的三角形，最终构成一幅图像。

<span style="color:red;">在Uniy中，每次引擎准备数据并通知GPU的过程称为一次Draw Call，这一过程是逐个物体进行的。</span>对于每个物体，不只GPU的渲染，引擎重新设置材质/Shader也是一项非常耗时的操作。因此，每帧的Draw Call次数是一项非常重要的性能指标。Unity内置了Draw Call合并技术，顾名思义，它的主要目标就是<span style="color:red;">在一次Draw Call中批量处理多个物体。只要物体的变换和材质相同，GPU就可以按完全相同的方式进行处理，即可以把它们放在一个Draw Call中。</span>Draw Call Baching（即Draw Call合并技术）的核心就是在可见性测试之后检查所有要绘制物体的材质，把相同材质的分为一组（一个Batch ），然后把它们组合成一个物体（统一变换），这样就可以在一个Draw Call中处理多个物体了（实际上是组合后的一个物体）。

因此，Draw Call优化合并（即Draw Call Batching）通过减少每帧的Draw Call数降低显卡计算从而提高性能。

在Unity中，渲染模型有两种方式：Skinned Mesh Renderer和Mesh Filter加Mesh Renderer。Skinned Mesh Renderer渲染带有骨骼动画的模型，而Mesh Filter加Mesh Renderer渲染没有骨格动画的模型。Draw Call合并只针对Mesh Filter加Mesh Renderer，也就是Skinned Mesh Renderer渲染带有骨骼动画的模型是没有Draw Call合并的。

Draw Call合并分为Dynamic Batching动态合并和Static Batching静态合并两种。

** 动态合并 
** 

动态合并不需要任何操作。游戏运行时，所有使用相同材质的游戏对象，无论是否使用相同的网格模型都可被合并。被合并的对象依然可以自由移动旋转，但有以下使用要求。

* 模型文件共计点数不超过900（重复使用同一个Mesh不计）。

* 单个物体可以不超过300点，Shader可以有法线UV，但如果Shader使用了UV0 UV1两套UV吗，或者使用了Tangent切线的话，单个物体只能不超过180点。

* 游戏对象使用相同模型和材质时，只有相同缩放（即xyz等比缩放，浮点尾数可以有细微差）的会被合并。比如：(1,1,1)与(1,1,1)、(2,2,2)与(2,2,2)、(0.5,0.5,0.5)与(0.5,0.5,0.5)、(2,2,2)与(2,2,2.0001)

* 场景烘焙：烘焙后同材质将不会被烘焙。lightmap有隐藏的材质参数：offset/scale，所以使用lightmap的物体不会被合并。

* Shader不能使用多Pass：多Pass的Shader会破坏Dynamic Batching。

** 静态合并 **

静态合并的原理是在运行游戏后将一组游戏对象的多个网格模型合并为1个网格模型。使用同一材质的游戏对象都在一个DrawCall中完成，这些游戏对象运行后无法移动缩放旋转。但是Drawcall一定是最大化合并的，并且不受动态合并的诸多限制。

注意在游戏运行时，静态合并会动态创建合并后的网格模型，因此过多的静态合并会增加内存占用。例如，场景里的树群就不适合静态合并，而适合动态合并。

静态合并的实现方法有以下两种。

* MeshRenderer勾选Batching Static。

* 代码中使用UnityEngine.StaticBatchingUtility实现（可以在任何平台调用），方法是创建一个空的游戏对象作为根对象，将所有要合并的静态物体（不需勾选Batching Static）置于其下，然后使用StaticBatchingUtility.Combine(root);。

以上两种实现方式的区别是勾选Batching Static属于完全自动合并，在MeshFilter里显示的是Combined Mesh(root:scene)，合并后不能移动；而代码调用StaticBatchingUtility合并到一个游戏对象下，合并后可以移动父节点游戏对象。

# 内存优化

介绍了CPU优化和谊染优化后，接下来我们介绍内存优化。内存优化一般来说主要从以下两方面人手：降低资源大小、及时释放不用资源。

## 降低资源大小

要降低资源大小，首先需要分析哪些资源占用较大的比重。在Build后，打开Console界面点击界面右上角的下拉菜单>Open Editor Log打开Editor日志，其中有在整个包中按大小排序的资源列表，它显示了资源的大小以及在安装包中所占的比例。一般来说，一个3D游戏项目中贴图及UI资源图至少古用50%以上的容量，其次可能就是模型资源、音频资源等。这些资源首先尽量选择合适的压缩格式，并在可接受的范围内适当降低品质，例如一个按钮框素材如果使用1024x 1024的尺寸就明显是浪费了，使用256x256的尺寸是更好的选择。而且，一定要慎用全屏背景图，这类图每张都会占用非常大的空间。在设计UI时，多考虑UI的复用性，多使用九宫格都是很有帮助的。

## 释放内存中的资源

及时释放内存中的资源也是内存优化中非常重要的环节。我们有AssetBundle.Unload、Resource.unload、 GC.Collect等方法可用。

# 其他优化经验

* 设置目标帧率。

通过Application.targetFrameRate设定FPS上限，好处是稳定帧率，减少在高帧率和低帧率间切换造成的不流畅感。这也会减少设备发热和耗电。移动设备上推荐设置为30。

* 音频格式

在游戏中播放时间较长的音乐（如背景音乐）时，建议音频使用OGG或者MP3压缩格式；短促音效（如枪声、按钮点击音效等）建议采用WAV或者AlF未压缩格式

* 
摄像机

摄像机将远平面设置成合适的距离。远平面过大会将一些不必要的物体加入渲染，降低效率。

* 
碰撞

尽量使用立方体或者圆柱体等基本碰撞体。对于复杂的网格模型，请尽量不要使用MeshCollider，而选用基本的立方体或者圆柱体等去拼近似形状。

# 习题

1.简述什么是预定义标签以及它可以用于哪些地方？
2.什么是Drawcall合并？它有哪几种实现方式？
3.使用Profiler分析Unity示例工程AngryBots运行时的性能。