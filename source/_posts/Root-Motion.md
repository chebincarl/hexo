---
layout: title
title: Root Motion
date: 2019-05-27 14:55:21
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.试下动画加上不同的Root Motion设置以后会有什么不同的效果。”

<!--more-->

“智哥，自从用了混合树来做人物移动，腰不酸腿不疼，思路更清晰了，一口气能写12小时代码！”
“哟，疗效这么好，我看你应该再码12个小时”
“那也没问题，你来看看我做的这个人物的混合树是不是棒极了”

小新信心满满地打开混合树，给大智看。

“嗯，这个混合树确实做的不错，比不过你这个角色还不能用啊！”
“怎么不能用了，你看这个走起来不是走的好好的么？”
“你走到那个坡上去试试”

小新操作人物往坡上走，只见直接穿了进去。
“哎？我好像忘了给人物加碰撞了，等我给他加上”

小新给角色加上Collider和Rigidbody组件，再次操作人物往坡上怼过去。
“智哥，你看现在OK了吧?”
“别急，你再下坡看看”
“哎，这怎么不受重力啊，怎么不会掉下去！明明加了刚体组件啊”
“这个是因为动画在控制人物的Y轴，所以才没有掉下去”
“但是我不都设置Apply Root Motion了么，咋还这样呢？”
“那你知道Root Motion到底是什么？”
“那不就是……人物的动画会带动人物移动嘛”
“这只是他的一个表现，这时候你该去好好理解下Root Motion到底是什么了，这个Unity的文档中有，你去好好看一下吧！”
“好嘞”

下面是小新的学习笔记。

Root Motion
首先要分清Body Transform和Root Transform。

# Body Transform（身体变换）

Body Transform是角色的质心（重心）。用于Mecanim系统的重定向引擎中来提供稳定的模型移动。身体朝向是角色模型在T姿势下上身和下身朝向的平均值。

Body Transform和朝向存储在Animation Clip中，这两个是Animation Clip中存储的唯二世界空间的曲线，其他的动画曲线都是以相对body transform的形式存储的。

> T-Pose T姿势
角色模型的胳膊向外伸直，整个身体呈T字型，一般的角色模型应该为这个姿态。

{% asset_img 1.png %} 
<center>T-Pose</center>

# Root Transform（根变换）

Root Transform是body transform在Y平面上的投影，并且是运行时计算的。每一帧Root Transform的变化实时计算。然后Transform的变化会被应用到GameObject上从而让物体移动。

{% asset_img 2.png %}
<center>角色脚下的圆圈代表了Root Transform</center>

# 调整Root Transform

通过对Animation Clip的设置来控制Body Transform投影到Root Transform的结果。

{% asset_img 3.png %} 

可以调整的有Root Transform Rotation, Root Transform Position (Y) 和 Root Transform Position (XZ) 。基于设置，Body Transform的部分数值可以转移到Root Transform中。例如你可以选择动画中的Y曲线是Root Motion的一部分，还是Body Transform（pose）的一部分。

## Root Transform Rotation

用于设置Root Transform的朝向（旋转）。

Bake into Pose：选中后，角色的朝向会基于body transform（Pose）。Root Orientation会是一个常量，意味着Animation Clip不会旋转这个物体。

只有AnimationClip的开始和结束位置旋转相似的时候，才应该使用这个选项。可以通过右边的绿灯判断。通常用于向前直行的走或跑的动画。

Base Upon：可以设置动画的朝向基于的地方。

Body Orientation：动画会朝向身体正前方。这个设置适用于大多数身体朝前的动画，比如走跑跳。但是如果动画是向左或向右平移的话，会有问题。这时候可以使用下面的Offset来调节角色的朝向。

Original：有的动画师会给动画手动加上旋转，确保动画的朝向正确，这时候可以使用这个选项，一般就不用再手动调整Offset了。

Offset：基于Base Upon的设置，调整偏移量。

## Root Transform Position (Y)

用于设置Root Transform位置的Y轴位置。

Bake into Pose：选中后，动画的Y轴的运动会保留在Body Transform（Pose）上。Root Transform的Y轴会是一个常数（不会受动画影响变化），也就是意味着动画不会改变物体位置的Y值。右边有一个绿灯指示动画起始位置和结束位置的高度是否一致，可以看出动画是否适合使用此选项。

大多数的动画应该选中此选项，除了那些会改变物体高度的动画比如跳起、跳下这些动画。

注意：Animator.gravityWeight是由Bake Into Pose position Y控制的。选中时gravityWeight = 1，不选中时gravityWeight = 0。gravityWeight用来在state转换时进行混合。

Base Upon：和Root Transform Rotation设置类似，除了Original 或 Mass Center (Body)选项外，还有一个Feet选项。Feet选项非常适合改变物体高度的动画（不勾选Bake Into Pose）。使用Feet时，Root Transform Position Y会匹配骨骼中脚部的Y位置（更低的那个）。Feet选项可以避免混合或转换时浮空的现象。

Offset：可以设置高度的偏移量。

## Root Transform Position (XZ)

用于设置Root Transform位置的XZ轴位置。

Bake Into Pose：通常用于原地不动的动画（动画在XZ轴上的位置为0）。可以用来去除动画循环累计的误差，造成位置的移动。也可以通过设置Based Upon Original来强制使用动画师设置的位置，否则会使用角色的重心作为Root。

## Loop Pose

Loop Pose（比如混合树或Transition中的混合）会基于Root Transform。Root Transform在每帧被计算出来后，动画的位置会相对Root Transform。开始帧和结束帧的差别会被计算出来，然后分布到动画的0-100%。完全没看明白，得问大智了

## Generic Root Motion

Generic和Humanoid基本是类似的，但是Generic的动画的Root Transform是手动设置的Root Node属性。

“大智，我看完Root Motion的文档了，不过还是有点云里来雾里去的，你能不能用简单的几句话说说设置Root Transform的作用是什么？”
“简单来说，<span style="color:red;">如果不设置Root Transform中的Bake Into Pose，动画中的曲线会影响物体的Root Transform，而勾选了Bake Into Pose以后，动画的曲线就不会影响物体的Root Transform。再直白点说，比如勾选了Root Transform Position (Y)的Bake Into Pose，那动画就不会影响物体的Y轴位置了。</span>对于你遇到的刚体不会掉落的问题，也能解决了。”

“emmm，大概能明白，我还是得去试一试看看不同的效果。第二个问题：什么是重定向？”
“重定向就是把A角色做的动画用到B角色上。如果A和B两个角色的骨骼结构完全一样，那动画可以直接重用。但是如果A和B的骨骼结构不一样，但是是Humanoid类型的，可以使用Unity中的Retargeting系统，这个文档里也有，建议你先去看看。”

“哦，我现在貌似还用不到，等我用到的时候去查一下。最后一个问题哈，Loop Pose的作用是什么？我看了半天也没看明白是什么意思”
“看不明白很正常，文档中那个解释确实有些不太直观。这个作用是，<span style="color:red;">如果一个循环动画的首尾帧有差别，选中这个选项Unity会给你插值，让首尾帧看起来是连贯的，循环起来没有缝隙，但是可能会看起来有些奇怪。不过我们使用的动画，一般动画师都会做成无缝循环的动画，所以这个选项也不经常使用。</span>”
“这么一说我就能明白很多了，看来文档也有不靠谱的时候啊”

# 总结

文档有时确实有描述不太准确或者晦涩的时候，这时候就需要你多动手。你看这个知道的‘知’字，左边是矢，也就是箭矢，右边是口，靶子也就是目标。这个字是说：不断地练习，命中目标才是知。所以要多实践。