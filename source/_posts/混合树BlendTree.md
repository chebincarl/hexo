---
layout: title
title: 混合树BlendTree
date: 2019-05-27 14:17:09
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.混合树和Transition中的混合有什么区别？
2.1D混合是什么意思？
3.每个动画有不同的权重是什么意思？

<!--more-->

# BlendTree混合树

Animator中有一个功能，用来<span style="color:red;">解决多个动画之间的混合，经常用于移动动画之间的混合，这个功能叫做BlendTree，混合树。</span>

混合树和Transition中的混合不同，<span style="color:red;">Transition中的混合只是在两个State转换时，在给定的时间内进行混合，避免动画切换过于突兀。而混合树中的混合，是时时刻刻进行不同程度的混合。</span>比如你的角色有站立、走、跑三个动作，走路的速度是2m/s，跑的速度是5m/s，那你想让角色的速度是3m/s，这时候怎么办？这时候用混合树就能很简单地解决。

# 创建混合树

在Animator窗口的空白处右键，Create State > From New Blend Tree，双击这个节点可以进入混合树图。

{% asset_img 1.png %} 

混合树有3种类型，在右边的Blend Type中可以设置。分别为：
* 1D
* 2D
* Direct

## 1D混合

1D混合树应用的情景比较少，1D混合是根据一个参数进行动画混合。

1、首先要设置用于混合的参数，也就是从Animator的Parameters中的选择一个参数。

2、添加动画：可以点击小加号按钮，或者在Blend Tree节点上右键Add Motion。点击后会在Motion列表中添加一个条目，可以将Animation Clip拖进来。

{% asset_img 2.png %} 

添加完动画，整个混合的样子：

{% asset_img 3.png %} 

最上面的图显示了参数对每个动画的影响。每个动画显示为一个蓝色的三角形。如果点击这个三角形，会在下面的动画列表中高亮一下。<span style="color:red;">每个三角形的顶角位置定义了参数在该位置时，会完全使用这个动画，这个值也叫做该动画的阈值（Threshold）。</span>比如上图中的run动画，阈值是0.5，在混合图的中心位置。

图中的红线代表了参数的数值，主要是用来预览调试。可以拖动红线，在下面预览窗口观察动画是如何混合的。

{% asset_img 4.png %}
<center>注意点播放按钮，可以预览动画播放的状态</center>

** 参数范围 **

{% asset_img 5.png %}

上图中，左右两个数字代表了参数的范围。点击数字可以变成输入框修改，也可以在数字上拖拽调节。修改时会影响到第一个动画和最后一个动画的阈值。

** Threshold 阈值 **

修改动画对应的阈值可以直接拖拽对应的蓝色三角形。如果没有勾选Automate Threshold（自动计算阈值），也可以在阈值编辑框中直接输入数值。选中Automate Threshold（自动计算阈值）时，阈值会自动在最小值和最大值之间自动平均分布。

下面有一个Compute Thresholds下拉框，使用这个下拉框，可以根据动画中的数据，自动计算阈值。数据包括：speed（速度），velocity x、y、z（xyz三个轴分别的速度），angular speed（转动速度，单位是角度或弧度）。这些数据如何知道呢？

{% asset_img 7.png %}

比如：走路动画的速度是1.5m/s，跑的速度是4m/s，如果选择Compute Thresholds中的Speed，walk动画的阈值会被设置为1.5，run动画的阈值会被设置为4。

** Time Scale **

{% asset_img 6.png %}

通过动画速度这一列（图标是一个表）可以调节动画的播放速度，比如你想让跑步的动画播放速度变为原来的2倍，可以设置为2。

\*\*Adjust Time Scale > Homogeneous Speed \*\* 可以将动画的速度调整对应到参数的最小值和最大值，但是保持动画的初始相对速度。

按钮可以将动画的播放速度调整到动画列表中所有动画速度的平均值。

** Mirroring 镜像 **

{% asset_img 7.png %} 

上面复选框可以左右镜像一个humanoid类型的动画Clip。这个功能可以使用同一个动画创建出来两个方向的动画，可以节省一倍的存储空间和内存。

比如一个向左走的动画，通过镜像可以创建出一个向右走的动画。

# 难点解析

“第一个问题：自动计算Threshold的时候，那些动画的速度啊，旋转速度怎么知道啊？”
“选中一个Animation Clip，你可以看到这些数据，比如这个：”

{% asset_img 8.png %} 

“第一行是这个动画在xyz轴上的速度，第二行是旋转速度。”

“第二个问题：\*\*Adjust Time Scale > Homogeneous Speed \*\*这个到底是干嘛的？”
“先<span style="color:red;">将所有动画的平均速度算出来，然后通过调节动画的speed让所有动画的速度都一致。</span>”


# 总结

混合树可以根据参数，混合多个动画，每个动画有不同的权重，这样就有了很多中间状态。