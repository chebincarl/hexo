---
layout: title
title: 性能分析工具-Profiler-1
date: 2019-04-10 09:22:12
categories: Unity
tags: Unity5.X游戏开发指南
---
Profiler是Unity中一个辅助优化游戏性能的工具，它在游戏运行时实时详细报告游戏各个部分每帧所耗费的时间。例如，图像渲染部分、动画系统或者脚本各耗费多少时间。

<!--more-->

在运行场景或者使用设备运行游戏的时候打开Profiler，它会记录并以时间轴为基础显示数据，让你可以知道哪些帧或者功能部分消耗较多的时间。

Profiler会检测代码，这会对性能造成定影响。一般来说，重点关注时间消耗最多的那些帧里消耗最多的部分。

# Profiler界面

点击导航菜单栏->Window->Profiler打开Profiler界面，再打开并运行场景，Profiler窗口如下图所示。本章将图中黑色部分（即以时间为横轴的性能数据）统称为波形图。

功能被分为如下5个部分检测：中央处理器使用率（CPU Usage）、渲染（Rendering）、内存（Memory ）、音频（Audio）、物理（Physics），具体功能分析如下表所示。

* 中央处理器使用率部分主要分析游戏运行时每帧游戏各个部分所消耗的CPU时间。波形图分为7个子项并以不同颜色显示。
* 渲染部分的波形图不再是表示每帧消耗的时间，而是表示渲染的性能指标。波形图分为4个子项并以不同颜色显示。
* 内存部分主要分析游戏运行时每帧游戏各部分所占用的内存大小。波形图分为7个子项并以不同颜色显示。
* 音频部分主要分析游戏运行时每帧的音频性能指标。波形图分为4个子项并以不同颜色显示。
* 物理部分主要分析游戏运行时每帧的物理性能指标。波形图分为两个子项并以不同颜色显示。

所有栏中的波形图都是在最右侧绘制最新的时间占用并向左移动。当前选中的CPU Usage栏，我们可以在这里看到有16ms（60FPS）、10ms（100FPS）、5ms（200FPS）3条横向基准线，它们代表的意思是当波形刚好位于基准线上的数值。5ms代表的是这一帧的CPUi计算占用了5ms的时间，1s等于1000ms，1000除以5等于200，也就是括号内的200FPS即帧的刷新率是200帧/秒。

Overview中显示的是脚本各占用了多少时间。

请将鼠标光标移动到CPU波形图的任意一点并单击鼠标左键，此时出现一条竖线，旁边伴以数据显示该帧的性能情况，参见下图。

# 连接设备

我们已经知道了性能分析的方法，游戏运行的性能分析还是要在具体设备上进行才有实际参考意义。下面就以安卓设备运行游戏并通过Profiler检测为例。

首先打开Build Settings窗口，勾选Development Build，再勾选Autoconnected Profiler，然后点击Build，将打出的apk文件安装至设备，运行设备，并确保设备和电脑处于同一网络环境下（连的是一个Wi-Fi）。

在手机上查看Wi-Fi得到IP地址，然后在Profiler窗口中点击Active Profile按钮，在下拉菜单中点击Enter Player IP按钮，并输入IP地址，点击Connect连接，如下图所示。

连接成功后就正常显示数据了。

# CPU优化

下面我们就来具体介绍下优化经验，并首先看看CPU优化。CPU优化主要关注那些卡的帧，也就是CPU耗时特别长的帧。原因各种各样，这里主要介绍一下各种情况，以便在开发过程中避免。

## 控制台日志与预定义标签

首先新建一个场景和一个空的游戏对象，然后新建一个脚本，命名为CodeExample.cs，并添加至游戏对象上。
```cs
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class CodeExample : MonoBehaviour
{
	public List<string> playerNames;

	void Update()
	{
		for (int i = 0; i < playerNames.Count; i++)
		{
			Debug.Log("player:" + playerNames[i]);
		}
	}
}
```
输出控制台日志，这可以很方便地调试。但是实际情况呢？我们在Inspector视图中将PlayerNames的Size设置为100，并填入各种名字，当然留空也可以，如下图所示。接着，打开Profiler视图并运行场景，数据如图所示。

LogStringToConsole就是对应Debug.Log控制台日志的CPU情况。可以看到，该项每帧的CPU占用时间达到了517ms，也就是仅FPS 1.9!正常的游戏起码要达到FPS 20，本例是远远低于该数值的，原因如下。

Unity中控制台日志是非常占用CPU的，无论是Debug.Log()还是print()，而且生成的应用安装包在运行时依然会输出日志。故在正式发布的时候，请一定不能包含控制台日志。

Update()每帧都会运行，能不放在Update()中运行的逻辑尽量不要放在Update()中。

## 预定义标签

预定义标签Script Define Symbols，编译器会在编译的时候根据预定义标签去生成对应的二进制编码。

点击导航菜单栏>Edit>Project Settings>Player，在Inspector窗口中打开的Player Settings界面的Script Define Symbols栏，在其中输入Test并回车，如下图所示。

然后修改代码。
```cs
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class CodeExample : MonoBehaviour
{
	public List<string> playerNames;
	#if Test
	void Update()
	{
		for (int i = 0; i < playerNames.Count; i++)
		{
			Debug.Log("player:" + playerNames[i]);
		}
	}
	#endif
}
```
其中, #if Test和#endif的意思是，当有预定义标签的时候才会运行标签中的代码，而如果在Player中将这个预定义标签去掉，那么标签中的代码在编辑器里为灰色，也不会被编译执行。我们可以将测试代码用预定义标签包裹，在测试的时候保留Player Settings里的预定义标签，在发包的时候去掉，所以合理的代码如代码清单14-3所示:

以下是一些注意事项。
* 尽量少用Cameobject.Find()搜索函数，因为这些方法效率比较低。相反，你应直接定义公共变量，如果目标是场景中的非动态创建对象，直接在Inspector视图中指定；如果是动态创建的对象，创建的时候就直接保存。

* 尽量少用SendMessage()函数，因为这些方法效率比较低。相反，你应直接调用方法，或者使用C#的委托代理delegate或System.Action。

* 尽最少用粒子系统，或者使粒子系统的粒子数献尽可能少，或者用帧序列动画代替。因为粒子系统中粒子运动的计算对CPU是不小的负担。

* 尽量将资源读取或者释放放在展示Loading画面的时候去做，或者将大的资源拆成小的模块分批加载，外放CPU对硬盘的读写运算,降低峰值。

* 尽量降低场景里的模型面数和点数,因为CPU需要对其进行运算再传递给GPU,