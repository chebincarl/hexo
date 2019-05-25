---
layout: title
title: Animator
date: 2019-05-25 11:53:27
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.导入Standard Assets中的Character包，看看里面的Animator Controller是如何设置的。

<!--more-->

{% asset_img 1.png %}

前两天我们主要学习了Animation Clip，是整个动画系统的基本元素。
如果把Animation Clip比作是一段视频的话，那么Animator就是一个视频播放器，用来控制多段视频的播放、切换等等。

# Animator组件

{% asset_img 2.png %}

在一个物体上播放动画，需要添加Animator组件到这个物体上。

Animator中有一个很重要的属性是<span style="color:blue;">Controller</span>，这个属性引用了一种叫<span style="color:red;">Animator Controller</span>的资源，这种资源以文件的形式存储在工程中，文件内存储了动画的各种状态以及状态之间的切换规则。下面会细讲。

<span style="color:blue;">Avatar</span> 用于人形动画，设置使用的骨骼节点映射。

<span style="color:blue;">Apply Root Motion</span> 应用根节点运动。如果不启用，动画播放时会保持在原地，需要通过脚本控制物体的移动。如果启用，如果动画中有位移，那么Animator组件所在的物体也会移动。

<span style="color:blue;">Update Mode</span> 设置Animator更新的时机以及timescale的设置。

* <span style="color:blue;">Normal</span> Animator按正常的方式更新（随着Update调用更新，timescale减小时，动画播放也会减慢，timescale的具体含义和用法后续会详解）

* <span style="color:blue;">Animate Physics</span> Animator会按照物理系统的频率更新（根据FixedUpdate调用更新，后续会详解），适用于物理交互，例如角色加上了物理属性可以推动周围的其他物体。

* <span style="color:blue;">Unscaled Time</span> 根据Update调用更新，无视timescale。一般用于UI界面，当你使用timescale暂停游戏时，界面保持正常动画。后面会详解。

<span style="color:blue;">Culling Mode</span> 裁剪模式

* <span style="color:blue;">Always Animate</span> 动画一直运行，即使物体在屏幕外被裁剪掉并没有渲染

* <span style="color:blue;">Cull Update Transforms</span> 当物体不可见时，禁用Retarget、IK、Transforms的更新（后续动画进阶模块会细讲）

* <span style="color:blue;">Cull Completely</span> 当物体不可见时，完全禁用动画

# Animator Controller

Animator Controller是Animator组件必须的资源，这种资源以文件的形式存储在工程中，文件内存储了动画的各种状态以及状态之间的切换规则。

{% asset_img 3.png %}
<center>图中有两个Animator Controller文件</center>

通常一个物体上有不止一段动画，使用Animator Controller可以很容易地管理各段动画以及动画之间的切换。比如角色身上有走、跑、跳、蹲的动画，使用Animator Controller可以很容易管理它们。不过，即使只有一段动画，仍然需要给动画物体添加Animator组件才能播放动画。

Animator Controller中使用了一种叫State Machine（状态机）的技术来管理状态以及状态之间的切换。

{% asset_img 4.png %}

# StateMachine 状态机

状态机由State（状态）和Transition（转换）组成。State代表一个状态，在Animator Controller中一个State可以包含一段动画、一个子状态机或一个混合树（后面会细讲）。Transition用来设置状态之间的切换条件，一般会有一个或多个条件，用于从一个状态切换到另一个状态。

{% asset_img 5.png %}

在Animator窗口中，可以可视化看到State以及Transition。

# 创建Animator Controller

创建Animator Controller资源有如下几种方式：

在Unity中创建Animation Clip时，如果选中的GameObject上没有Animator组件，会自动添加Animator组件并在工程中创建一个Animator Controller文件（和Animation Clip文件同目录）。

将任意Animation Clip拖到一个物体上时，如果拖到的物体上没有Animator组件，会自动添加Animator组件并在工程中创建一个Animator Controller文件（和Animation Clip文件同目录）。

可以在Project窗口中手动创建Animator Controller文件，如下图所示：

