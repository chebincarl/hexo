---
layout: title
title: Animator中的Transition
date: 2019-05-25 11:59:22
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题:
1.导入Standard Assets中的Character包，看看里面的Transition是如何设置的。

<!--more-->

{% asset_img 1.png %}

# Transition

Transition代表状态之间的切换条件，一般会有一个或多个条件，用于从一个状态切换到另一个状态。

## 添加Transition

在一个State上右键，在弹出菜单中选择Make Transition，可以创建一个到其他State的Transition。

{% asset_img 2.gif %}
<center><font color="gray">增加Transition</font></center>

点击代表Transition的箭头，可以在Inspector上看到这条Transition的具体情况。选中Transition的源State（从哪个State出发），也可以在State的Inspector中看到这条Transition的具体信息。

{% asset_img 3.png %}

Transitions 显示当前选中的Transition。后面有两个复选框包括Solo和Mute。

Solo 如果两个State之间有多条Transition，勾选这个选项后，只有选中Solo的Transition生效。其他Transition会被禁用。

{% asset_img 4.png %}
<center><font color="gray">比如Transition1设置为Solo，则从源State到目的State的3个Transition中只有1会生效</font></center>

比如Transition1设置为Solo，则从源State到目的State的3个Transition中只有1会生效

Mute 勾选这个选项后，该条Transition会被禁用。如果同时选中了Solo和Mute，Mute会优先生效。

{% asset_img 5.png %}

Name Field 名称框。如上图所示，可以给Transition命名，用于区分两个State之间的多个Transition时非常有用。

Has Exit Time 是否有退出时间条件。退出时间是一种特殊的transition条件，它没有依赖参数（下面会讲），而是根据设置的退出时间点作为条件进行状态转换。

Settings transition的一些参数设置。

Exit Time 如果勾选了Has Exit Time，该参数是可以设置的，设置动画退出的单位化时间。例如设置为0.75，代表动画播放到75%时为true，如果没有其他条件，会直接切换到下一个State。

对于循环的动画，如果exit time小于1，那么每次循环到对应位置的时候，该条件都会为true。比如第一次播放到75%，第二次播放到75%……时退出条件都会为true。

如果exit time大于1，该条件只会检测一次。比如exit time为3.5，state的动画会在循环3次后，在播放到第4次的50%时为true。

Fixed Duration 勾选时，下方Transition Duration参数的单位是秒，不勾选时，参数会作为一个百分比。

Transition Duration transition的过渡时间。两个状态在转换时，一般不会瞬间从一个状态转换到另一个状态，而是会经过平滑混合，这个属性就是设置了平滑混合的时间。可以从下图的两个蓝色箭头看出转换的时间。

{% asset_img 6.png %}

Transition Offset 目标状态开始播放的时间偏移。比如设置为0.5，则转换到下一个State时，会从50%的位置开始播放。

{% asset_img 7.png %}
<center><font color="gray">如图设置为0.5时，下一个State会从50%开始转换</font></center>

Interruption Source和Ordered Interruption 这两个参数可以用来控制transition的打断。下面会进行详解。

## Transition图

上面的参数不仅可以手动修改数值，也可以通过Transition图预览、修改。

{% asset_img 8.png %}

## Conditions 条件 

一个Transition可以有一个条件，也可以有多个条件，甚至没有条件。

{% asset_img 9.png %}

如果Conditions中没有条件，但是勾选了Has exit time，那么exit time会被作为state退出的条件，到达exit time时，会切换到下一个state。

如果有一个或多个条件，需要同时满足这些条件才能切换下一个state。

一个条件可以是：

相等/不相等判断，一个参数等于/不等于一个常量时为true（int，float，bool类型参数）

比较判断，一个参数与一个常量的比较结果（int，float类型参数）

触发器，触发器激活时为true

如果Has Exit Time勾选了，并且transition还有一个或多个条件，那么transition需要同时满足到达exit time同时条件全为true，才会切换到下一个state。

一个transition至少要有一个条件（Has Exit Time可以作为一个条件），否则transition会被忽略。

# 【选读】Transition Interruption

