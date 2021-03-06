---
layout: title
title: 3D数学与Unity
date: 2019-04-06 17:16:21
categories: Unity
tags: Unity 3D实战核心技术详解
---
用Unity引擎开发游戏就是在引擎的上层再封装一层游戏架构，在游戏开发的逻辑中需要根据需求重新封装一些数学算法以便于逻辑调用。

<!--more-->

# Unity坐标系

3D坐标系表示的是三维空间，3D坐标系存在三个坐标轴，分别为x轴、y轴、z轴，如图1-1所示。

{% asset_img 1.png %}

3D坐标系分为左手坐标系和右手坐标系。Unity使用的是左手坐标系。

Unity引擎的左手坐标系也被称为世界坐标系，做游戏开发时需要美工制作美术模型，运用MAX工具把建好的模型放到游戏场景中。在默认情况下，局部坐标和世界坐标系的原点是重合的，不能把所有的模型都叠加在世界坐标系的原点上，因此需要移动模型。<span style="color:red;">模型移动时就会发生模型的局部坐标到世界坐标的转换，这个移动过程就是把模型的局部坐标转化成世界坐标。</span>只是这个转化过程是在引擎编辑器内部实现的，实际上它就是将模型的各个点与世界矩阵相乘得到的。

Unity编辑器中的物体都在世界坐标系里面，比如我们通常使用的函数transform.position，它就是获取到当前物体的世界坐标位置，用户无须自己去计算，因为引擎内部已经计算好了。明白了原理后，再使用编辑器解决问题更有助于理解，做到知其然且知其所以然。如果要获取物体自身的坐标，也就是局部坐标，可以使用函数transform.localPosition获取当前模型的局部坐标。

用Unity引擎开发移动端手游会经常用到屏幕坐标系，<span style="color:red;">屏幕坐标系就是通常使用的电脑屏幕，它是以像素为单位的，屏幕左下角为（0，0）点，右上角为（Screen.Width，Screen.Height）点，Z的位置是根据相机的Z缓存值确定的。</span>通常使用鼠标在屏幕上单击物体，它就是屏幕坐标。通过函数Input.mousePosition可以获得鼠标位置的坐标。我们使用的虚拟摇杆可以在屏幕上滑动，它也是屏幕坐标，可以通过函数Input.GetTouch（0）.position获取到手指触摸屏幕坐标。

在游戏开发中，比如单击场景中的3D物体就需要从屏幕上发射一条射线与物体的包围盒相交，用于判断是否选中物体，对于UI的操作也都是基于屏幕坐标系的。

通过相机才能看到虚拟世界的物体。相机有自己的视口坐标，物体要转换到视口坐标才能被看到。相机的视口左下角为（0，0）点，右上角为（1，1）点，Z的位置是以相机的世界单位来衡量的。（0，0）点和（1，1）点是通过公式进行缩放计算的，这里面存在一个变换，读者了解就可以了。这也是为什么视口的大小通常都是（0，0）和（1，1），效果如图1-2所示，图的中心点是摄像机。

{% asset_img 2.jpg %}

下面介绍世界坐标、屏幕坐标、相机坐标之间的转换方式。举一个简单的例子，在一个空场景里面放置一个立方体，物体在编辑器中也就是世界坐标系中的摆放如图1-3所示。

{% asset_img 3.jpg %}

获取物体位置的通常写法是transform.position，它表示的是立方体在3D世界中的世界坐标的位置。如果使用的是触摸屏幕，那么可以通过函数Input.GetTouch（0）.position获取到屏幕坐标。它们之间的转换方式如下。

* 世界坐标到屏幕坐标的转化函数：camera.WorldToScreenPoint（transform.position）。

* 屏幕坐标到视口坐标的转化函数：camera.ScreenToViewportPoint（Input.GetTouch（0）.position）。

* 世界坐标到视口坐标的转化函数：camera.WorldToViewportPoint（obj.transform.position）。

这些转换也是固定流水线的矩阵变换，只是Unity将其封装好了而已。如果想学习固定流水线，可以参考《手把手教你架构3D游戏引擎》一书，里面有固定流水线的详细讲解，下面介绍向量运算。

# 向量

向量的基本运算包括加法、减法、点乘、叉乘、单位化运算等，其中减法、点乘、叉乘、单位化运算在游戏开发中使用得最为广泛。

首先介绍一下向量，向量的表示如图1-4所示。

{% asset_img 4.jpg %}

向量是具有方向和长度的矢量，它并不是一条射线，向量有2D、3D、4D等的。在游戏开发里面一般使用的是2D向量和3D向量。2D向量表示为<x，y>两个数值，而3D向量是由<x，y，z>三个数值表示的。3D游戏开发中经常使用的3D向量的几何表示如图1-5所示。

{% asset_img 5.jpg %}

