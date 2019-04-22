---
layout: title
title: Unity脚本程序开发（2）
date: 2019-04-22 13:50:10
categories: Unity
tags: Unity游戏开发技术详解与典型案例
---
本章涵盖：
* 预制体（prefab）资源的应用
* 常用输入对象
* 与销毁相关的方法

<!--more-->

# 预制体（prefab）资源的应用

在一个项目的开发过程中经常会应用到预制件（prefab）资源。在场景的开发中会同时创建多个完全相同的游戏对象，如果一一创建会耗费大量的时间，并且也会耗费游戏资源，在管理上也会有一定的难度。这时就需要实例化预制件（prefab）来实现。

<!--more-->

## 预制件（prefab）资源的创建

在Assets菜单Create命令的选项中选择创建perfab，就会在资源项目列表中创建一个预制件（perfab）资源。此时的预制件（perfab）只是一个空壳，还需添加一些具体的游戏对象，具体操作步骤如下。

(1)通过菜单创建一个prefab。执行“Assets->Create->Prefab”命令，即可在项目资源列表中创建一个prefab，然后将其名字改为“BallPrefab”。

(2)在场景中创建一个球体。执行Create->3D Object->Sphere命令，即可在游戏组成列表中创建一个Sphere，然后将其改名为“Ball”。

(3)向项目中导入一个球面纹理图片资源。执行“Assets->Import New Asset...”命令，会立刻弹出一个Import New Assets对话框，在对话框中选中需要的球面纹理图片，单击“Import”按钮完成导入。

(4)为创建的Ball添加刚体属性和球体碰撞者属性，即先选中Ball，再执行“Component->Physics->Rigidbody”命令，即可为Ball添加刚体属性，再执行“Component->Physics->Sphere Collider”命令，即可为Ball添加球体碰撞属性。

(5)将导入的球面纹理图片资源添加到创建的Ball上面。选中纹理图片然后将其拖动到创建的Ball上面即可，添加的各个属性都将在属性查看器中显示出来。

(6)为刚创建的BallPrefab添加真实的游戏对象。这个操作跟5中一样，只要选中刚刚创建的Ball对象，然后将其拖到Assets面板中已经创建好的BallPrefab上即可，此时这个空的BallPrefab就具有了与Ball对象完全相同的属性。

### 通过prefab资源进而实例化对象

在实际的开发过程中，若要创建大量的重复化的资源，就需要使用到prefab资源。通过脚本编写程序实例化这些游戏对象，这样可以省去创建过程的时间，以及为各个游戏对象添加相同属性的烦琐操作，并且还会节省大量的游戏资源，提高项目的运行效率。

下面将通过一个实例化篮球的小案例来讲解prefab实例化的具体操作过程。

1.以上面的BallPrefab为例，在此有关prefab的具体创建过程就不再进行讲解。

2.编写脚本，实例化篮球，执行“Assets->Create->C# Script”命令，在项目中创建一个C#脚本，在此将脚本的名字改为“BallPrefabScript”，然后双击脚本进入脚本编辑器。对脚本进行编辑。
```cs
using UnityEngine;
using System.Collections;

public class BallPrefabScript : MonoBehaviour 
{
	public int i = 5; // 声明整型变量i
	public int j = 0; // 声明整型变量j
	public Rigidbody BallPrefab; // 声明刚体BallPrefab
	public float x = 0.0f; // 初始化x,y,z的坐标
	public float y = 4.0f;
	public float z = 0.0f;

	public float k = 2.0f;
	public int n = 4;  // 声明实例化球的行数
	int count = 0;  // 声明一个计数器
	public Rigidbody[] BP; // 声明刚体数姐

	void Start() // 声明start方法 
	{
		Bp = new Rigidbody[10];  // 初始化刚体组数
		count = 0; // 计数器置0
		for (i = 0; i <= n; i++) // 对变量i进行循环
		{
			for (j = 0; j < i; j++) // 对变量j进行循环
			{
				// 在自定义坐标位置实例化10球
				BP[count++] = (Rigidbody)Instantiate(BallPrefab, new Vector3 (x-2.0f*k*i + 4.0f*j*k,  2.0f, z-2.0f*1.75f*k*i), BallPrefab.rotation);
			}
		} 	
	}  
}
```
首先声明变量，主要声明了整型变量i、j，刚体BallPrefab, x、y、z的坐标，刚体的行数及计数器，并且对相关的参数进行了赋值。在开发环境下的属性查看器中可以为各个参数指定资源或者取值。