之前我们提到了Interruption Source和Ordered Interruption 这两个参数可以用来控制transition的打断。那么究竟什么是transition打断呢？

一般情况下，动画系统的transition是不能打断的：一旦transition开始从一个state切换到另一个state，没有打断的方法。就像乘坐跨大西洋航班的乘客一样，你舒适地坐在座位上，直到到达目的地，无法改变主意。对于大多数用户来说，这很好。

但是如果你需要对transition进行更多控制，可以通过多种方式配置动画系统来满足需求。如果你对目前的目的地不满意，你可以跳进飞行员的座位，在飞行途中改变计划。这意味着更具响应性的动画，但也有很有可能迷失在复杂的打断中。

我们通过几个例子来解决这个问题。我们从一个相当简单的状态机开始，这个状态机具有四个状态，标记为A到D，并且使用trigger作为每个transition的条件。

{% asset_img 10.png %}

默认情况下，当A到B的切换触发后，状态机开始切换到B，在切换到B之前无法被改变。但是，如果将A->B的transition的interruption source属性从None切换到Current State，A到B的切换就可以被A上的一些触发器中断。

{% asset_img 11.png %}

为什么只有一些呢？因为Ordered Interruption属性默认也会被勾选。这意味着只有优先级大于当前的transition才能打断。选中State A，在Inspector中查看，我们看到A -> C的优先级高于 A -> B，那也意味着只有A -> C能打断A -> B的转换。

{% asset_img 12.png %}

如果我们激活AtoB的trigger，然后立马激活AtoD的trigger，A到B的transition不会被打断。但是，如果我们激活AtoB的trigger，然后立马激活AtoC的trigger，A到B的transition会被打断，转而切换到C。

在动画系统内部，会记录下被打断时的动画的状态，然后从打断的状态混合到新的目标动画。

{% asset_img 13.png %}

如果不勾选Ordered Interruption属性，会发生什么情况呢？A->C 和 A->D 都能打断 A -> B 的transition了。但是，如果在同一帧激活了AtoC和AtoD的trigger，A->C仍然会优先激活因为A->C的优先级更高。

如果将A -> B的interruption source属性改为Next State，A->C 和 A->D就不能打断A -> B了。如果我们激活AtoB的trigger，然后立马激活BtoD的trigger，A到B的transition会被打断，转而切换到D。

B上的Transition的顺序也有影响。但是这时候Ordered Interruption属性就无法勾选了（因为A -> B是在State A上不在State B上，不参与B的排序）。B上transition的顺序会决定同时触发时，会使用哪一个transition。例如下图的排序，如果B->D 和B->C在同一帧被触发，B->D的transition会被执行。

{% asset_img 14.png %}

如果想完整控制，我们可以设置interruption source属性为Current State Then Next State或Next State Then Current State。设置为这两个值时，State A和State B上的transition都会被考虑在内。例如设置如下，选中了Current State Then Next State：

{% asset_img 15.png %}

如果A到B切换时，同时激活的A->C, A->D, B->C和B->D，会发生什么情况？

如果选中了Ordered Interruption，那么首先可以忽略A->D（因为比A->B）的优先级低。然后先考虑Current State A，那么A->C会胜出，甚至不用考虑Next State B了。

{% asset_img 16.png %}

如果同样的配置，只激活了B->C 和 B->D，那么B->D会胜出，因为B->D的优先级比B->C更高。

** 小结 **

上面我们只使用了A->B一种情况作为例子进行了讲解，其他的中断都是类似的，只需要根据他们自身的规则即可。

有一点很重要需要记住的是：不管打断发生了几次，只要transition没有完成，source state会一直不会变。比如A->B被B->C打断，又被C->D打断，transition未完成前source state会一直是A。Animator.GetCurrentAnimatorStateInfo()也会返回State A。

简而言之，transition中断功能很强大，并提供了很大的灵活性，但会变得非常混乱。因此，合理地使用transition中断，如有不确定，一定要在编辑器中进行测试。

# 总结

* Transition代表状态之间的切换条件，一般会有一个或多个条件，用于从一个状态切换到另一个状态。
* 一个transition至少要有一个条件（Has Exit Time可以作为一个条件），否则transition会被忽略。