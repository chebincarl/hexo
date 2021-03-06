---
layout: title
title: 预计算实时GI的优化
date: 2019-05-22 15:58:00
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：

<!--more-->

昨天我们学习了烘焙GI的优化，今天我们来学习一下预计算实时GI的优化。

相比烘焙GI的优化，预计算实时GI优化的空间非常大。一个预计算需要数小时的场景，通过优化可以缩短到几分钟。

全局参数优化
我们在学习预计算GI时，对于全局的参数，只设置了一个Indirect Resolution。

Indirect Resolution越大，光影细节程度越高，但是相应的预计算时间越长。

复习一下Indirect Resolution的参考值：

场景  实时GI合适的分辨率
室内  每单位2 - 3个像素
户外  每单位0.5-1个像素
地形  每单位0.1 - 0.5像素
建议场景中1个单位=1米    
为了更好地理解这里面的原因，更好的学习下面的优化参数，在这我们先解释一下Chart。

【选读】理解Chart
这部分内容可以帮助你更好的理解一些设置及后续优化的方法。但是略过此部分也不会影响Unity实时GI的优化设置。

在预计算实时GI中，Chart是光照贴图的一个区域，用来映射到对应的物体的Lightmap UV上。Chart是一张大的光照贴图中的一小块，包含了对应物体的光照信息。一个Chart由两个部分组成：光照信息和主要光的方向。

如图所示，左下角的6个小方块就是Cube对应的6个Chart

预计算实时GI在运行时，会计算所有Chart中的所有像素的光照。场景中过多的Chart是影响预计算时间的重要原因之一，所以理解Chart工作的原理以及如何管理它们对于优化预计算时间非常重要。

Lightmap的UV边缘会留出半个像素来避免和其他区域的渗透（bleed）现象

默认情况下，不管一个物体在场景中的尺寸或UV的大小，一个Chart最小是44像素，也就是16像素。例如，如果物体是11米，物体有一个Chart，indirect resolution设置的是1，那么这个物体就需要16个像素。Chart的4\*4最小尺寸（单个边至少4个像素）保证Unity可以将Chart的边缘缝，保证物体边缘光照的连续性。

如图中边长1厘米的cube，依然需要6个Chart，每个Char是4\*4像素

如果一个物体是1\*1厘米，但是拥有50个UV Chart，尽管这个物体很小，但Unity需要为这个物体创建800像素。一个物体拥有大量的chart可以大幅提高所需像素的数量。更多的像素意味着更多的照明预计算和更多的运行时计算、压缩、存储。在复杂的场景中，可能导致非常长时间的预计算和运行时的性能下降。

不合适的Chart是照明预计算无法完成或花费太长时间的罪魁祸首。基于这一点，很多减少预计算时间的策略是减少场景中的图表数量。

一个物体的Chart数量主要是由Lightmap UV（2通道UV）决定的，但是也可以在外部的3D建模软件（合并UV shell）或者Unity中进行优化。

使用Probe Lighting
对于小尺寸的物体可能会大幅增加Chart数量和所需计算的像素的数量。所以对于这些对间接光贡献不大（由于尺寸小，间接光的贡献有限）的小物体来说，不参与实时GI的预计算是更好的选择。

这些小物体，只需要通过Light Probe接收光照信息就可以实现非常好的光照效果，没有他们反射的间接光不会对场景光影造成影响。

所以在这一步：将场景中尺寸较小的物体整理到一起，取消他们的Lightmap Static。同时在场景中加入Light Probe给这些物体提供间接光。

物体MeshRenderer设置

UV Charting Control
在上面Chart小节，我们知道了Chart会对预计算的时长、运行时的效率影响很大，除了通过外部3D建模软件来优化Lightmap UV以外（较少采用此方法），在Unity中也可以进行优化（常用方法）。

Optimize Realtime UVs
如果美术同学已经手动优化了Lightmap UV，那么不要勾选此选项，可以跳过这一部分。如果没有，请勾选并接着看。

Max Distance
Unity自动优化合并UV时，只考虑小于Max Distance的两个UV块。在保证视觉效果的基础上，这个值越大，Chart相对会越少，效率会越高。

Max Angle
如果两个UV块的夹角小于Max Angle，Unity会将两个UV块缝合，共享同一条边，相当于减少了Chart。

Ignore Normals
有些时候，Unity会将一个Mesh拆分（比如顶点数过多），但是同时也会将Chart拆分开。这可能会增加Chart数量并且增加预计算时间，光影效果也可能不正常。勾选这个选项可以解决这个的问题。

最佳实践
优化UV时，目的是减少Chart的数量，同时减少光影的失真。

调整单个物体的参数时，建议将物体放到一个单独的场景进行调整，这样预计算很快，更容易直观看到调整参数后的结果。然后将参数通过复制或制作成Prefab（后面会详细讲解）的形式，将参数保存下来。

参数进行调整后，对比上一张图，小的Chart就少了很多


Cluster（簇）
之前我们讲了簇的概念：实时GI并没有用真正的网格来计算光影，而是生成了简化的模型，然后用简化后的模型来计算光照。


这个简化的模型是如何生成的？简化后的复杂度如何？这里面还有很大的优化空间。

可以通过Scene看到当前的Clusters

簇的大小和什么有关呢？

Indirect Resolution

Lightmap Paramter中的Resolution和Cluster Resolution

边长1米的立方体和边长5米的地面，实时GI默认值下Cluster的边长也是1米

Cluster的边长变成了1/4

\*\*Cluster的边长 = Cluster Resolution/(Indirect Resolution\* Resolution) \*\*

显然对于场景中的复杂物体，Cluster可以相对多一些，但是对于场景中的大地形来说，Cluster就不需要那么多，可以将这些远处的大地形、大物体的Resolution设置到0.01-0.05，如果效果达不到预期，再增加这个值。

Lightmap Parameters中的其他参数
Irradiance Budget
Irradiance Budget决定了光照贴图中每个像素在Cluster之间取样时可以使用的内存，决定了光照的精准度。越低的值光影会更模糊，越高的值光影会越清晰，但是也增加了运行时的内存和CPU占用。

如果使用实时GI后，在运行时帧率比较低，可以尝试降低这个值。优先降低光影细节比较少的物体，比如远处的、模糊的物体。

Irradiance Quality
Irradiance Quality决定了光照贴图的一个像素在进行周围Cluster的可见性检测时所使用的射线的数量。

更高的值可以有更好、更精准的光影结果，但同时会增加预计算的时间。不过这个值不会影响运行时的性能。

Lightmap Parameters最佳实践
由于Lightmap Parameters是存在于工程中的资源文件，所以建议针对不同类型、不同用途的物体创建对应的Lightmap Parameters。方便对场景中的物体进行设置以及添加新物体时的光照参数设置。

总结
今天学习了使用Unity中的光照系统中的实时GI的优化，可以将本篇当作一个优化手册（检查清单），在需要优化时对照逐项检查。

今日思考题
拿出你的场景，记录下优化前预计算的时长，试试这些参数调整后，预计算节省了多长时间。
