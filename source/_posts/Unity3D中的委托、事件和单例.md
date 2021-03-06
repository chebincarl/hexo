---
layout: title
title: Unity3D中的委托、事件和单例
date: 2019-06-19 17:27:47
tags:
---
思考并回答以下问题：
1.
2.
3.

<!--more-->

来源：[Unity3D中的委托、事件和单例](https://mp.weixin.qq.com/s/upe7XhLNWPuyE7IsoQsTCQ "Unity3D中的委托、事件和单例")

在此，将演示多人协同工作时怎样去创建委托、事件和单例。这篇教程是针对Unity3D来写的，但也同样适用于所有采用C#或.NET的应用。

在判断一些事件或者动作是否发生的时候总是会去写很多布尔类型的声明。我通过协程和一些其他函数来监听这些事件以及返回值。如果你也发现自己是这样做的，那么尽快停止吧！


单例

如果你不知道什么是单例，你或许真的不知道。单例是不能实例化或者复制的脚本。它是，额...单一的。

我建议在游戏中当某样东西不需要被多次复制的时候使用单例。比如存储系统。典型的例子是玩家只需要一个存储系统，所以我们并不想调用它的多个实例，我们只需要一个。当我们调用它时，我们想要确认当前它到底存不存在。

有很多创建单例的方法，但下面这个方法使用率最高因为它很简单...


这里我写了一个Clicker的类并把它添加到我的摄像机上。这个类控制了我用光线投射向3D场景中发送的所有点击事件。


在其他脚本中使用DoSomething函数的时候只需要这样简单的调用...


这样就减少了一系列静态方法和变量的调用，而我们只是添加了一个实例而已！


委托和事件？

委托可以被理解为一个引用指向一个对象或者方法。当它被调用时，就会通知所有引用了该委托的方法。

所以，先说重要的...

定义委托和方法



名称为OnClickEvent的委托传递一个GameObject类型的参数用来让我们定义它是来自什么游戏对象。然后我们定义一个OnClick事件，当委托被调用时调用它。


现在，在同样的脚本里，我们需要调用委托并传递GameObject。我通过光线投射完成了调用...


正如你看到的那样，如果射线接触到场景中的物体并且我们点击了鼠标，那么就成功的调用了事件并传递了该物体。

最后，我们必须做的是在其他脚本中注册该委托并实现相关调用。为此，我创建了一个叫GoldPile的类。


在Awake()函数中，我们定义了监听事件，并指定了本地方法OnClick。OnClick函数并不要求和委托函数同名，但同名也是可以的。

注意：在此之前我们为Clicker类添加了一个单例。这样就允许我们这样使用：Clicker.Instance

正如你所看到的那样，我们也在点击的时候创建了OnClick()函数并传递了GameObject。

注意：你必须这样使用if(g == gameObject),否则，它将会在场景中隐藏其他实例的相关方法...这就是为什么我们为该引用传递GameObject的原因！

现在，如果你需要的话可以在你游戏中的任何脚本里添加这个方法。但别忘了在Awake()函数中定义方法和委托。
