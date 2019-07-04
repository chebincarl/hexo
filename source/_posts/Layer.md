---
layout: title
title: Layer
date: 2019-07-04 21:28:24
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
* Layer有什么作用？应用的场景是什么？如何创建Layer？
* Weight是什么意思？0和1有什么区别？
* Mask上的M小图标是什么意思？
* Layer可以和其他层混合吗？
* Avatar Mask是什么意思？如何创建？两种定义身体遮罩的方式分别是什么？
* 身体映射图包含几个部分？绿色和红色分别代表什么？
* 手部和脚部的IK是什么意思？
* 使用Mask的好处是什么？
* 什么是正向动力学？反向动力学呢？
* Layer中的IK Pass是什么意思？
* Sync的作用是什么？

<!--more-->

本章涵盖：
* Animation Layers
	* Avatar Mask
	* Sync

Animator状态机里面的状态复杂了之后会变得很多，会太难管理。比如有很多姿势，不拿枪，拿手枪，拿冲锋枪，拿步枪，每种姿势都要做一个混合树，再去加蹲的状态，会十分繁琐。

{% asset_img 0.png %}

每个混合树里都有好多状态，这些动画之间有没有什么相似的地方可以提取出来呢？它们腿部的动画是类似的，不管拿枪姿势如何，腿部都是一样的，只有胳膊的动画不太一样。

这时候可以用到Animator Controller中的Layer。

# <span style="color:#039BE5;">Animation Layers</span>

<span style="color:red">使用Layer可以用来管理角色的不同身体部位。</span>比如下半身用于行走或跑步，上半身用于射击或投掷物体。

可以从Animator左上角的Layer标签中管理Layer。

{% asset_img 1.png %}

点击加号可以添加一个Layer。点击Layer旁边的齿轮图标，会弹出一个小窗口，可以设置该Layer对应的参数。

{% asset_img 2.png %}

<span style="color:red">Weight</span> 这一层的权重，0代表该层权重是0（该层不生效），1代表该层权重为1（该层中的动画能完全表现）

<span style="color:red">Mask</span> 设置该Layer能控制身体的哪部分，设置后该Layer上面会显示一个M的小图标。

<span style="color:red">Blending</span> 和其他层混合的模式。

* <span style="color:blue">Override</span> 覆盖上面的层中对应Mask的部位
* <span style="color:blue">Additive</span> 会加在之前Layer的动画之上

<span style="color:red">Sync</span> 复用其他层中的状态机。

<span style="color:red">IK Pass</span> 每帧会调用脚本中的OnAnimatorIK方法，可以在这个方法中动态设置IK

# <span style="color:#00ACC1;">Avatar Mask</span>

用来设置该Layer的动画会影响角色的哪一部分的遮罩。

## <span style="color:#EF7060;">创建Avatar Mask</span>

在Project窗口中右键，<span style="color:red">**Create ›Avatar Mask**</span>点击后会创建一个AvatarMask文件，此时可以对文件命名。

## <span style="color:#EF7060;">编辑AvatarMask</span>

AvatarMask有两种定义身体遮罩的方式：

* 通过Humanoid身体映射图
* 通过Transform层次结构选择包含或不包含的骨骼

** Humanoid **

如果你的动画是人形动画，建议用这一种方式可以快速设置AvatarMask。

{% asset_img 3.png %}

身体映射图包含以下几部分：

* 头部
* 胸腹
* 左臂
* 右臂
* 左手
* 右手
* 左腿
* 右腿
* Root（脚下的圆形阴影部分）

可以通过鼠标点击每一部分，绿色代表动画可以影响这一部分，红色代表动画不会影响。在空白处点击可以全选/全不选。

手部和脚部的IK可以开关，表示该部分的IK曲线是否参与动画混合。

**Transform**

如果动画没有使用Humanoid，或者你想更精细的控制遮罩，你可以通过Transform层级结构来控制每一个骨骼。

1、选择一个Avatar（动画模型的Avatar）
2、点击<span style="color:red">**Import Skeleton按钮**</span>。avatar的层级结构会显示出来。
3、可以设置对应的骨骼是否受动画控制

{% asset_img 5.png %}

**其他用途**

在模型导入设置的Animation页签中，也可以设置Mask，设置后只会导入对应Mask的动画数据。

**<span style="color:red">使用Mask的好处是：可以减少内存占用和CPU占用。因为不需要的身体部位的动画曲线不会被加载，并且不会参与计算。</span>**

## <span style="color:#EF7060;">实例</span>

比如你想人物保持行走、跑步、站立的同时上身能够投掷东西，你可以按下图设置Layer：

{% asset_img 4.png %}

# <span style="color:#00ACC1;">Sync</span>

有时能够在不同的Layer重用其他Layer的状态机非常有用。比如你想模拟一个“受伤”的状态，你有受伤状态的各种动画比如走跑跳。这时候你可以选中Sync复选框，选择你想要同步的Layer。<span style="color:red">状态机的结构会保持一致，但是可以设置不同的Animation Clip。</span>选中时，Layer旁边会有一个S小图标。

这意味着，同步出来的Layer不需要再去定义状态机，并且源Layer中状态机的任何变化都会同步到这一同步Layer。唯一需要你做的就是设置每个State中使用的动画。

Sync复选框旁边还有一个Timing复选框。

* 不选中时，Sync出来的Layer中每一个State的动画长度会变为源Layer中的时长。
* 选中时，根据Weight调整动画时长，Weight为1时使用Sync Layer中的动画时长。

{% asset_img 6.png %}

## <span style="color:#039BE5;">IK Pass</span>
<span style="color:red">**大多数动画是通过旋转骨骼来实现的。子骨骼的位置跟着父骨骼的旋转而改变，因此关节链的终点可以根据前面的各个骨骼的角度和相对位置确定。这种构成骨骼动画的方法称为正向动力学。**</span>

反向动力学就是根据骨骼的最终节点，反向推算之前的骨骼节点的位置。有些时候我们需要根据空间中的位置来确定骨骼节点的位置，比如让角色拿枪，不同的枪可能握持的位置不太相同，就需要根据握持的位置来决定角色手的位置。Unity中的IK支持所有人形动画。

Animator中的State设置中的Foot IK就是设置脚部受IK的影响。

# <span style="color:#039BE5;">总结</span>

有了Layer，动画可以分为上半身和下半身两个Layer，这样一来就不需要对每个拿枪的动作设置移动的混合树了，能简化不少工作。合理利用Layer不仅可以减少动画师的动画制作工作量，还能在性能上对游戏进行优化。

