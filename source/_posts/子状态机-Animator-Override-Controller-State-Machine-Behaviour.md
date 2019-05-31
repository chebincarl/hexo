---
layout: title
title: 子状态机/Animator Override Controller/State Machine Behaviour
date: 2019-05-31 17:36:03
categories: Unity
tags: 大话Unity2018
---
今日思考题
用今天新学的内容把你的状态机好好整理整理

<!--more-->

除了使用Layer还有没有更好的组织状态的方式呢？一个Layer里面状态多的时候，会显得很乱

还可以使用Sub-State Machines，也就是子状态机，可以将一些状态放到子状态机里。

## Sub-State Machines 子状态机

很多时候，角色的一个行动由多个动作组成。比起使用一个动画完成这个行为，可能定义几个不同的阶段，并且给每个阶段使用一个单独的State会更合理。比如一个“稳定射击”的动作，包含蹲下、射击、站起的动作。





这样分拆的动画更利于控制，但是带来的负面影响是状态机会变得庞大、复杂、难以管理。这时候，可以使用状态机提供的子状态机的功能，将这些状态整合到一个子状态中。

创建子状态机
在Animator窗口的空白处右键，在弹出菜单中选择Create Sub-State Machine。





子状态机会显示为一个细长的六边形用来和正常的state区分。





编辑子状态机
双击这个子状态机可以打开它，界面会显示这个子状态机内的内容（初始会是一个空的状态机）。窗口的上方会有一个面包屑路径显示当前编辑的是哪个子状态机（子状态机内也可以再创建子状态机），点击对应的面包屑路径可以直接打开对应的位置。





子状态机内的State编辑和之前的State编辑相同。

子状态机的Transition
子状态机只是从视觉上将一些状态折叠到一个子状态机中，所以如果想转换到sub-state machine时，需要选择具体转换到哪一个状态或整个状态机。

创建Transition时需要选择


选择状态时，Transition会直接转换到对应的状态。选择状态机时，相当于将Transition转换到对应状态机的Entry状态。

子状态机中有一个额外的状态名字为（Up）XXX。





这个状态代表上一层的状态机。你可以在子状态机中创建Transition转换到上一层状态机中的某个状态。





选择状态时，Transition会直接转换到对应的状态。选择状态机时，相当于将Transition转换到对应状态机的Entry状态。




Animator Override Controller可以让你在保留Animator Controller中的结构和逻辑的同时，覆盖里面的一些动画，比如你有多个角色的状态机结构逻辑都相同，但是动画不同，这时候就可以用Animator Override Controller。State Machine Behaviour是一种用脚本的形式，可以挂在State上面，会有一些回调用来处理State不同状态。



# Animator Override Controller

Animator Override Controller是Project中的一种资产，可以用来扩展已有的Animator Controller，替换Animator Controller中的动画，但是保留原Animator Controller中的结构、参数和逻辑。

例如：游戏中有很多类型的NPC（哥布林、兽人、精灵等），状态机逻辑相同，但是每种NPC有自己独特的动画。这时候你只需要创建一个基础的Animator Controller，结合使用Animator Override Controller可以创建出很多不同的变体。

创建Animator Override Controller
在Project中的Create菜单中，选择Animator Override Controller。





左边是Override Controller的图标，右边是Animator Controller的图标


Animator Override Controller和Animator Controller的图标很相近，除了左下角一个是加号，一个是播放的标志。

编辑Animator Override Controller
给Controller赋值

首先需要给Animator Override Controller一个基于的Animator Controller。赋值完成后，下面会显示状态机中所有的动画，这时候你可以用新的动画来覆盖原来状态机对应状态的动画。





最后Override Controller可以用于Animator组件的Controller。

注意：Avatar可能需要替换为对应模型的Avatar。

# State Machine Behaviour

State Machine Behaviour是一种特殊的脚本。和通用的Unity脚本（MonoBehaviours）挂到GameObject上面类似，StateMachineBehaviour可以挂到Animator Controller的State上面。可以在StateMachineBehaviour脚本中编写代码，在状态进入、离开、停留在特定的state时执行。你就不需要自己去检测状态的变化。

可能用于的场景举例：

进入、离开状态时播放音效

只在特定的状态中执行一些代码

只在特定的状态中激活特效

创建StateMachineBehaviour
选中一个State，点击Inspector中的Add Behaviour按钮可以选择已有的StateMachineBehaviour或创建一个新的StateMachineBehaviour。





StateMachineBehaviour中的事件
StateMachineBehaviour中有一些预定义的事件方法：
OnStateMachineEnter 转换到一个StateMachine时调用。注意转换到子状态机中的状态时不会调用。
OnStateMachineExit 离开StateMachine时调用。注意转换到子状态机中的状态时不会调用。
OnStateEnter 进入当前State时调用
OnStateExit 离开当前State时调用
OnStateUpdate 处于当前状态时，每次Update都会调用（不包括Enter和Exit的两帧）
OnStateMove 在MonoBehaviour.OnAnimatorMove之后调用
OnStateIK 在MonoBehaviour.OnAnimatorIK之后调用

# 总结
