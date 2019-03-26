---
layout: title
title: Unity2D游戏开发
date: 2019-03-24 23:50:40
tags: Unity
---
Unity 2D功能主要包括2D游戏对象Sprite、2D游戏换帧动画图片的制作工具以及2D游戏物理引擎。

<!--more-->

1.2D游戏对象Sprite
Unity为2D游戏开发提供了专门的2D游戏对象Sprite。Sprite对象上有一个Sprite Renderer组件用于Sprite对象的渲染，该组件用一张需有Alpha通道的图片渲染到平面上。此外，Sprite Renderer组件上专门提供了一个设置2D游戏遮挡层的参数。

2.2D游戏换帧动画图片的制作工具
Unity专门提供了一个制作2D游戏换帧动画图片的工具--Sprite Editor，该工具可以将带有动画每帧图片的一张整图片分割成每帧一张图片，通过该工具就可以将包含Sprite动画每帧图片的整张图片制作成2D换帧动画。

3.2D游戏物理引擎
Unity为2D游戏开发集成了Box2D物理引擎并提供了一系列的2D物理组件，通过这些组作可以非常简单地在2D游戏中实现物理特性。这些组件可以添加到Sprite对象上，使Sprite对象具有某种物理特性（如刚体等）。

Unity 2D游戏开发工作流程

(1)首先需要创建一个2D项目。打开2D项目场景中的摄像机（默认使用平行投影）。Scene界面默认选中2D选项，该界面显示了一张平面网格作为显示在屏幕上的显示范围，如下图所示。

(2)创建Unity 2D游戏开发的核心对象Sprite。创建一个Sprite对象的具体步骤为“GameObject->2D Object->Sprite”，如下图所示。这时场景的平面中就会出现一个Sprite对象，它可以和3D物体一样可以进行平移、旋转和缩放，只是对Z轴的操作无效。

(3)为Sprite对象指定贴图。首先需要在项目中导入一张带有Alpha通道的贴图，设置贴图的Texture Type为Sprite，如下图所示。将Sprite对象的Sprite Renderer组件的Sprite参数设置为该贴图，这时场景中的Sprite对象就会出现这张贴图。

(4)通过不断改变Sprite Renderer组件的Sprite参数实现动画的换帧效果，通过脚本控制Sprite对象就可以开发出2D游戏。

# Unity 2D核心功能对象-Sprite

## Sprite对象的创建和基本用法

1.Sprite对象的创建
Sprite对象的创建方法有两种。
一种是通过依次单击“GameObject->2D Object->Sprite”来创建一个空的Sprite对象，这种方法创建的Sprite对象包含一个Sprite Renderer组件（用于在场景中渲染这个Sprite对象），但是这个Sprite Renderer组件的Sprite参数是空的。


另一种创建Sprite对象的方法是在项目中导入一张带有Alpha通道的贴图，设置贴图的Texture Type为“Sprite”，然后将这种贴图拖曳到场景中，这时场景中就会出现带有这种贴图的Sprite对象。通过这种方法创建的Sprite对象的Sprite Renderer组件的Sprite参数就是这种贴图。

2.Sprite对象的基本用法

创建出来的Sprite对象和3D对象一样可以进行平移、旋转和缩放等基本变换，只是因为Sprite对象是2D对象所以对Z轴的基本变换无效。Sprite对象通过Sprite Renderer组件绘制在屏幕上，而绘制的是Sprite参数设置的带有Alpha通道的贴图。

Unity专门提供了Sorting Layer用于设置Sprite对象之间的遮挡关系。通过在Tag&Layers界面的Sorting Layer中添加层，并且通过拖曳调整层遮挡的前后关系，然后设置Sprite Renderer组件的Sorting Layer参数，将Sprite对象放在某个层中。

## 换帧动画的制作

1.换帧图片的制作
换帧动画制作之前首先需要制作换帧动面所需的图片。使用图片制作软件制作出包含每帧换帧贴图的整张图片，然后将图片导入到项目中，设置图片的Texture Type为“Sprite”，设置Sprite Mode为“Multiple”，如下图所示，表明该图片包含多帧换帧贴图。

换帧图片中的每帧贴图通过Sprite Editor工具分离出来。点击Sprite Editor打开Sprite Editor工具，如下图所示，Unity默认通过图片Alpha通道的值自动分割贴图，但是系统默认切割的可能不符合实际的要求，因此可以通过手动调整每个分离框的大小和位置调节分割的贴图。

2.换帧动画的制作
Unity 2D中制作换帧动画的方法有多种，但原理基本一样，都是通过定时改变Sprite Renderer组件的Sprite贴图来实现换帧动画的。常用的一种方法是通过脚本定时改变Sprite Renderer组件的Sprile参数来实现。

Unity专门提供了一个制作动画的工具可以制作换帧动画。将制作换帧需要的贴图同时选中，一起指曳到场景中就会弹出保存该换帧动画的对话板，保存完成后就会在场景中出现一个包含拖曳到场景中的贴图的换帧动画的精灵。

选中该精灵，依次单击“Window->Animation”就会弹出修改该动画的Animation工具，如下图所示。通过Animation工具可以修改换帧动画的每帧贴图和换帧动画的播放速度，如果有需要还可以添加帧和修改帧的顺序。

## 制作换帧动画的具体步骤
(1)新建一个2D项目，创建一个场景，命名为“text”。
(2)创建背景图片精灵。将Assesisprite件夹中的background图片文件导入到项目中,设置background图片的Texture Type为"Sprite",设置Sprite Mode为"Single".其他设置如图14-10所示。


(3)将backgroundI图片拖变到background精尺对象的Sprite Renderer组件的Sprite参数上.单击Sprite Renderer姐件的Sorting Layer选项,单击"Add Sorting Layer..最加层,如图1411所示。

(4)显示出TapsdLayers界面后,在Sorting Layers中点击“4"添加两个层,上面一个层命名为"beiingn"作为背景层,下面一个层命名为"wui"作为物体对象层,如图14-12所示。层位置越往上越深,下面的层遠挡住上面的层.