对Start方法进行了重写，初始化了刚体数组，然后先对i进行循环，再在i的循环中将j进行循环，最后在自定义坐标位置通过实例化刚体数组创建了10个球体，并且对10个球的位置按照一定的规律进行了排列。

3.将编写完的脚本挂载到摄像机上，然后对摄像机脚本属性中的各个变量参数进行设置。设置完毕后运行，在游戏场景中会显示出实例化的效果。本案例中创建了10个BallPefab。

# 常用的输入对象

在游戏的开发过程中，时常需要获取用户的输入情况，类似于手机或平板中的触控行为，计算机端的键盘鼠标操作行为等，在其他的开发平台中，要获取这些操控参数往往需要通过开发人员编写不少代码来实现，而Unity 3D引擎在设计时就封装好了这些常用的方法与参数。

针对用户的输入，引肇专门为开发人员提供了两个输入对象--Touch与Input。开发人员通过Touch与Input输入对象中的方法以及参数可以非常方便地获取用户输入的各种参数，包括触控的位置、相位、手指按下位移，以及用户鼠标键盘的输入等。

<!--more-->

## Touch输入对象

Touch输入对象中提供了非常详细的参数以及方法，通过使用该对象可以获取如Android、IOS等移动平台中的详细的触摸操控信息，读者可以将分析Touch的代码写在对应的脚本中，然后挂载到对应的游戏对象上，就可以轻松地获取到Touch的信息了。 Touch事件变量如下表所示。

| 变量名  | 含义  |
| :------------ | :------------ |
|  fingerID |  手指的索引 |
|  position | 手指的位置  |
|  deltaPosition | 距离上次改变的距离增量  |
|  deltaTime |  自上次改变的时间增量 |
|  tapCount | 点击次数  |
|  phase |  触摸相位 |

Touch触摸输入对象的各个参数在开发的过程中一般都是相互配合使用的，只有这样才能符合开发的需要。接下来将给出一个解析玩家手势操控的案例，希望读者可以通过该案例与学习的内容进行相互印证以加深理解。

1.首先搭建所需的游戏场景。新建一个场景并将其命名为“TouchTest”，在其中创建一个平行光Direction Light，然后执行“GameObject->3D Object->Sphere”命令，创建一个小球并赋予其纹理图。调整小球的大小与位置以及摄像机的位置,如下图所示。

