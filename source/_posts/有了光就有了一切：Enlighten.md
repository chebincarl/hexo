---
layout: title
title: 有了光就有了一切：Enlighten
date: 2019-03-10 18:45:30
categories: Unity
tags: Unity AR/VR开发\-\-从新手到专家
---
本章涵盖：
* Unity光照系统介绍 
* 全局光照
* 实战：给BattleStar游戏场景添加光照

<!--more-->

# Unity光照系统介绍

本章将介绍游戏中至关重要的灯光系统，从一定层面上说，光照效果直接决定了游戏所表达的情绪。场景中明亮通透的灯光能给玩家舒缓放松的感觉，而阴暗低沉的灯光则能很好地营造出紧张阴郁的氛围。 

Unity引擎中提供的光照系统叫做Enlighten，它作为引擎渲染功能的一部分，负责构建场景中的灯光。

## Light组件简介 

Unity中的灯光系统并不复杂，各个不同类型的灯光组件实现不同类型的光效。Unity的灯光组件大致分为两个类别：光源组件和烘焙组件，光源组件应该非常容易理解，只有能自己发出光的物体，才能被称作光源。这一概念放到Unity中，也就是如下几种灯光：Directional Light、Point Light、Spot Light。

## 常见的光源类型
### Directional Light  

Directional Light也即平行光，它是场景中的主要灯光。几乎每一个场景中都会使用到这个光源对象，它常用于模仿太阳光的效果。它与Point Light和Spot Light最大的不同在于：Directional Light并没有真正的“源”。在游戏中，Directional Light从同一个角度照射场景，也就是说Directional Light在整个场景中的任何一个角落的光照强度都是相同的。 

在Unity中创建新场景时，场景中默认会有一个Directional Light和一个Main Camera。

各个光源对象在Scene视图中的图标都不同，如Directional Light的图标是一个太阳，比较便于理解。读者可以自行尝试调整Directional Light的位置，来观察它对于场景的影响。

事实上Directional Light的光照效果完全不受位置的影响，最直观影响Directional Light灯光效果的因素是角度。在场景中调整Directional Light的角度为朝正上方，可以很明显地看到整个场景变成了黑色，不再有光照的效果。

下图就是在Unity中使用Directional Light和地球模型，模拟宇宙中太阳光照的效果。

在Directional Light的Inspector视图中，同样显示了灯光的类型。左上角各个不同的灯光类型也有各自的图标。在Type选项中可以直接切换灯光的类型，如下图所示。

### Point Light
Point Light顾名思义就是点光源，点光源从中心呈球形向四周扩散，如火把、室内灯具等通常都使用Point Light实现。

Point Light的效果受到范围（Range）和强度（Intensity）的影响。和Directional Light不同，点光源的效果是会受到位置影响的，如下图所示。

### Spot Light
Spot Light即聚光灯，从中心呈扇形向某一个方向发出，受扇形角度（Angle）和范围（Range）的影响。Spot Light可用于模拟手电筒和车灯等的效果，如下图所示。

除以上三种基本的光源组件外，Unity还提供了一种特殊的光源：Area Light，也就是区域光。

### Area Light  
区域光和以上三种灯光最大不同在于，它只能在烘焙的情况下使用，而Directiona  Light、Point Light、Spot Light能够在实时（Realtime）和烘焙（Bake）两种情况下使用。Area Light用于一些较特殊的情况，如某个场景的主要场景在室内，以上三种灯光都无法较好地实现灯光效果时就可以使用Area Light来实现这一效果，如下图所示。 

### Light inspector中的参数简介  

灯光效果的把控非常依赖于开发者的个人审美和感觉，所以开发者应该了解灯光组件中各个参数的用处，这样才能调试出最理想的灯光。

* 1)在Unity中新建一个项目，将其命名为MyLights。保存默认的场景，将其命名为MainScene。在Hierarchy视图中右键单击，并添加一些简单的几何体对象。

* 2) 在Hierarchy视图中选择Directional Light，在Inspector视图中确认Directional Light组件下的Shadow Type设置为Soft Shadow。关于阴影的其他参数保持默认即可。Soft Shadow所呈现的阴影比较柔和，更接近真实，但性能开销也更大。Hard Shadow所呈现的阴影更硬朗，锯齿感也更强。

