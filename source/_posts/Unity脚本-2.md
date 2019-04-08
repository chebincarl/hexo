---
layout: title
title: Unity脚本-2
date: 2019-04-08 16:51:53
categories: Unity
tags: Unity5.X游戏开发指南
---
本章涵盖：
* 工具类
* 输入控制

# 工具类

Unity为开发者提供了很多实用的工具类，极大地方便了开发。它们是由系统封装的一些功能与方法，不需要开发者去实现类似功能。

## 时间类

Unity提供了Time类，这个类主要用来得到与时间相关的信息。代码所示的例子使用了Time类，将游戏运行中的重要时间数值显示在屏幕上。
```cs
using UnityEngine;
using System.Collections;

public class TimeDemo : MonoBehavlour 
{
	void OnGUI()
	{
		GUILayout.Label("当前游戏时间" + Time.time);
		GUILayout.Label("游戏时间的缩放" + Time.timeScale);	
		GUILayout.Label("上一帧所消耗的时间" + Time.deltaTime);
		GUILayout.Label("固定增量时间" + Time.fixedTime);
		GUILayout.Label("上一帧所消耗的固定时间" + Time.fixedDeltaTime);
		GUILayout.Label("真实逝去时间" + Time.realtimeSinceStartup);
	}

}
```

* Time.time：从游戏开始时计时，截止到目前共运行的游戏时间，受Time.timeScale影响，游戏暂停时该时间不增加
。

* Time.timeScale：时间流逝的速度。当该值设置为1时表示和现实中的时间流逝一致；当该值设置为0.5时，表示真实时间逝去1秒时，游戏时间仅逝去0.5秒；当设置该值为2表示真实时间逝去1秒时，游戏时间逝去2秒。

* Time.deltaTime：上一帧所消耗的时间。

* Time.fixedTime：每一次执行FixedUpdate()函数的时间间隔。可通过导航菜单栏“Edit->Project Settings-> Time”菜单项设置。

* Time.fixedDeltaTime：固定更新上一帧所消耗的时间。

* Time.realtimeSinceStartup：从游戏开始时计时，截止到目前共运行的真实时间，不受Time.timeScale影响，游戏暂停时该时间仍然增加。

## 随机数

在开发中，有时需要获取程序中的随机数，这可以使用Random类中的Random.Range()函数实现，其中该函数的第一个参数传入的是随机数的最小值，第二个参数传入的是随机数的最大值。两个参数共同决定了生成随机数的值域。
```cs
using UnityEngine;
using System.Collections;

public class RandomDemo : MonoBehaviour
{
	void OnGUI()
	{
		if(GUILayout.Button("生成随机数"))
		{
			// 生成随机数
			int i = Random.Range(0, 10);
			Debug.Log("随机生成的一个0~10之间的整数是:" + i);
			float f = Random.Range(0f, 10f);
			Debug.Log("随机生成的一个0~10之间的浮点数是:" + f);.
		}
	}
}
```

## 数学类

Unity提供了一个数学类Mathf，该类位于UnityEngine命名空间下。以下是Mathf类里常用的函数和属性。

* Mathf.Abs(a)：返回a的绝对值，参数为整数或者浮点数。
* Mathf.Clamp(a, min, max)：将a限制在min和max之间，参数为整数或者浮点数。 
* Mathf.Lerp(from, to, a)：插入值，返回值=from + to(1-a)。
* Mathf.Min(a, b, c)：返回两个或n个数的最小值，参数为整数或者浮点数。
* Mathf.Max(a, b, c)：返回两个或n个数的最大值，参数为整数或者浮点数。 
* Mathf.Pow(a, b)：a的b次方。
* Mathf. Deg2Rad：常量浮点数，0.0174532924f，用于角度转换弧度。
* Mathf.Rad2Deg：常量浮点数，57.29578f，用于弧度转换角度。
* Mathf.Pi：常量浮点数表示圆周率3.141592653...。
* Mathf.Sin(a)：返回弧度a的正弦值。
* Mathf.Cos(a)：返回弧度a的余弦值。
* Mathf.Tan(a)：返回弧度a的正切值。

## 四元数

四元数（Quaternion）是非常重要的工具类之一。<span style="color:red;">在Unity中所有用到模型旋转的，其底层都是由四元数实现的，它可以精确地计算模型旋转的角度。</span>在场景中创建一个立方体，并添加代码脚本。点击运行，立方体会一直旋转。

```cs
using UnityEngine;
using System.Collections;

public class QuaternionDemo : MonoBehaviour 
{
	// 绕y轴自转的速度
	float rotateSpeed = 50f;

	void Update()
	{
		// 绕y轴自转
		transform.rotation = Quaternion.Euler(0f, rotateSpeed * Time.time, 0);
	}
}
```
我们使用Quaternion.Euler(Vector3 vec)函数，传入一个Vector3(x,y,z)，分别代表围绕x、y、z轴旋转的角度，返回该角度对应的四元数，将四元数赋值给立方体的rotation旋转变量完成旋转。