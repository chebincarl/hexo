---
layout: title
title: Standard Shader的用法
date: 2019-05-20 12:57:36
categories: Unity
tags: 大话Unity2018
---
思考题:
动手创建3个材质，分别使用Standard Shader的三种，看看有什么异同。

<!--more-->

对于大部分的材质需求，使用Unity内置的Standard着色器就够用了。

# Standard Shader

Unity内置的Standard Shader包含了非常全面的功能。可以用来渲染真实世界中的物体，比如石头、木头、玻璃、塑料、金属等。Standard Shader在一个Shader中支持了多种Shader类型以及它们之间的组合，仅仅通过设置或不设置材质中的某些属性即可。

Standard Shader还包含一种名为Physically Based Shading（PBS，基于物理的着色）的高级照明模型。PBS模拟了现实世界中材质和光照之间的相互作用。从Unity5开始支持在实时渲染中使用PBS。PBS在灯光和材质一起作用下才能发挥最佳效果。

PBS可以让我们的模型在不同的灯光下看起来表现一致。它的算法模拟光在现实中的特性，遵循物理原理，包括能量守恒（物体不会反射比它们接收的光更多的光），菲涅耳（Fresnel）反射（所有表面在掠射角度下反射会更加强烈），以及表面互相遮挡等等。

Standard Shader是基于硬表面（也称为“建筑材料”）设计的，可以支持大多数真实世界的材质，如石头、玻璃、陶瓷、黄铜、银器、橡胶等。不过它也可以用于不是硬表面的材料如皮肤，头发、布料等，也有一个不错的效果。

Standard Shader将多个类型的shader（例如漫反射，高光，法线高光，折射）合并到单个着色器中。好处是可以在场景的所有区域使用相同的照明计算方式，从而为使用Standard Shader的所有模型提供逼真、一致的光照和阴影。

# 三种Standard Shader的区别

如果你尝试切换Shader，你会发现有三种Standard Shader：

{% asset_img 1.png %}<center>三种Standard Shader</center>

这三种Shader能达到的效果是相同的，只是在属性参数设置上稍微有些不同。

# Standard

Shader的参数中有一个“metallic”属性，用来设置材质的金属度。

金属度最低时，材质会有与入射光相同颜色的高光，并且模型表面时几乎不会反射。金属度最高时，Albedo颜色会控制高光的颜色，并且大部分光线将反射为高光。

{% asset_img 2.gif %}<center>metallic参数从0-1的变化</center>

# Standard(Specular高光 setup)

这个Shader适合很多传统的美术工作流程。高光颜色用于控制材质中镜面反射的颜色和强度。这样就能设置与漫反射不同颜色的镜面反射颜色。

{% asset_img 3.png %}<center>只修改灰度，和metallic属性的效果类似</center>

{% asset_img 4.png %}<center>但是还可以给个颜色</center>

如上图所示，Specular的颜色的灰度值可以对应到Standard中的Metallic的0-1，此外Specular还可以设置颜色，这个颜色会影响高光。

# Standard(Roughness setup)

Roughness与上面两个Shader不同的是，Roughness Shader中没有Smoothness（光滑度）这个属性，取而代之的是Roughness（粗糙度）这个属性。顾名思义，光滑度就是设置材质的光滑程度，取值范围是从0到1，0代表光滑度很低（很粗糙），1代表非常光滑（类似镜面）；粗糙度就是设置材质的粗糙程度，取值范围是从0到1，0代表粗糙度很低（很光滑），1代表粗糙度很高（很不光滑）。

所以Standard（Roughness setup）与Standard的区别是：一个取值相反参数。如果在Standard中Smoothness设置为0.8，那么在Standard(Roughness setup)中Roughtness值设置为0.2就能取得相同的效果。

一般情况下，上面三个Shader选择使用任何一种都可以很好地表现大多数常见材质。因此大多数情况下，选择一种或另一种是个人喜好的问题，只要适合你的项目的美术工作流程即可。例如，以下是三种不同Shader创建的橡胶塑料材质的示例：

{% asset_img 5.png %}

第一个代表metallic工作流程，材质的Metallic设置为零（非金属）。第二个设置几乎完全相同，但高光设置为几乎黑色（所以我们不会看到镜面反射）。第三个将Roughness设置为0.4，即1-Smoothness。

有人可能会问，这些值从何而来，什么是“近乎黑色”，究竟是什么让草与铝不同？在基于物理着色的世界中，我们可以使用来自已知真实世界材质作为参考。Unity官方已经将这些参考文献中的一部分编译成了一组可用于创建材料的便利图表。

    
{% asset_img 6.png %}<center>Standard参考图表</center>

{% asset_img 7.png %}<center>Standard(Specular setup)参考图表</center>

> Standard与Standard(Roughness setup)的设置几乎一样，只是Smoothness = 1 - Roughness，所以Standard(Roughness setup)可以参考Standard图表。

# 总结

Standard有三个Shader，选择使用任何一种都可以很好地表现大多数常见材质。因此大多数情况下，选择一种或另一种是个人喜好的问题，只要适合你的项目的美术工作流程即可。