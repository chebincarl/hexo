---
layout: title
title: Unity脚本程序开发-综合案例
date: 2019-03-17 20:00:24
tags: Unity
---
这是一个控制飞机飞行的案例。该案例的初衷就是运用基本的方法完成对飞机运动状态的控制，以及摄像机对目标物体的跟随。

<!--more-->

本案例基本上包含了Start方法和Update方法的使用、向量的应用、标签功能的应用、对Android设备各个键的监听、整体场景的搭建，以及灯光的控制，正是通过这些应用的相互配合，才能使项目得以顺利运行。

效果如下：gif

## 案例策划及准备工作
实现对飞机的飞行状态的控制（主要包含了飞机的前进、转向等功能），在项目开发前，首先要对项目开发所需要使用的资源进行收集和归类。需要的主要资源如下所示。
图片资源列表
| 图片名  | 大小/KB  | 像素  | 用途  |
| :------------ | :------------ |
| plane_texture.jpg  | 1024  | 2048X 2048  | 飞机机身贴图  |
| plane_glass.jpg  | 7.59  | 64x64  | 飞机挡风玻璃贴图  |

模型资源列表
| 图片名 | 大小/KB  | 格式  | 用途  |
| :------------ | :------------ |
| airplane.FBX  | 146  | FBX  | 2D物理属性  |

## 创建项目及场景搭建
(1)导入飞机模型包。执行"Assets-Import Package-Custom Package"命令,会立刻弹出Import package..对话框,如图3-3所示。在该对话框内浏览所要导入的包并选中,单击打开按钮,就会弹出图3-4所示的显示了资源包内的文件列表的对话框。勾选所需要导入的资源文件,单击Import按钮,开始导入资源。地形资源包的导入方法与飞机模型包的导入步骤相同。

(2)完成资源的导入后,在Project视窗中选中飞机模型 aipanl.FBX,如图3-5所示。将其拖曳到场景中,如图3-6所示。.

(3)在Project视窗中分别选中纹理贴图plane texture.jpg和plane glass.jpg,如图3-7所示。将选中的纹理贴图拖曳到飞机模型身上即可为模型添加纹理,最终效果如图3-8所示。

(4)接下来为飞机模型添加刚体组件。首先选中模型,在Inspector视窗中,单击Add Component按钮,如图3-9所示,选择"Physcis-Rigibody",添加完成后,设置刚体组件的参数,如图3-10所示,取消勾选USe Gravity选项,使物体不受重力影响.

(5)在Project视窗中选中地形文件SeneTerrain-Town-Scene, preab, 图3-1所示,将其拖曳到场景中,并调整位置,如图3-12所示。


## 飞机控制脚本实现
(1)首先创建一个名字为"AirControl.cs"的脚本,将此脚本挂载到飞机模型上.

```cs
using UnityEngine;
using System.Collections;
public class AirControl : MonoBehaviour 
{
    private transform m_transform; //保存Transform实例
    public float speed = 600f; //飞机的飞行速度
    private float rotationz = 0.0f; //绕2轴的旋转量
    public float rotatespeed Axisz = 45f; // 烧2轴的旋转速度
    public float rotatespeed AxisY = 20f; //绕丫轴的旋转速度
    private Vector2 touchPosition; //触摸点坐标
    private float screenWeight; //屏幕宽度

    void Update()
    {
    
    }
}
```

口第1-2行为导入系统包。
	口第4行为Mono维承自MonoBehaviour类,只有继承自MonoBehaviour的类才可以作为Unity脚本组件被使用。
口第5行声明了一个Transform实例,用于后面存放Transform组件的调用。
口第6行为飞机飞行的速度,该变量为public型,可在Unity中直接更改它的值.
口第7行为飞机绕2轴的旋转量,用于保存飞机实时的姿态。
口第8-9行为飞机绕2轴的旋转速度和绕y轴的旋转速度。
口第10行为手指触摸到移动设备屏幕上的坐标。
口第11行为移动设备屏幕的宽度。
	(2)接下来重写Start函数。Start函数是在第一次调用Update或FixedUpdate函数之前被调用,这里一般用于编写在脚本开启后先执行且执行一次的代码,例如一些初始化操作。具体代码如下。

