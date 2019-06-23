---
layout: title
title: NavMeshAgent
date: 2019-06-10 17:54:13
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.让角色在场景中寻路的步骤是什么？
2.GetComponent<NavMeshAgent>().destination是什么意思？

<!--more-->

NavMesh烘焙好了，接下来学习NavMeshAgent组件，让角色动起来，参与寻路的角色都需要添加这个组件。

{% asset_img 1.png %}

# <span style="color:#039BE5;">NavMeshAgent组件</span>

场景的NavMesh烘焙好之后，就可以让角色在场景中寻路。一般的步骤如下：

1、选中或创建一个角色，在测试时可以创建一个圆柱体：GameObject > 3D Object > Cylinder，并设置好角色的尺寸。

2、给角色添加NavMeshAgent组件并设置组件的相关属性（简单测试时可以保持默认值）

{% asset_img 2.png %}

3、现在可以通过脚本设置NavMeshAgent的目的地，示例代码如下：
```cs
using UnityEngine;
using UnityEngine.AI;

public class PlayerController : MonoBehaviour {
    private NavMeshAgent _agent;

    void Start ()
    {
        _agent = GetComponent<NavMeshAgent>();
        _agent.destination = Vector3.one;
    }
}
```
通过设置NavMeshAgent组件中的destination属性，可以设置角色的运动目的地。

## <span style="color:#00ACC1;">NavMeshAgent组件属性</span>

NavMeshAgent组件主要有以下作用：
1、通过NavMesh的数据进行寻路，移动到目标位置
2、NavMeshAgent之间会互相躲避
3、躲避动态障碍物

{% asset_img 3.png %}

<span style="color:blue;">Agent Type</span> Agent类型。通过Navigation窗口的Agents页签进行设置，可以设置多种类型的Agent。设置后，在组件的Agent Type属性中可以通过下拉框选择。

{% asset_img 4.png %}

** Agents设置 **

设置Agents类型时，一般根据角色的尺寸进行分类设置。比如人形角色、巨人族、怪兽等等。

<span style="color:blue;">Base Offset</span> Agent的偏移量，调整这个值让Agent能适合角色的偏移。

{% asset_img 5.png %}
<center><font color="gray">图中的圆柱形网格代表了Agent的尺寸及偏移</font></center>

** Steering 移动参数 **
<span style="color:blue;">Speed</span> 最大移动速度（m/s）
<span style="color:blue;">Angular Speed</span> 旋转速度（角度/秒）
<span style="color:blue;">Acceleration</span> 最大加速度（米/秒^2）
<span style="color:blue;">Stopping Distance</span> 距离目标位置为这个位置后停止运动。
Auto Braking 选中时，agent在即将到达目标位置时会减速。但是有些情况应该禁用，比如agent在多个目标点之间巡逻。

** Obstacle Avoidance 障碍躲避 **
<span style="color:blue;">Radius</span> 用来计算和其他障碍物和Agent碰撞的半径
<span style="color:blue;">Height</span> 用来计算和其他障碍物和Agent碰撞的高度
<span style="color:blue;">Quality</span> 障碍躲避的质量。如果场景中有很多Agent，可以降低质量来减少CPU的占用。如果将躲避设置为None，Agent不会自动躲避障碍物，会直接装上去。
<span style="color:blue;">Priority</span> 比当前Agent的优先级低的Agent在躲避时会被忽略。这个属性的范围是0-99，数字越小代表优先级越高。

** Path Finding **
<span style="color:blue;">Auto Traverse OffMesh Link</span> 选中时会自动通过OffMeshLink的位置。如果你想要在通过这些位置时播放动画或进行特殊操作，不要勾选此选项。

<span style="color:blue;">Auto Repath</span> 选中时，Agent到达部分路径的尽头时会自动尝试去找接下去的路。如果没有路径能到达目标位置，会生成一个部分路径到达目标位置的最近位置。

<span style="color:blue;">Area Mask</span> NavMesh中可以给不同的位置进行分类。这个属性可以定义agent可以在哪些区域寻路。比如你可以将楼梯设置为特殊的区域类型，从而禁止一些agent使用楼梯。

## <span style="color:#00ACC1;">Navigation Area</span>
可以给NavMesh定义多个Area（区域），每个区域也可以设置一个Cost用来指示在这个区域行走的代价。Cost越高代表这个区域越不好走，agent在寻路时会倾向寻找代价更低的路线。

比如：

有水的区域行走就会比较慢，通行Cost就会更高

门只有特定的角色才能通过，比如人类可以通过，但是僵尸就不能。

{% asset_img 6.png %}

1、在Navigation窗口的Areas中设置区域类型

{% asset_img 7.png %}

2、在Navigation窗口的Object中设置选中物体的Navigation Area


# <span style="color:#039BE5;">疑难解答</span>

Navigation系统中涉及到Agent Size的地方有三个：

Navigation窗口Bake页签中的Baked Agent Size

{% asset_img 8.png %}

这个Size设置仅用来设置烘焙NavMesh时所使用的Agent的尺寸，和具体的Agent无关。但是这个设置会影响NavMesh信息的生成，例如在这设置了Radius很大，那么对于小Radius的Agent就会支持不太好。所以这个Size设置时，要考虑到所有的Agent，取值要能覆盖到所有的Agent。Radius取所有Agent最小值，Height取所有Agent最小值，Slope取所有Agent最大值，Step Height取所有Agent最大值。

Navigation窗口Agents页签中的Agent Types中的Agent设置

{% asset_img 9.png %}

这里面的Agent设置是针对同一类的Agent，用于Agent在NavMesh上导航时的设置，相当于在静态物体上导航时的设置。

NavMeshAgent组件中Obstacle Avoidance中的Size设置。

{% asset_img 10.png %}

这个设置只会影响NavMeshAgent和其他Agent以及动态障碍物碰撞的计算。

# <span style="color:#039BE5;">总结</span>

导航系统本身是个很复杂的系统，Unity为了简化这个流程已经做了很多工作了