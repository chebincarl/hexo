---
layout: title
title: Standard Shader的参数设置（2）
date: 2019-05-22 11:59:44
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.动手调节一下参数的值，尝试给每个属性一个贴图试试看。也可以下载Asset Store中的Adam Character资源包，看看Unity官方制作的角色是如何使用这些参数的。

<!--more-->

# Heightmap（高度映射）

高度映射（也称为视差映射）与法线映射类似，但是这种技术更复杂 - 因此性能相对较差。高度贴图通常与法线贴图一起使用，通常用于渲染物体表面较大的凹凸。

法线贴图修改了材质表面上的光照，高度贴图更进一步，实际移动了物体表面的贴图，实现了表面遮挡效果。这意味着凹凸在近侧（面向相机）被放大，在远侧（远离相机）将减小并且看起来被遮挡。

这种技术可以产生非常真实的效果，但仍然局限于物体网格的表面。虽然表面凹凸会突出并相互遮挡，但模型的“轮廓”并没有被修改，因为该效果最终会被绘制到模型的表面上，而不会修改实际的网格。

高度图应该是灰度图像，白色区域代表纹理的高区域，黑色代表低区域。如下图是一个匹配的Albedo贴图和高度图。

{% asset_img 1.png %}
<center>Albedo贴图和与其匹配的高度图</center>

{% asset_img 2.png %}

在上图中从左到右：
1、指定了岩石墙的Albedo贴图，但没有法线贴图或高度图。
2、添加了法线贴图，光照在表面发生了变化，但岩石不会相互遮挡。
3、添加了normalmap和heightmap的最终效果。岩石看起来凸了出来，并且挡住了后面的石头。

通常（但不总是）用于高度图的灰度图像也适用于遮挡贴图。

# Occlusion Map（遮挡贴图）

{% asset_img 3.png %}

遮挡贴图用于提供模型的哪些区域应该接收高或低间接光照的信息。间接照明来自环境照明和反射，因此模型的陡峭凹陷部分（如裂缝或折叠）实际上不会接收到太多间接光照。

通常使用建模软件或第三方软件基于3D模型计算遮挡贴图。

遮挡贴图是灰度图像，其中白色表示应该接收全部间接照明的区域，而黑色表示没有间接照明。有时候，对于简单的曲面（比如上面的heightmap示例中显示的可凸起的岩石贴图），遮挡贴图与灰度高度图几乎一样。

在其他时候，生成正确的遮挡贴图稍微复杂一些。例如，场景中的角色戴着头巾，头巾的内侧边缘应该设置为非常低的间接照明，或者根本没有。在这些情况下，遮挡贴图通常会由美术同学通过3D软件生成。

{% asset_img 4.png %}
<center>这张遮挡贴图用于下面的模型</center>

{% asset_img 5.png %}
<center>使用遮挡贴图的前后：注意脖子周围的颜色变化</center>

# Emission（自发光）

{% asset_img 6.png %}

该属性用来控制材质表面发光的颜色和亮度。在场景中使用自发光材质时，它会是场景中的光源。

自发光材质通常用于从内部照亮外部的物体，例如显示器的屏幕，汽车急刹车时被烧红的刹车盘，仪器控制面板上的发光按钮或黑暗中怪物的眼睛。

可以使用单一的HDR颜色或自发光贴图来设置自发光颜色。

{% asset_img 7.png %}

HDR颜色相对于普通颜色，多了一个Intensity的属性，可以用来定义发光的强度。

> HDR(High Dynamic Range)
这是通常指超出正常颜色范围（0-1）以外的颜色。例如，太阳通常比蓝天亮十倍。

{% asset_img 8.png %}
<center>使用HDR的场景：车窗中反射的阳光比场景中的其他物体亮得多，因为已经使用HDR进行处理</center>

使用自发光材质的物体，即使在黑暗环境中，由于他们自身的发光，仍可以看见它们。

{% asset_img 9.png %}

除了使用一个简单的HDR颜色来控制材质的自发光，你也可以使用自发光贴图。和其他的贴图类似，通过自发光贴图，你可以控制哪个区域发光，哪个区域不发光。

