---
layout: title
title: Mecanim动画系统（2）
date: 2019-04-16 09:19:24
categories: Unity
tags: Unity5.X 完全自学手册
---
本章涵盖
* 在游戏中使用角色动画

<!--more-->

思考并回答以下问题：
1.什么是IK？
2.IK如何设置？
3.IK的回调函数是什么？
4.使用什么组件，组件的函数有哪些？

# 在游戏中使用角色动画

## Mecanim系统逆向运动学功能

<span style="color:red;">IK与FK对应，正向运动学就是根骨骼带动节点骨骼运动。而反向运动学就是反过来，由子节点带动父节点运动。</span>

对于Humanoid的动画，使用的方法很简单，在Animator窗口中，对于要使用IK的动画状态勾选Foot IK选项，在Base Layer中勾选IK Pass选项。然后在代码中实现OnAnimatorIK函数来控制IK。
```cs
using UnityEngine;
using System.Collections;
using System;

public class IKCtrl : MonoBehaviour
{
    protected Animator animator;
    public bool isActive = false;
    public Transform rightHandObj = null;
    private bool isFirst = true;

    void Start()
    {
        Debug.Log("Start is being called");
        animator = GetComponent<Animator>();
    }

    void OnAnimatorIK(int layerIndex)
    {
        Debug.Log("OnAnimatorIK is being called"); 
        if (animator)
        {    
            if (isActive)
            {
                if (rightHandObj != null)
                {
                    Debug.Log("Set Avatar's position and rotation");
                    animator.SetIKPosition(AvatarIKGoal.RightHand, rightHandObj.position);
                    animator.SetIKRotation(AvatarIKGoal.RightHand, rightHandObj.rotation); 
                } else 
                {
                    
                }
            }
        }
    }
}
```

在3ds Max中使用IK的情景还是非常多的。比如一个小章鱼，在其每只脚绑上IK，然后就可以通过脚步移动控制整条腿的运动。如果不用IK的话，要使角色运动起来很麻烦，而且不自然。FBX的格式里面同样存储有IK信息，只是Unity过滤了相关的数据。大家在导出动画之前做这样的操作，动画即可正常显示。

在3dsMax中按【Ctrl】+【A】键选中所有骨骼，在右侧的选项卡中执行Motion->Trajectories命令，如果已经选择好骨骼，“Collapse”按钮就可以正常单击，单击这个按钮，然后正常地导出动画。这样Unity中的动画表现就跟3ds Max中的一致了。

值得注意的是，使用Collapse功能会修改动画的帧（使帧间隔变得一样），这样我们很多动作播放的时候就会被改变，比如攻击动作会变得很慢，没有力度。美工应该在Collapse后再次修改动画（或者是在制作动画之前使用Collapse）以保证动画的正确性。

## Animator组件

<span style="color:red;">Animator组件是关联角色及其行为的纽带。每一个含有Avatar的角色动画模型都需要有一个Animator组件。</span>Animator组件引用了一个Animator Controller用于为角色设置行为，包括StateMachines状态机、Blend Trees混合树以及通过脚本控制的Events事件。其具体内容如下表所示。


> Animator组件中文名与功能详解

| 英文  | 中文  | 功能详解  |
| :------------ | :------------ | :------------ |
| Controller  | 控制器  | 关联到该角色的Animator控制器  |
| Avatar  | 骨架结构的映射  | Mecanim动画系统的简化人形骨架结构到该角色的骨架结构的映射  |
| Apply Root Motion  | 应用RootMotion选项  | 是使用动画本身还是使用脚本来控制角色的位置  |
| Animate Physics  | 动画的物理选项  | 动面是否与物理属性交互  |
| Culling Mode  | 动画的裁剪模式  | 决定动画是否裁剪以及裁剪的模式<br>□ Always animate：总是启用动面，不进行裁剪<br>□ Based on Renderers：当看不见角色时只有根节点运动，身体的其他部分保持静止  |

## Animator Controller

Animator Controller被用于显示和控制角色的行为。详细地说，是通过在Project视图中执行Create->Animator Controller命令来创建的，同时会在Assets文件夹内生成一个后缀名为.controller的文件。当设置好运动状态机后，就可以在Hierarchy视图中将该Animator Controller拖入到含有Avatar的角色模型的Animator组件上。


Animator Controller窗口包括内容如下图所示。

值得注意的是，Animator Controller窗口总是显示最近被选中的后缀名为.controller资源的状态机，与当前所载入的场景无关。

## Animator动画状态机

一个角色拥有多个可以在游戏中不同状态下调用的不同动作是一件很普遍的事。例如，一个角色可以在等待时呼吸或者摇摆，在得到命令时行走或者从一个平台掉落时惊慌地伸手。当这些动画进行回放时，使用脚本控制它们可能是一个潜在的复杂工作。Mecanim动画系统借用了计算机工程师熟知的一个概念\-\-状态机，来简单的控制和序列化角色动画。如下图所示，为Animator动画状态机原理图。

值得游戏开发者注意的一个最基本的观点是：一个角色应该在任何给定的时刻执行某些特定的动作，这些动作是否可用，是基于游戏进程的，但是典型的动作包括等待、移动、跑动、跳跃等。这些动作被称为状态。在场景中当角色正在行走、等待或者做其他什么的时候都会处于某一个状态。

一般来说，角色在进入下一个状态时会被限制，而不是可以从任意一个状态跳转至另一个任意状态。比如，一个“跑动跳跃”动作只可以在角色正在跑动时执行而不是当角色正在站立的时候执行。你永远不应该从等待动作中直接跳转到跑动跳跃动作，让角色正确跳转状态的选项被称为状态转移。而将上面这些（状态的集合、状态转移的集合和一些用于记录正确状态的变量）整合起来的东西就是一个状态机。

状态机对于动画的重要性在于它们可以很简单地通过相对较少的编码完成设计和更新，每个状态都有一个当前状态机在那个状态下将要播放的动作集合。这将允许动画师和设计师不使用代码而定义可能的角色的动画和动作序列。

Mecanim的动画状态机提供了一种可以预览某个独立角色的所有相关动画剪辑集合的方式，并且允许你能够在游戏中通过不同的事件触发不同的动作。

如下图所示，动画状态机可以通过动画状态机窗口进行设置。

状态机包括状态、状态转移和事件，并且在大的状态机中可以设置一个小的子状态机。具体包含内容如下图所示。