2.简单地搭建好场景后接下来进行脚本的开发。新建一个名为“TouchTest.cs”的C#脚本，将其挂载到主摄像机MainCamera上。脚本代码如下：
```cs
using UnityEngine;
using System.Collections;

public class TouchTest : MonoBehaviour 
{
	public GameObject ball;  // Sphere游戏对象的引用
	private float lastDis = 0;  // 上一次两个手指的距离
	private float cameraDis = -20;  // 摄像机距离球的距离
	public float ScaleDump = 0.1f;  // 缩放阻尼

	void Update() {
		if (Input.touchCount == 1)  // 触控
		{ 
			Touch t = Input.GetTouch(0);  // 获取触控
			if (t.phase == TouchPhase.Moved) // 手指移动中
			{  
				ball.transform.Rotate(Vector3.right, Input.GetAxis("Mouse Y"), Space.World); // 竖直旋转
				ball.transform.Rotate(Vector3.up, -1 * Input.GetAxis("Mouse X"), Space.World);// 水平旋转
			}
		} else if (Input.touchCount > 1)
		{
			Touch t1 = Input.GetTouch(0);  // 获取触控
			Touch t2 = Input.GetTouch(1);  // 获取触控
			if (t2.phase == TouchPhase.Began) // 开始触摸
			{ 
				lastDis = Vector2.Distance(t1.position, t2.position); // 初始化lastDis
			} elseif (t1.phase == TouchPhase.Moved && t2.phase == TouchPhase.Moved) // 两个手指都在移动
			{ 
				float dis = Vector2.Distance(t1.position, t2.position); // 计算手指位置
				if (Mathf.Abs(dis -lastDis) > 1)  // 若是手指距离>1
					cameraDis += (dis - lastDis) * ScaleDump;  // 设置摄像机到物体的距离
					cameraDis = Mathf.Clamp(cameraDis, -40, -5); // 限制摄像机到物体的距离
					lastDis = dis;  // 备份本次触摸结果
			}
		}
	}

	void LateUpdate()
	{
		this.transform.position = new Vector3(0, 0, cameraDis); // 调整摄像机的位置
	}
	
	void OnGUI()
	{
		// 打印信息与退出按钮
		string s = string.Format("Input.touchCount={0}\ncameraDIS=\n{1}",Input.touchCount, cameraDis);  // 打印字符
		GUI.TextArea(new Rect (0, 0, Screen.width / 10, Screen.height), s); // 用Text控件显示字符串
		if (GUI.Button(new Rect (Screen.width *9/10, 0, Screen.width / 10, Screen.height / 10), "quit")) // 退出按钮
		{
			Debug.Log("quit");  // 打印点击信息
			Application.Quit();  // 退出程序
		} 
	}  
}
```

口第1-7行的主要功能为命名空间的引用以及声明变量。在声明变量的部分中声明了场景中的Sphere游戏对象的引用Ball，方便下面对其进行旋转等变换；同时还声明了一些数据全局变量，用途将在下面的讲解中进行介绍。
口第8-14行代码是在Update方法中对单指操控行为进行解析。当发生触控并且用户的手指在移动状态，就可以通过Input.GetAxis("Mouse XY")获取用户的手指位移，然后将其转换为旋转角对ball进行旋转。运行时就可以看到，当用户滑动手指时，场景中的小球根据滑动方向进行旋转。

口第15-27行是解析用户多点操控的行为，当手指数目大于1时,计算两个手指问的距离，并与上一次计算出的距离进行比较，若是距离变大就将摄像机向近推产生放大的效果，反之摄像机向后推就可以得到缩小的效果。这里还在第25行对摄像机的位置进行了限制，使其不能无限放大或者缩小。最后备份下这一帧中手指间的距离用于下一帧中和新的距离进行比较。

口第28-30行为对LateUpdate方法的重写，这个方法在Update方法回调完后进行回调。在这部分中根据上一步算出来的cameraDis对摄像机进行前推或者后拉，产生放大或者缩小的效果。

口第31-39行代码与触控的检测没有什么关系，主要是使用Text控件对触控的信息进行打印，使其在真机上也可以看到，方便学习与调试，最后还设置了一个退出按钮，单击该按钮后程序结束运行。

在第12,13行中使用了Input.GetAxis("Mouse XY")来获取用户手指的位移而没有使用Touch. deltaPosition，这是因为手机屏幕的不同Touch. deltaPosition的返回值是不同的，所以使用起来比较不方便；而Input.GetAxis("Mouse X/Y")也可以实现相应的效果，并且在iOs平台中也可以使用该方法。具体使用哪个，读者可以根据开发需要自行选择。

(3)将案例导入手机中运行，就可以看到小球根据玩家的手指滑动或两指放大收缩而发生旋转或缩放了。需要注意的是,与Touch有关的项目都需要在真机上进行测试。

## Input输入对象
如果说Touch输入对象可以用于获取用户的触摸操作信息，那么Input对象就可以获取用户其他所有行为的输入，如鼠标、键盘、加速度、陀螺仪、按钮等,所以掌握Input输入对象就可以在外部输入信息和系统之间架立一座桥梁，因此显得其尤为重要。Input对象的主要变量如下表所示。