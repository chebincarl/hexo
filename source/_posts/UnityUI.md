---
layout: title
title: UnityUI
date: 2019-03-24 23:48:44
tags: Unity
---
本章将利用Unity UI制作游戏中的主要UI界面，同时从中学习并掌握UI系统功能，包括如何在游戏场景中实现显示得分、生命条、暂停等按钮的功能。

<!--more-->

首先生成新场景，重命名为scMain，导入资源包UI_Textures.unitypackage。导入后，将生成的UI Textures文件夹移动到04.Images。

# Canvas对象
Canvas是Unity提供的游戏对象之一，所有游戏界面的UI元素（纹理、图像、按钮、滑动条等）都必须位于Canvas之下，成为其子对象。Canvas拥有的是Rect Transform组件，而并非其他游戏对象拥有的Transform组件。另外，一个场景中可以有多个Canvas对象，也可以导入其他Canvas对象作为当前Canvas对象的子节点。

要想生成Canvas对象，需要在菜单中选择GameObject->UI->Canvas，也可以点击层次视图中的Create菜单（层次视图中右击鼠标，在弹出菜单中选择生成）。生成Canvas对象后，EventSystem对象会自动生成。EventSystem对象可将系统中发生的键盘、游戏杆、触摸屏等输入信息传递给Canvas包含的UI元素。

## EventSystem对象

EventSystem对象包含EventSystem组件、Standalone Input Module组件等，其中EvenSystem组件的First Selected属性可以设置哪个UI元素将会第一个获得焦点。EventSystem对象很重要，如果没有它，UI元素就无法响应各种输入事件并执行相应动作。

Canvas对象包含Rect Transform、Canvas、Canvas Scaler和Graphic Raycaster这4个组件。与之前用到的游戏对象不同，UI元素必须拥有Rect Transform组件，它保存着锚点（Anchors）、枢组（Pivot）、大小（W，H）、位置（Pos X， Pos Y， Pos Z）、旋转以及比例等很多信息。简言之，Rect Transform组件可以理解为专用于UI界面的Transform组件。不可以直接修改Canvas对象的Rect Transform组件的属性，系统会根据画布大小自行设置。

## Canvas组件
Canvas组件可将游戏所需的各种UI元素放置到画面，并对其进行渲染。根据渲染模式的不同，UI元素的画面配置方式有如下几种。

1.Screen Space-Overlay
Canvas组件默认设置值，UI元素置于画面最表层，可以根据画面分辨率的设置自动调节位置，如图7-5和图7-6所示。