{% asset_img 10.png %}
<center>图中的贴图有发光区域和不发光区域</center>

{% asset_img 11.png %}
<center>在场景里的表现</center>

除了基本的颜色和贴图设置，Emission参数还有一个Global Illumination（全局光照，后续光影的课程会讲到）设置，用来设置这个自发光的物体发出的光照会如何影响周围的物体。有两个选项：

Realtime 实时。这个自发光光源会添加到全局光照的计算中，这个光源附近的物体，包括移动的物体会受到这个光源的影响。

Baked 烘焙。这个自发光光源的会被烘焙到静态的灯光贴图（后续光影的课程会讲到）中，周围的静态物体会被点亮，但是动态物体不会受影响。

{% asset_img 12.png %}
<center>自发光的物体烘焙后对场景的影响</center>

# Tiling/Offset

{% asset_img 13.png %}

Tiling表示UV坐标的缩放倍数，Offset表示UV坐标的起始位置。贴图的左下角为UV坐标系的(0,0)，右上角为(1,1)。

这么说有点晦涩，我们来举个例子：

{% asset_img 14.png %}
<center>左侧为Tiling为(1,1)，中间Tiling为(2,2)，右侧Tiling为(0.5, 1)</center>

{% asset_img 15.png %}
<center>左侧为Offset为(0,0)，中间Offset为(0.3,0.3)，右侧Offset为(0.5, 0.5)</center>

# Secondary Maps（次要贴图）

{% asset_img 16.png %}

次要贴图（又叫辅助贴图、细节贴图），可以在主贴图的基础上覆盖第二层贴图。次要贴图可以包含Albedo贴图和法线贴图。

通常使用次要贴图的情况是：需要在物体表面重复一些很小（相对于主贴图）的纹理。比如：皮肤的细节（毛孔和毛发），砖墙上细小的裂缝或青苔，大型金属容器上的小划痕和磨损。

次要贴图能让材质从近处看时非常精细，而从远处看时是有正常的细节，避免了使用单张精度极高的贴图，可以提高性能。

比如下图是一个没有次要贴图的皮肤：

{% asset_img 17.png %}
<center>未添加次要贴图</center>

{% asset_img 18.png %}
<center>添加了次要贴图</center>

{% asset_img 19.png %}
<center>添加了次要贴图后，能清楚看到皮肤上的毛孔</center>

> ** 注意 **
如果只有一张法线贴图，一定要放在Main Maps的Normal Map中。Secondary Maps中的法线贴图虽然和Main Maps的作用相同，但是比在Main Maps中会耗费更多渲染资源。

# Detail Mask（细节贴图遮罩）

{% asset_img 20.png %}

细节遮罩贴图只在使用了Secondary Map时生效，用来遮挡Secondary Map的某些区域。可以用来控制只在某些区域显示细节纹理，在其他区域隐藏细节纹理。在上面的脸部例子中，你可能需要考虑创建一个遮罩，在嘴唇或眉毛上不显示毛孔。

# Advanced Options（高级选项）

{% asset_img 21.png %}

* Enable GPU Instancing（开启GPU实例化）
使用GPU Instancing可以将绘制多个同样的Mesh合并成一次渲染。对于场景中大量重复的的建筑、植被等物体的渲染非常有用。

GPU Instancing可以减少调用底层图形接口的次数，可以大幅改善了项目的渲染性能。

* Double Sided Global Illumination（双面全局光照）
启用后，在计算全局照明时，会计算网格的正反两面。背面不会渲染，也不会添加到光照贴图，但会遮挡其他物体。

使用Progressive Lightmapper时，反面反射的光使用与正面相同的albedo和emission（目前此设置仅在使用Progressive Lightmapper进行烘焙时可用。）

# 总结

今天学习了Standard Shader的几个属性：
* Heightmap 高度映射
* Occlusion Map 遮挡贴图
* Emission 自发光
* Tiling/Offset
* Secondary Maps 次要贴图
* Detail Mask 次要贴图遮罩