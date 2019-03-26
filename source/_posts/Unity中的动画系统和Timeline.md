---
layout: title
title: Unity中的动画系统和Timeline
date: 2019-02-17 18:18:56
tags: Unity
---

# 普通动画
新建项目AnimationProject，在Project视图下新建Scenes、Scripts、Prefabs、Animations文件夹。保存场景为01-Normal Animation，在场景中创建一个Cube。可以给任意类型的游戏物体施加动画，每个组件基本上都是可以控制的。
选中Cube，选择Window里面的Animation，出来一个专门创建动画的窗口，如下图所示。

<!--more-->

{% asset_img 1.png %}

Animation Clip就是一个动画的文件。点击Create，动画要干什么就取相应的名字，取名为CubeMove，后缀为anim，保存到Animations文件夹中。此时Cube多了一个Animator组件，如下图所示。

{% asset_img 2.png %}

Animator是用来播放状态机的，Animator的Controller属性就是状态机，Animation是动画。
CubeMove是动画，Cube是Animator Controller，是一个状态机，这个状态机里面管理了CubeMove。一个动画可看做是一个状态。

{% asset_img 3.png %}

新建一个AnimatorControllers文件夹，把Cube放进此文件夹中。

{% asset_img 4.png %}