---
layout: title
title: Unity脚本-2
date: 2019-04-08 16:51:53
categories: Unity
tags: Unity5.X游戏开发指南
---
思考并回答以下问题：
1.如何通过脚本移动，旋转和缩放对象？
2.四元数是什么？如何使用？
3.如何自定义输入？
4.如何使用重力感应？

<!--more-->

本章涵盖
* 移动、旋转和缩放游戏对象
* 工具类
* 输入控制

# 移动、旋转和缩放游戏对象

在3D世界中，任何一个游戏对象在创建的时候都会附带Transform组件，并且该组件是无法删除的。

Transform面板中一共包含3个属性：Position（位置），Rotation（旋转）和Scale（缩放）。既可在场景中使用移动工具来拖动和旋转模型，也可以直接在Inspector窗口下的Transform面板中手动填写对象的位置、旋转和缩放的数值。

## 游戏对象的位置

在3D世界中，任何一个模型的三维坐标都保存在Vector3容器中，该容器将记录物体在x轴、y轴和z轴方向的坐标。一旦在程序中修改该游戏对象的坐标，那么Scene视图中游戏对象的位置将发生改变。

## 移动游戏对象

游戏对象在原有位置的基础上继续移动，在代码中可以使用transform.Translate()函数实现，此函数的唯一参数为位移的数值：
transform.Translate(Vector3 offset);
该函数相当于transform.position = transform.position + offset。

## 缩放游戏对象

在Unity中，可以通过代码动态缩放游戏中的游戏对象。
```cs 
transform.localScale = new Vector3(x,y,z);
```
其中Vector3的x为x轴向的缩放，y为y轴向的缩放，z为z轴向的缩放。也可以通过下面的代码格式快速整体缩放：
```cs
transform.localScale *= 1.2f; // 对象整体放大1.2倍
```

## 旋转游戏对象

游戏对象的旋转方式分为两种：第一种是自转；第二种是围绕旋转，也就是围绕一个点或者一个游戏对象来旋转。

* transform.Rotate()：该函数用于设置游戏对象自转。
* transform.RotateAround()：该函数用于设置游戏对象围绕某一个点旋转。
* Time.deltatime：上一帧所消耗的时间，这里用作模型旋转的速度系数。
* Vector3.right：x轴正方向。
* Vector3.up：y轴正方向。
* vector3.forward：z轴正方向。

## 实例

在本实例中，我们会通过点击按钮对游戏对象进行对应的移动缩放旋转操作，如代码所示。
```cs
using UnityEngine;
using System.Collections;

public class testDemo : MonoBehaviour 
{
    public GameObject cube;
    public GameObject cylinder;
    
    void OnGUI()
    {
        if(GUILayout.Button("向左移动物体"))
        {
            cube.transform.Translate(new Vector3(-0.5f, 0f, 0f));
        }
        if(GUILayout.Button("向右移动物体"))
        {
            cube.transform.position = cube.transform.position + new Vector3(0.5f, 0f, 0f);
        }
        if (GUILayout.Button("放大物体"))
        {
            cube.transform.localScale *= 1.2f;
        }
        
        if(GUILayout.Button("缩小物体"))
        {
            cube.transform.localScale = 0.8f;
        }
        if (GUILayout.Button("旋转物体"))
        {
            cube.transform.Rotate(new Vector3(0, 10, 0));
        }
        if(GUILayout.Button("围绕圆柱体旋转物体")
        {
            cube.transform.RotateAround(cylinder.transform.position, Vector3.up, 10);
        }
    }
}
```

# 工具类

## 时间类

Unity提供了Time类，这个类主要用来得到与时间相关的信息。
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
* Time.time：从游戏开始时计时，截止到目前共运行的游戏时间，受Time.timeScale影响，游戏暂停时该时间不增加。

* Time.timeScale：时间流逝的速度。当该值设置为1时表示和现实中的时间流逝一致；当该值设置为0.5时，表示真实时间逝去1秒时，游戏时间仅逝去0.5秒；当设置该值为2表示真实时间逝去1秒时，游戏时间逝去2秒。

* Time.deltaTime：上一帧所消耗的时间。

* Time.fixedTime：每一次执行FixedUpdate()函数的时间间隔。可通过导航菜单栏Edit->Project Settings->Time菜单项设置。

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
* <span style="color:red;">Mathf.Lerp(from, to, a)：插入值，返回值=from+to(1-a)。</span>
* Mathf.Min(a, b, c)：返回两个或n个数的最小值，参数为整数或者浮点数。
* Mathf.Max(a, b, c)：返回两个或n个数的最大值，参数为整数或者浮点数。 
* Mathf.Pow(a, b)：a的b次方。
* Mathf.Deg2Rad：常量浮点数，0.0174532924f，用于将角度转换成弧度。
* Mathf.Rad2Deg：常量浮点数，57.29578f，用于将弧度转换成角度。
* Mathf.Pi：常量浮点数，表示圆周率。
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