-2mgu
void start0) (
m transtorm  this.transforma
this.gameabject. GetComponent<Rigidbody> () .useGravity- false; //关闭重力影响screenleight  Screen.widthz  //获取屏幕定度
	口第2行用于保存this.transform的调用,避免了后面在Update方法中多次调用游戏对象身上的Transform组件。这样写可以减少外部代码的调用,提高运行效率。
	口第3行用于关闭游戏对象的受重力的影响。使用GetComponent方法获取了游戏对象身上的Rigidbody组件后,为其useGiravity变量赋值"false".
口第4行用于获取设备屏幕的宽度。
	(3)接下来让飞机开始动起来。首先是飞机可以向前飞行以及飞机的螺旋桨可以转动。此时需要重写Update方法,具体代码如下.
1
45
void Update () 0
m transform.Translate (new Vector3(0, o, speed * Time.deltaTime) );//向前移动Gameobject.Find ("propeller").transform.Rotate (new Vector3(0, 100of.
Time, deltaTime, 0));
//寻找到名称为"propeller"的对象并使其绕丫轴旋转
	口第2行用于使飞机朝2轴方向移动, Vector3是一个三维向量,用于在Unity传递3D位置和方向, 3个参数分别代表x、y,z轴上的分量.Time.deltaTime为完成最后一帧的时间,在每帧中发生增减变化的数值就需要与"Time.deltalime"相乘。口第3行用于查找到propeller对象,并使其旋转。通过Find方法,查找飞机身上的名称为"propeller"的游戏对象。 propeller对象的位置如图3-13所示。通过Rotate方法使其发生旋转,这里仍是使用到了Vector3,因为螺旋桨是围绕y轴旋转的,所以第二个参数有值,为旋转的速度。
	(4)接下来是控制飞机左右转向的代码。控制飞机转向是通过触摸屏幕来实现的,当玩家触摸屏幕的左半侧时飞机左转,触换屏幕的右半侧时飞机右转,当没有触摸事件发生时恢复平衡状态。具体代码如下。
void Update ( 1
mtransform.Translate (new Vector3(o, o, speed . Time.deltaTime) ) ://向前移动17寻找到名称为"propeller"的对象并使其绕y轴旋转
GameCbject. Find ("propoller") .transform. Rotate (new Vector3(0, 1000ETime.deltaTime, D))1/获取飞机对象绕2轴的旋转量
rotationz - this.transform.eulerAngles.z;
if (Input.touchCount >0)  //当離摸的数量大于。
	for (int i-0; i < Input.touchCount; i++) 1
	Touch touch -Input.touches[1]:  //实例化当前触搜点
	1/手指在屏幕上没有移动或发生滑动时触发的事件
	if (touch.phase TouchPhase. stationary Il touch.phaseTouchPhase.Moved) C
	//获取当前触摸点坐标
	touchPosition touch.position;
	1/触摸点在屏幕左半树
	if (touchPosition.x < screenWeight / 2) (
	11飞机左转
	m transform. Rotate tnesVector3(0,-Time.deltaTime * 30, 0), Space.World) ;
//触摸点在屏幕右半侧
else if (touchPosition.x >= screenweight / 2) (
1/飞机右转
m transform. Rotate (new Vector3 10, Time.deltaTime 30, O), Space. Morild) ;
i (Input. touchcount-o) (BackToBlance ():
//手指离开屏幕时触发的事件
else if (touch.phase s TouchPhase. Ended) (
	BackToBlance ();  "//调用恢复平衡状态方法
	// 表  手指触搜屏幕时  复平衡状态方法

口 第5-6行为获取飞机对象绕2轴的旋转量。
口 第7-8行用于判断触摸事件是否发生了。
口第9行用于实例化当前触摸点,用于后面判断触摸事件的类型和获取触摸点坐标。
	口第11行用于判断触摸事件。通过phase的值来判断,当其等于TouchPhase.Stationary时代表手指在屏幕上但没有移动:当其等于TouchPhase. Moved时,手指在屏幕上并发生了滑动。口第14-18行用于判断飞机左转。如果触摸点在屏幕左半侧,就以一定的速度向左发生旋转。因为是左转,所以旋转的速度取负值。
口第19-22行用于判断飞机右转,与左转类似,故不再赞述。
	口第24-27行用于判断当手指离开屏暮时触发的事件,所调用的BackToBlance方法在后面介绍。
口第28-30行为没有手指触摸屏幕时。触发的事件,同样是调用BackToBlance方法。(5)为了使飞机的转向更加真实,需要在飞机转向时让飞机的机身倾斜(例如,飞机左转时,机身应该同时向左稍微倾斜),所以需要改进上面的代码。将上面的第15-18行代码改为如
if (touchPosition.x < screenWeight / 2) 1  //触摸点在屏幕左半侧
if ((rotationz <- 45 11 rotationz 2 315)) (
//飞机向左倾斜
m transform.Rotate (rew Vector3 (0, o, (Time.deltaTine rotateSpeed Axis2), Space.Sel);
//飞机左转
m-transform. Rotate (new Vector3(0, -Time.deltaTime . 30, 0), Space.World);
]第2行为判断飞机发生倾斜的阈值。
0第3-4行用于使飞机发生向左倾斜,同样是使用了Rotate方法,此时是通过使飞机绕自:轴旋转,来实现倾斜的效果。因为是要绕自身坐标旋转,所以要使用Space.Self参数。......主意: 向右倾斜功能的代码与向左倾斜类似。读者可参照随书光盘中的源代码
6)下面介绍上面所调用的BackToBlance方法。该方法用于在没有发生转向时,使飞机恢复