{% asset_img 6.png %}
<center>创建Animator Controller文件</center>

# 编辑Animator Controller

双击Animator Controller文件，可以打开Animator窗口，编辑该文件。

今天我们先简单学习一下如何将导入的动画播放出来，后续的动画进阶模块会更详细讲解Animator Controller中的高级功能。

在Project窗口中直接创建Animator Controller时，其中是不包含任何动画的。如下图所示：

{% asset_img 7.png %}

图中包含三个节点：

<span style="color:blue;">Entry</span> 入口。动画状态机会从这个节点开始，根据Transition进入一个默认State。

<span style="color:blue;">Any State</span> 任意状态。用于从任意状态转换到特定状态。比如射击类游戏中，如果被子弹打中后，不管当前处于什么状态，都会倒地死亡。

<span style="color:blue;">Exit</span> 退出状态机。一般用于嵌套的状态机的退出。

## 添加状态

可以在空白处右键添加Empty State，也可以将Animation Clip文件拖到Animator窗口中添加一个State。

{% asset_img 8.png %}

如果当前在Project窗口选中了一个Animation Clip，也可以通过上图的From Selected Clip创建一个State，不过还是直接将Clip拖到Animator中创建State更简单。

{% asset_img 9.png %}

第一个创建的State默认是橘黄色的，代表是默认状态。有一条黄色的箭头从Entry指向橘黄色的State。Animator组件会在一开始播放New State，如果New State中有动画，也会播放对应的动画。

这时候如果你Play这个场景的话，设个物体就会播放默认State的动画。

## State设置

每个State可以包含一段Animation Clip，处于该State时Animator组件所在的物体会播放该动画。选中一个State时，在Inspector中可以看到如下内容：

{% asset_img 10.png %}

<span style="color:blue;">Motion</span> 可以设置一个Animation Clip，如果是从Animation Clip创建的动画，这里应该已经有动画了，你也可以从工程中选择动画。

<span style="color:blue;">Speed</span> 动画的播放速度

<span style="color:blue;">Multiplier</span> 乘数，可以使用一个参数来控制动画的播放速度，动画最终的播放速度会是Speed * Multiplier。后面会讲解Animator的参数以及如何在代码中控制参数。

<span style="color:blue;">Normalized Time</span> 单位化时间，范围是0-1，需要使用参数控制。

<span style="color:blue;">Mirror</span> 镜像动画。也可以使用一个参数控制。

<span style="color:blue;">Cycle Offset</span> 循环偏移量。可以用来同步循环的动画。偏移量使用的是单位化时间，范围是0-1。也可以使用参数来控制。

<span style="color:blue;">Foot IK</span> 只用于人形动画。角色的脚是否使用反向动力学。

<span style="color:blue;">Write Defaults</span> 是否初始化该State没有用到的参数为默认值。

<span style="color:blue;">Transitions</span> 该状态参与的状态转换。下面会细讲。

## Parameters 参数

上面我们提到了参数的概念，那么参数是什么呢？

{% asset_img 11.png %}

Animator Controller中的参数可以作为控制transition切换的条件，也可以控制上面可以参数化的属性比如State中的几个属性。

{% asset_img 12.png %}
<center>State中可以用参数做属性值的来源</center>

Animator Controller的参数可以通过代码进行控制，进而控制整个Animator状态机的运转。

参数共有4种类型：

* <span style="color:blue;">Int</span> 整数类型
* <span style="color:blue;">Float</span> 浮点数（小数）类型
* <span style="color:blue;">Bool</span> true或false（真或者假，用于逻辑判断），界面上显示为复选框
* <span style="color:blue;">Trigger</span> 触发器，与Bool有点类似，但是transition在使用这个参数后会被自动设置为false状态。界面上显示为一个圆形按钮。

# 总结

记住以下几点：
* 如果把Animation Clip比作是一段视频的话，那么Animator就是一个视频播放器，用来控制多段视频的播放、切换等等。
* Animator Controller就是一个剧本，用来指导视频播放器如何播放多段视频。