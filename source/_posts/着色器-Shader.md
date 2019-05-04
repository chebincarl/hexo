---
layout: title
title: 着色器--Shader
date: 2019-05-04 16:11:15
categories: Unity
tags: Unity游戏开发技术详解与典型案例
---
本章将介绍Unity中的着色器和着色语言\-\-ShaderLab。通过本章的学习，读者将会对Unity中的着色器及其开发产生一定的了解。接下来，将先对Unity中的着色器进行简要概述，帮助读者初步认识着色器。

<!--more-->

# 初识着色器

实际游戏开发中的许多特效，如镜面反射和折射、动物毛发、卡通效果等，都是使用着色器来实现的。这些效果如果直接通过编程实现会比较困难，即使实现了，在程序运行的时候其计算量会比用着色器实现同样效果的计算量大很多，因而会影响游戏的整体运行。

## 着色器概述

着色器\-\-Shader是一款运行在图形处理单元（Graphie Processor Unit，GPU）上的程序，其可以让开发人员对图形硬件的渲染功能进行设置。Unity中大多数的渲染都是通过着色器来完成的。Unity中有大量内置着色器程序，开发人员可以直接使用，也可以根据需求开发自己的着色器程序。

目前这种面向GPU的编程有以下3种高级图像语言可供选择。

1.HLSL语言

Microsoft公司提供的HLSL（High Level Shading Language）语言，是通过Direct3D图形软件库来编写着色器程序，只能供Microsoft的Direct3D和XNA使用（Direct3D是Microsoft公司的Directx Graphics的三维部件）。

2.Cg语言

NVIDIA公司和微软公司合作提供了Cg（C for Graphics）语言，Cg与C语言相似，不过有了自己的一套关键词和函数库。Cg是独立于三维编程接口的，完全和Direct3D或者OpenGL结合在一起，一个正确的Cg程序可以编写一次，之后工作在Direct3D或者OpenGL上，这种灵活性意味着Cg提供了一种方法用来编写能够同时工作在主要的三维程序接口和任何操作系统上的程序。Cg的这种多厂商、跨API和多平台的特征使它成为可编程图形处理器编写程序的最好选择。

3.GLSL语言
OpenGL委员会提供了GLSL（OpenGL Shading Language）语言，它是用来在OpenGL中进行着色编程的语言，即开发人员写的短小的自定义程序。它们是在图形卡的GPU上执行的，代替了固定的渲染管线的一部分，使渲染管线中不同层次具有可编程型。

Unity引擎对着色语言的支持非常全面，但为了实现对跨平台性的支持，Unity对着色语言的重点支持为Cg语言。作为一款跨平台性最好的游戏开发引擎，对于要适应不同GPU的着色器来说，Unity使用自定义ShaderLab来组织着色器程序的内容，并将对不同的平台进行编译。

# ShaderLab语法基础

Unity中的着色器程序使用的是ShaderLab着色语言，该语言具备了显示材质所需的一切信息，同时还支持使用Cg，HLSL或GLSL语言编写的着色器程序。ShaderLab着色语言类似于微软公司的FX文件或NVIDIA的CgFX，顶点和片段程序是用Cg或HLSL语言编写。下面将介绍ShaderLab的基本语法结构。

1.Shader

Shader是一个着色器程序的根命令，每个着色器程序都必须定义唯一一个Shader，其中定义了材质如何使用这个着色器渲染对象。Shader命令的语法如下。
```html
Shader "name" {
    [Properties]
    Subshaders |...|
    [Fallback]
}
```

口上面的语句定义了一个名为"name"的Shader.这些内容在材质属性查看器上列于"name"下。着色器程序通过"Properties"来可选地定义一个显示在材质设定界面的属性列表。后面紧跟"SubShaders"的列表,并可额外添加一个代码块用于应对"Fallback"的情况。

口着色器程序拥有一个"Properties"的列表,任何定义在着色器程序中的属性都会显示在属性查看器中。典型的属性有颜色、纹理或是任何被着色器所使用的数值数据。



口着色器程序还包含一个子着色器的列表,其中至少有一个子着色器,当加载一个着色器程序时, Unity将遍历这个列表,获取第一个能被用户机器支持的子着色器,如果没有子着色器被支持, Unity将尝试使用降级着色器,即Fallback操作.

Subshader和Fallbak,在后面的内容中会进行详细讲解。

2. Properties

着色器可以在属性块中定义一些属性参数,这些参数可以由开发人员在Unity的"Inspector"面板中编辑和调整,而不需要单独的编辑器,着色器程序中的Properties块就是用来定义这些参数的地方. Proprties 的基本语法如下.
Rroperties (属性块)

它定义了属性块,其中可包含多个属性。其定义如表7-1所示。
表7-1  Properties类型