# 输入控制

玩家点击鼠标左键开火、按住键盘w键前进等都属于输入控制，如何监测输入是非常重要的内容。

## 计算机输入

计算机设备的输入指的是仅对应键盘和鼠标的输入检测，一般单项检测分为3类：按下、拉住、抬起。

```cs
using UnityEngine;
using System.Collections;

public class DetectInput : MonoBehaviour
{
    void Update()
    {   
        // 按下键盘A键
        if (Input.GetKeyDown(KeyCode.A)){}
        // 按住键盘A键
        if (Input.GetKey(KeyCode.A)){}
        // 抬起键盘A键
        if (Input.GetKeyUp(KeyCode.A)){}
        // 按下键盘左Shift键
        if (Input.GetKey(KeyCode.LeftShift)){}
        // 按住键盘左Shift键
        if (Input.GetKey(KeyCode.LeftShift)){}
        // 抬起键盘左Shift键
        if (Input.GetKeyUp(KeyCode.LeftShift)){}     
        // 按下鼠标左键               
        if (Input.GetMouseButtonDown(0)){}
        // 按住鼠标左键
        if (Input.GetMouseButton(0)){}
        // 抬起鼠标左键
        if (Input.GetMouseButtonUp(0)){}
        // 按下鼠标右键 
        if (Input.GetMouseButtonDown(1)){}
        // 按住鼠标右键
        if (Input.GetMouseButton(1)){}
        // 抬起鼠标右键
        if (Input.GetMouseButtonUp(1)){}        
    }
}
```
## 自定义输入

但是键盘鼠标输入检测十分局限，一般仅用于计算机等设备，下面就来介绍另一种方法。

自定义输入可以设置输入类型名称、输入设备类型、输入键位等参数，从而方便地解决了计算机与家用机的输入兼容。

点击导航菜单栏->Edit->Project Settings->Input，打开输入设置界面。如下图所示，Unity提供了默认的输入设置，包括了“Horizontal”横向移动、“Vertical”纵向移动、“Fire1”开火按钮等输入。展开“Fire1”输入项，各项参数如图所示，参数说明见下表。

| 参数  | 说明  |
| :------------ | :------------ |
| Name  | 名字  |
| Descriptive Name  | 控制设置中显示的正值名称  |
| Descriptive Negative Name  | 控制设置中显示的负值名称  |
| Negative Button  | 该按钮用于负方向移动轴  |
| Positive Button  | 该按钮用于正方向移动轴  |
| Alt Negative Button  | 备选按钮用于负方向移动轴  |
| Alt Positive Button  | 备选按钮用于正方向移动轴  |
| Gravity  | 当没有相关按钮按下时,回归0的速度。单位/秒  |
| Dead  | 模拟的死区大小。设定范围内所有模拟设备的值为0  |
| Sensitivity  | 灵敏度,单位/秒。仅用于数码设备  |
| Snap  | 如果启用,当按下相反方向的按钮,该轴值将重设为0  |
| Invert  | 如果启用,负按钮将提供正值,反之亦然  |
| Type  | 控制轴的输人设备类型  |
| Axis  | 连接设备的轴将控制这个轴  |
| Joy Num  | 连接操纵杆将控制这个轴  |

可以发现很多键如Horizontal都出现了重复，这是因为所有键位键盘鼠标是单独的一套键而手柄则是另一套。例如“Fire1”开火键1分为键盘鼠标版和手柄版，无论哪种都能触发脚本中对应的逻辑。

** 1.按钮 **

这里以开火按钮为例，如图3-16所示我们可以检测开火按钮的按下、按住、抬起3个状态下面就来介绍如何实现按钮输入。

□ 键盘鼠标。键盘鼠标实现按钮非常简单。

第一步：设置类型，首先将Type设置为Key or Mouse Button。
第二步：填写名字，这里Name填的是Fire1。
第三步：设置键位。因为是单一按钮，所以仅仅需要填写正向部分。Positive Button填left ctrl，Alt Positive Button填mouse 0，也就是键盘左侧的Control键或者鼠标左键都对应开火键。

□ 手柄。步骤和键盘鼠标完全一样，只是键位名称不一样而已，这里Positive Button填写的是joystick button 0。

□ 脚本。当在Input Manager界面中设置好键位后，我们可以通过脚本监测输入。

```cs 
using UnityEngine;
using System.Collections;

public class DetectInput : MonoBehaviour
{
    void Update()
    {   
        // 按下Fire1键
        if (Input.GetButtonDown("Fire1")){}
        // 按住Fire1键
        if (Input.GetButton("Fire1")){}
        // 松开Fire1键
        if (Input.GetButtonUp("Fire1")){}     
    }
}
```
** 2.方向轴 **

