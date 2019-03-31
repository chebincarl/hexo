---
layout: title
title: UGUI综合示例--音乐播放器的UI搭建
date: 2019-03-31 21:30:33
categories: Unity
tags: Unity游戏开发技术详解与典型案例
---
本章将给出一个由UGUI系统搭建的音乐播放器的UI。

<!--more-->

{% asset_img 1.png %}

(1)首先导入需要用到的图片资源。图片列表以及图片用途如下表所示。

{% asset_img 7.png %}

{% asset_img 2.png %}

(2)需要注意的是，导入的图片类型是PNG类型的，在使用前需要在Inspector面板中设置为精灵模式Sprite(2D and UI)，这样才能通过Image控件正常显示出来。

{% asset_img 3.png %}

(3)搭建UI。下面将用表格的形式介绍场景中Canvas下的MusicPlayer及其子对象，可以按照下表中的内容及层级关系依次进行创建。

{% asset_img 4.png %}

{% asset_img 5.png %}

(4)按照上面的表格内容创建MusicPlayer游戏对象及其子对象，为每个控件赋予相应的贴图后，在Game窗口中应该可以看到下图所示的UI。其子对象结构如下图所示。下面将对部分特殊控件的设置进行介绍。

{% asset_img 6.png %}

在搭建MusicPlayer时，若子对象中有Image组件，需要赋予贴图，每个控件对应的贴图可以自行查表。

(5)首先将图4-105中所示的Canvas组件中的Render Mode设置为"World Space",如图4-106所示。这样,该画布就可以在3D场景中进行旋转等变换了。然后将SoundBT下的子对象BackGroundMask中的Image组件的颜色设置为黑色,透明度设置为半透明。如图4-107所示。

```cs

```