* 3) 接下来可以调整Directional Light的角度，可以很直观地看到，整个场景的色调、阴影的效果都发生了改变，如下图所示。

读者也可以自行调整Inspector视图中的Color(色彩)和Intensity(强度),来进一步改变Directional Light的效果。

* 4) 接下来在场景的中心位置添加一个Point Light组件,并调整Color为更显眼的颜色,比如热情洋溢的红色。选中Hierarchy视图中的Point Light组件,可在Scene视图中看到Point Light的范围,在Inspector视图中调整Range (范围)的1中心较明显的红色区域也会越来越大。这缘于Point Light的范围特性,并不是说范围内所有区域的灯光强度都是相同的,而是呈从中心向边缘递减的效果。如果读者不太理解,可以继续调整Range的值,并观察场景中红色区域的范围。

Point Light同样支持Hard Shadow和Soft Shadow。但默认情况下, PointLight的阴影效果并不会呈现出来。因为实时的Spot Light和Point Light并不支持阴影,只有在烘焙后才能看到它们的阴影效果。关于烘焙的具体操作,将在后续章节中进行介绍。

此外灯光组件还有两个较常用的属性: Cookie和Flare, Flare即光晕,而Cookie则用于显示一些特殊的阴影,如图5-10所示。

如图5-10中,聚光灯穿透纸面,在幕布上投射出特殊的阴影。Cookie的作用类似于纸面,开发者在图片编辑器(如PhotoShop等)中调整好材质后,设置Spot Light或其他灯光组件的Cookie属性即可。
图:

# 全局光照

在前面的章节中介绍光源时,只是介绍各个灯光单独作用的场景。但在实际开发中,  大多数情况下灯光都是相互作用的,如灯光照射到物体A上,物体A反射的光会照射到物  体B。这种关联关系是通过全局光照(Global Illumination, GI)系统来进行处理的。
5.2.1 全局光照简介图5-11显示了全局光照的作用效果,在一个封闭空间内,两个玻璃球体互相反射。全局光照极大提升了场景中光照的真实性,但这种程度的实时计算是非常消耗资源的。  但从另外一个方面来说,我们只需要对场景中的动态物体进行实时计算,保证光照效  果,而那些固定的物体,或许不应该在它们身上浪费太多资源。先想象一下,如果一个场景中所有物体全部是静止的状态,那么实时计算光照效果显然是白白浪费资源的,我们只需要执行一次计算即可。这种技术在Unity中称为烘焙（Ba-  ke)。当对场景进行灯光烘焙后,场景中的光照信息就会存储在Lightmap中，当场景运行时, Unity直接读取Lightmap中的数据即可,而无需再进行一次计算,这种工作流程很好地避免了不必要的性能消耗。

## 烘焙
图5-11全局光照的作用效果

稍等片刻烘焙结束后,可以看到场景中的火在Scene视图中,之前勾选为Lightmap Static的对象无法被移动,修改Directional Light的角度会发现,对象的阴影并不会改变。修改Point light的颜色、范围等,同样场景不会有任何改变。

这是因为Directional Light. Point Light和场景中所有对象的光照信息都已经烘焙到Lightmap中了,现在场景中的光照数据来自Lightmap,而不是根据灯光变化实时计算。

如果场景中的灯光发生了变化,开发者需要手动再次进行烘焙,场景中的光效才会发生改变。

## Lightmap的使用
	
读者现在已经知道,在进行灯光烘焙后,场景中的光照信息全部储存到了Lightmap中。由Lighting视图切换到Global maps中,如图5-15所示。

灯光烘焙的数据可以直接在Assets视图中查看,存储在与场景同级的文件夹中,文件夹名称和场景名称一致。双击图5-15标记的左侧烘培数据,即可通过图片浏览器打开该文件,如图5-16所示。

图5-16中就是完成烘焙后的阴影、灯光信息。我们通常不会对这些数据进行更改,但开发者有必要知道它们存放的路径,当需要移动场景文件的路径时,也需要移动这些文件,或者重新进行烘焙

## Light Probe和Reflection Probe的使用  

当场景中的灯光烘焙后,光照信息和阴影都变成“静止”的了。如果场景中有动态的  物体,比如可以自由行走的玩家,那玩家岂不是没有阴影了?这个问题我们该怎么解决呢?  这个时候就需要用到Light Probe和Reflection Probe了。