方向轴常用于控制玩家角色的左右移动或上下移动。它的设置界面和按钮是完全一样的，但用法却不一样，方向轴有两个按钮分别对应正负两个方向。以Horizontal方向轴为例，如下图所示，按下键盘右箭头是正值，按下键盘左简头是负值。输出的范围是[-1,1]的浮点数，我们可以用它来控制角色的左右移动。下面就来介绍如何实现。

□ 键盘鼠标。前几步和按钮一样，只是需要额外设置Gravity、Dead、Sensitivity、Snap等参数。

Gravity填写3表示当松开对应按钮后，输出值会以3/秒的速度迅速归零。Dead填写0.001表示当输出在[-0.001,0.001]之间时会被忽略不计，强制输出0。Sensitivity填写3表示当按钮下对应按钮后，输出值会以3秒的速度变化，当按下的是正向按钮时会迅速到达1，当按下的是负向按钮时会迅速到达-1。

□ 手柄。如下图所示，这里对应的不再是手柄的按钮而是手柄轴。

第一步：设置类型。首先将Type设置为Joystick Axis。
第二步：填写名字。这里Name填的是Horizontal,
第三步：设置手柄。Joy Num栏如果填Get Motion from all Joysticks表示对应所有手柄，也可以填lystick I等手柄序号。
第四步：设置轴，Axis栏这里填的是Xaxis，也就是手柄十字键的横向。

□ 脚本。脚本中获得轴的代码非常简单。

```cs
using UnityEngine;
using System.Collections;

public class AxisDemo : MonoBehaviour
{
    void Update()
    {
        // 得到Horizontal轴的值
        float axisH = Input.GetAxis("Horizontal");
    }
}
```

## 移动设备输入

移动设备也就是手机、平板等通过手指点击屏幕操作的设备。Unity有专门的接口检测与屏幕互动的各个手指的位置状态等信息。

与屏幕接触的手指的信息对应名为Touch类的对象，可以通过Input.Touches变量得到所有Touch。Touch常用的参数见下表。

| 参数  | 说明  |
| :------------ | :------------ |
| FingerId  | 手指的编号，整型  |
| Phase  | 手指的阶段，枚举类型。分为这几个阶段：<br>Began开始接触屏幕<br>Moved移动<br>Stationary静止<br>Ended手指离开屏幕<br>Canceled系统关闭触控  |
| Position  | 手指触碰屏幕的位置，Vetor2类型。坐标以屏幕左下角为原点1像素对应一个单位，例如iPhone 4s的分辨率是960*640。所以如果应用是横屏的话，那么左下角的Position是(0,0)，右上角的Position是(960,640)  |

** 1.实测 **

如下代码所示，我们得到手指的信息并输出到屏幕。
```cs
using UnityEngine;
using System.Collections;

public class MobileTouchDemo : MonoBehaviour
{
    void OnGUI()
    {   
        // 遍历所有的Touch
        foreach (Touch touch in Input.touches) 
        {
            // 输出Touch信息
            GUILayout.Label(string.Format("手指：{0} 状态：{1} 位置：{2}", touch.fingerId, touch.phase.ToString(), touch.position));
        }
    }
}
```
** 2.重力感应 **

Acceleration，即加速度传感或者重力感应。有很多著名的手机游戏的主要操作是基于重力感应的；在手机赛车游戏中重力感应也可用于控制赛车的转向。重力感应的原理是当手握移动设备晃动时，移动设备内的加速度计会计算设备在X、 Y、Z这3个方向上的线性加速度变化。以设备为基准，X轴正向为设备向右的方向，Y轴正向为设备向上的方向，Z轴正向为设备向使用者的方向。在Unity中可以通过Input.acceleration得到重力感应的值，类型为vector3。每个轴向的值域是[-1, 1]。

可以通过以下代码将重力感应信息输出至屏幕。可以在设备上运行此场景，并观察设备以不同朝向转动时Input.acceleration值的变化。
```cs
using UnityEngine;
using System.Collections;

public class AccelerationDemo : MonoBehaviour
{
    void OnGUI()
    {   
        GUILayout.Label("X:" + Input.acceleration.x);
        GUILayout.Label("Y:" + Input.acceleration.y);
        GUILayout.Label("Z:" + Input.acceleration.z);
    }
}
```
** 3.其他 **

Input还有一些接口可以在设备运行游戏时得到关于设备输入的信息。
* 通过Input.deviceOrientation可以得到当前游戏运行的朝向。
* 通过Input.touchSupported可以得到当前游戏是否支持手指触控操作。
* 通过Input.multiTouchEnabled可以设置游戏是否支持多点触控。

# 习题

1.使用OnGUI()函数做一个调查问卷，要求填写名字、性别以及年龄并在提交后将信息输出至控制台。
2.在平台上创建一个球体，实现通过键盘的WASD键对其进行前后左右移动操作。