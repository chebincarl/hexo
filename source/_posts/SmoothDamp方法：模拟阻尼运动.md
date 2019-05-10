---
layout: title
title: SmoothDamp方法：模拟阻尼运动
date: 2019-03-12 07:51:25
categories: Unity
tags: Unity API解析
---
思考并回答以下问题：
1.SmoothDamp是干嘛用的？
2.如何使用？


<!--more-->

** 基本语法 **

(1) public static float SmoothDamp(float current, float target, ref float currentVelocity, float smoothTime);
(2) public static float SmoothDamp(float current, float target, ref float currentVelocity, float smoothTime, float maxSpeed);
(3) public static float SmoothDamp(float current, float target, ref float currentVelocity, float smoothTime, float maxSpeed, float deltaTime);

对以上重载方法中涉及的参数进行说明：
参数current为起始值；参数target为目标值；参数currentVelocity为当前帧速度，ref类型；参数smoothTime为预计平滑时间；参数maxSpeed为当前帧最大速度值，默认值为Mathf.Infinity（无穷）；参数deltaTime为平滑时间，值越大返回值也相对越大，一般用Time.deltaTime计算。

** 功能说明 **

此方法的功能是模拟平滑阻尼运动，并返回模拟插值。smoothTime:float，预计平滑时间，物体越靠近目标，加速度的绝对值越小。实际到达目标的时间往往要比预计时间大很多，建议smoothTime的取值范围为(0.0f, 1.0f)，若想控制物体到达目标的时间可以通过控制maxSpeed来达到目的。maxSpeed:float = Mathf.Infinity，每帧返回值的最大值，默认值为Mathf.Infinity。

** 提示 **

可以观察实例演示中maxSpeed取默认值和取较小值时当前速度随时间变化的示意图。

实例演示 下面通过实例演示方法SmoothDamp的使用。

```cs
using UnityEngine;
using System.Collections;

public class SmoothDamp_ts: MonoBehaviour
{
    float targets = 110.0f; // 目标值
    float cv1 = 0.0f, cv2 = 0.0f; // 输出值
    float maxSpeeds = 50.0f; // 每帧最大值
    float f1 = 10.0f, f2 = 10.0f; // 起始值

    void FixedUpdate()
    {
        // maxspeed取默认值
        f1 = Mathf.SmoothDamp(f1, targets, ref cv1, 0.5f);
        Debug.Log("f1:" + f1):
        Debug.Log("cv1:" + cv1);

        // maxSpeed取有限值50.0f
        f2 = Mathf.SmoothDamp(f2, targets, ref cv2, 0.5f, maxSpeeds);
        Debug.Log("f2:" + f2);
        Debug.Log("cv2:"+ cv2);
    }
}
```

在这段代码的FixedUpdate方法中调用了两次SmoothDamp方法，第一次调用时取maxSpeed值为默认值，即无穷大，第二次调用时取maxSpeed值为有限值，然后分别打印出它们每帧的输出值。下图分别是对输出值cv1和lcv2随时间变化的可视化显示，可以发现，起始时输出速度提升很快，结束时速度下降却很平缓，在移动距离相同的情况下，有最大速度限制的花费时间也更多。