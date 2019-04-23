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

思考并回答以下问题：
1.鼠标有几个输入来源？如何得到？
2.键盘这么多的按键怎么得到？
3.如何得到不同的操作状态，比如按下，按住，抬起？


# 预制体（prefab）资源的应用

在一个项目的开发过程中经常会应用到预制件（prefab）资源。在场景的开发中会同时创建多个完全相同的游戏对象，如果一一创建会耗费大量的时间，并且也会耗费游戏资源，在管理上也会有一定的难度。这时就需要实例化预制件（prefab）来实现。

## 预制件（prefab）资源的创建

在Assets菜单Create命令的选项中选择创建perfab，就会在资源项目列表中创建一个预制件（perfab）资源。此时的预制件（perfab）只是一个空壳，还需添加一些具体的游戏对象，具体操作步骤如下。

(1)通过菜单创建一个prefab。执行“Assets->Create->Prefab”命令，即可在项目资源列表中创建一个prefab，然后将其名字改为“BallPrefab”。

(2)在场景中创建一个球体并改名为“Ball”。

(3)向项目中导入一个球面纹理图片资源。执行“Assets->Import New Asset...”命令，会立刻弹出一个Import New Assets对话框，在对话框中选中需要的球面纹理图片，单击“Import”按钮完成导入。

(4)为创建的Ball添加刚体属性和球体碰撞者属性，即先选中Ball，再执行“Component->Physics->Rigidbody”命令，即可为Ball添加刚体属性，再执行“Component->Physics->Sphere Collider”命令，即可为Ball添加球体碰撞属性。

(5)将导入的球面纹理图片资源添加到创建的Ball上面。选中纹理图片然后将其拖动到创建的Ball上面即可，添加的各个属性都将在属性查看器中显示出来。

(6)为刚创建的BallPrefab添加真实的游戏对象。这个操作跟5中一样，只要选中刚刚创建的Ball对象，然后将其拖到Assets面板中已经创建好的BallPrefab上即可，此时这个空的BallPrefab就具有了与Ball对象完全相同的属性。

### 通过prefab资源进而实例化对象

在实际的开发过程中，若要创建大量的重复化的资源，就需要使用到prefab资源。通过脚本编写程序实例化这些游戏对象，这样可以省去创建过程的时间，以及为各个游戏对象添加相同属性的烦琐操作，并且还会节省大量的游戏资源，提高项目的运行效率。

下面将通过一个实例化篮球的小案例来讲解prefab实例化的具体操作过程。

1.以上面的BallPrefab为例，在此有关prefab的具体创建过程就不再进行讲解。

2.编写脚本，实例化篮球。在项目中创建一个“BallPrefabScript”脚本。
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
	public Rigidbody[] BP; // 声明刚体数组

	void Start()
	{
		Bp = new Rigidbody[10];  // 初始化刚体数组
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

针对用户的输入，引擎专门为开发人员提供了两个输入对象\-\-Touch与Input。开发人员通过Touch与Input输入对象中的方法以及参数可以非常方便地获取用户输入的各种参数，包括触控的位置、相位、手指按下位移，以及用户鼠标键盘的输入等。

## Touch输入对象

Touch输入对象中提供了非常详细的参数以及方法，通过使用该对象可以获取如Android、IOS等移动平台中的详细的触摸操控信息，读者可以将分析Touch的代码写在对应的脚本中，然后挂载到对应的游戏对象上，就可以轻松地获取到Touch的信息了。Touch事件变量如下表所示。

| 变量名  | 含义  |
| :------------ | :------------ |
|  fingerID |  手指的索引 |
|  position | 手指的位置  |
|  deltaPosition | 距离上次改变的距离增量  |
|  deltaTime |  自上次改变的时间增量 |
|  tapCount | 点击次数  |
|  phase |  触摸相位 |

Touch触摸输入对象的各个参数在开发的过程中一般都是相互配合使用的，只有这样才能符合开发的需要。

1.首先新建一个场景并将其命名为“TouchTest”，创建一个小球并赋予其纹理图。调整小球的大小与位置以及摄像机的位置，如下图所示。

2.新建一个名为“TouchTest.cs”的C#脚本，将其挂载到主摄像机MainCamera上。脚本代码如下：
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
				if (Mathf.Abs(dis -lastDis) > 1)
				{  
					// 若是手指距离>1
					cameraDis += (dis - lastDis) * ScaleDump;  // 设置摄像机到物体的距离
					cameraDis = Mathf.Clamp(cameraDis, -40, -5); // 限制摄像机到物体的距离
					lastDis = dis;  // 备份本次触摸结果
				}
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

首先引用命名空间以及声明变量。在声明变量的部分中声明了场景中的Sphere游戏对象的引用Ball，方便下面对其进行旋转等变换；同时还声明了一些数据全局变量。

在Update方法中对单指操控行为进行解析。当发生触控并且用户的手指在移动状态，就可以通过Input.GetAxis("Mouse X/Y")获取用户的手指位移，然后将其转换为旋转角对ball进行旋转。运行时就可以看到，当用户滑动手指时，场景中的小球根据滑动方向进行旋转。

接着解析用户多点操控的行为，当手指数目大于1时，计算两个手指间的距离，并与上一次计算出的距离进行比较，若是距离变大就将摄像机向近推产生放大的效果，反之摄像机向后推就可以得到缩小的效果。这里还对摄像机的位置进行了限制，使其不能无限放大或者缩小。最后备份下这一帧中手指间的距离用于下一帧中和新的距离进行比较。

对LateUpdate方法进行重写，这个方法在Update方法回调完后进行回调。在这部分中根据上一步算出来的cameraDis对摄像机进行前推或者后拉，产生放大或者缩小的效果。

OnGUI函数与触控的检测没有什么关系，主要是使用Text控件对触控的信息进行打印，使其在真机上也可以看到，方便学习与调试，最后还设置了一个退出按钮，单击该按钮后程序结束运行。

使用了Input.GetAxis("Mouse X/Y")来获取用户手指的位移而没有使用Touch.deltaPosition，这是因为手机屏幕的不同Touch.deltaPosition的返回值是不同的，所以使用起来比较不方便；而Input.GetAxis("Mouse X/Y")也可以实现相应的效果，并且在iOS平台中也可以使用该方法。具体使用哪个，读者可以根据开发需要自行选择。

(3)将案例导入手机中运行，就可以看到小球根据玩家的手指滑动或两指放大收缩而发生旋转或缩放了。需要注意的是，与Touch有关的项目都需要在真机上进行测试。

## Input输入对象

如果说Touch输入对象可以用于获取用户的触摸操作信息，那么Input对象就可以获取用户其他所有行为的输入，如鼠标、键盘、加速度、陀螺仪、按钮等，所以掌握Input输入对象就可以在外部输入信息和系统之间架立一座桥梁，因此显得其尤为重要。Input对象的主要变量如下表所示。


> Input对象的主要变量

| 变量名  | 含义  |
| :------------ | :------------ |
| mousePosition  | 当前鼠标的像素坐标  |
| anyKeyDown  | 用户点击任何键或鼠标按钮，第一帧返回true    |
| acceleration  | 加速度传感器的值  |
| anyKey  | 当前是否有按键按住，若有返回true  |
| inputString  | 返回键盘输入的字符串  |
| touches  | 返回当前所有触摸（Touch）列表  |

1.mousePosition变量
 
变量mousePosition是一个三维的坐标，用于获取当前鼠标的像素坐标。像素坐标是以屏幕左下角为（0,0），屏幕右上角坐标为（Screen.width, Screen.height）计算的。
```cs
void Update()
{
    if(Input.GetButtonDown("Fire1")) // 鼠标左键点下  
    {
        Debug.Log(Input.mousePosition);  // 打印鼠标位置
    }
}
```

2.anyKey变量与anyKeyDown变量

变量anyKey用于显示当前是否有任何按键按下，若是有，就始终返回True。将下面的代码添加到脚本中，将脚本挂载到摄像机上，当按下任何按键时就会不停地显示打印信息。
```cs
void Update()
{
    if(Input.anyKey) // 有按钮按下
    {
        // 打印信息
        Debug.Log("A key or mouse click has been detected");
    }
}
```
变量anyKeyDown和变量anyKey有些许差别，前者只有按下按钮后的第一帧返回True。将上面的代码片段稍做修改后运行场景即可发现，只要有按钮按下，就会打印一次信息；若是按钮持续处于按下状态，也仅仅打印第一次。
```cs
void Update()
{
    if (Input.anyKeyDown) // 按钮按下
    {
        // 打印信息
        Debug.Log("A key or mouse click has been detected");
    }  
}
```

3.inputString变量

变量inputString返回键盘在这一帧中输入的字符串。注意，在返回的字符串中只包含ASCII码中的字符，若是本次没有输入字符串就会返回一个空串，如下面的代码片段所示。
```cs
void Update()
{
    if (Input.inputString != "") // 若当前输入字符串不为空
    {
        Debug.Log(Input.inputString); // 打印输入字符串
    }
}
```

4.acceleration变量

变量acceleration可以获取设备在当前三维空间中的线性加速度，常见于3D游戏中的重力感应操控模式。当用户倾斜设备时，若设备上有加速度传感器，就会回传一个代表设备倾斜加速度的三维向量，使用Input.acceleration变量就可以获取该参数。具体实现代码如下。

```cs
using UnityEngine;
using System.Collections;

public class example : MonoBehaviour
{
    public float speed = 10.0f; // 移动速度

    void Update()
    {
        Vector3 dir = Vector3.zero; // 新建一个三维向量
        dir.x = -Input.acceleration.y; // 获取重力感应y轴参数
        dir.z = Input.acceleration.x; // 获取重力感应x轴参数

        if (dir.sgrMagnitude >1) // 若是获取的三维向量不是标准向量
            dir.Normalize(); // 规格化向量
        
        dir *= Time.deltaTime; // 将方向向量转换为速度
        transform.Translate(dir * speed); // 平移物体
    }
}
```
将该脚本挂载在一个游戏对象上，导入支持重力传感器的设备中运行后就可以看到，游戏对象会根据用户倾斜手机的方向进行相应方向的移动。当然，这段代码需要设备支持重力感应，否则就会一直返回Vector3.Zero。

5.touches变量

上一节中介绍过Touch输入对象，与之相对应地，通过Input.touches变量可以获取到当前在屏幕上的所有触控的引用（Touch[]类型），开发人员就可以根据索引轻易地获取各个触控点的信息，所以该变量也会经常被使用到。
```cs
void Update()
{
    int fingerCount = 0; // 手指数目计数器
    foreach (Touch touch in Input.touches) // 遍历每个触控点
    {
        if (touch.phase != TouchPhase. Ended && touch.phase != TouchPhase.Canceled) // 当前触控点不是结束状态且当前触控不是取消状态
        {
            fingerCount++; // 触摸计数器自加
        }

        if (fingerCount > 0) // 有触摸
        {
            // 打印信息
            print("User has " + fingerCount + "finger(s) touching the screen");
        }
    }
}
```

该代码片段的作用为，每当触控发生时就通过Input.touches获取到每个触控的引用，然后遍历触控列表。若触控的相位不是结束状态或取消状态，就将手指数目计数器fingerCount加1，最后打印出当前在屏幕上的有效触控手指的数目。

Input输入对象中不仅包括了丰富的变量，而且还提供了大量的实用方法。下面将对Input输入对象中封装好的常用方法进行进行详细的介绍。

> Input输入对象中的主要方法

| 方法名  | 含义  |
| :------------ | :------------ |
| GetAxis  | 返回被表示的虚拟轴的值  |
| GetButton  | 若虚拟按钮被按下返同true  |
| GetButtonUp  | 虚拟按钮抬起的一帧返回true  |
| GetKeyDown  | 按下指定按钮的一帧返回true   |
| GetMouseButton  | 指定的鼠标按键按下时返回true  |
| GetMouseButtontUp  | 指定鼠标按键抬起的一帧返回true  |
| GetAxisRaw  | 返回没有经过平滑处理的虚拟轴的值  |
| GetButtonDown  | 虚拟按钮被按下的一帧返回true  |
| GetKey  | 按下指定按钮时返回true  |
| GetKeyUp  | 抬起指定按钮的一帧返回true  |
| GetMouseButtonDown  | 指定的鼠标按键按下的一帧返回true  |
| GetTouch  | 根据索引返回当前触控（Touch类型）  |


6.GetAxis方法和GetAxisRaw方法

GetAxis方法和GetAxisRaw方法都是获取虚拟轴的值的方法。在游戏的开发过程中，经常会在屏幕中添加一些2D的虚拟轴，可以通过触控或者鼠标事件改变虚拟轴的值来控制场景中的游戏对象。具体使用方法如下面的代码片段所示。
```cs
using UnityEngine;
using System.Collections;

public class InputTest : MonoBehaviour 
{
    private float speed = 0.1f; // 移动速度 
     
    void Update()
    {
        float moveX = Input.GetAxis("Horizontal");  // 获取水平轴的值
        float moveY = Input.GetAxis("Vertical");  // 获取垂直轴的值
        this.transform.Translate(new Vector3(moveX, moveY, 0) *speed); //移动物体
    }
}
```
将上面的脚本挂载到场景中的游戏对象上，使用键盘的方向键就可以控制游戏对象的移动了。这时若是把moveX的值打印出来就会发现，当按下方向键时，其值是从-1~ 1之间平滑过渡的。接下来运行下面的代码片段。

```cs
void Update()
{
    float moveX = Input.GetAxisRaw("Horizontal"); // 获取水平轴的值
    Debug.Log(moveX);
}
```
这段代码使用了GetAxisRaw方法，运行后按下方向键就会观察到moveX的值只会有-1、0、1三种变化，没有中间的过渡值。这是因为GetAxisRaw方法没有使用平滑滤波器。在需要自定义差值的情况下可以使用GetAxisRaw方法。

7.GetButton方法、GetButtonDown方法与GetButtonUp方法

这3种方法用于监听虚拟按钮的按下状态，包括按钮按下时、按钮按下中、按钮抬起时3个状态。开发人员需要在Update方法中回调这些方法来判断按钮的状态。其中的区别可以参看下面的功能代码片段。
```cs
using UnityEngine;
using System.Collections;

public class InputTest : MonoBehaviour
{
    void Update()
    {
        if (Input.GetButton("Fire1")) // 使用GetButton监听“Fire1”按键
        {
            Debug.Log("Fire GetButton");
        }

        if(Input.GetButtonDown("Fire1")) // 使用GetButtonDown监听“Fire1”按键
        {
            Debug.Log("Fire GetButtonDown");
        }
        
        if (Input.GetButtonUp("Fire1")) // 使用GetButtonUp监听“Fire1”按键
        {
            Debug.Log("Fire GetButtonUp");
        }
    }
}
```
将上述脚本挂载到主摄像机上，按住鼠标左键不放，就会发现第6行的打印始终在被回调，而第8行的打印代码仅在按下时回调了两次。当松开鼠标左键时，才会发现第12行代码被回调。通过这个简单的脚本，读者应该已经可以区分这3种方法的区别了。

8.GetKey方法、GetKeyDown方法与GetKeyUp方法

这3种方法用于监听键盘上的按键的状态，开发人员需要在Update方法中调用这些方法，并传入想要监听的键名或键码。每个按钮的状态也分为按下、抬起、按住3种，使用者可以根据需要进行选用，使用方法如下面的代码片段所示。
```cs
using UnityEngine;
using System.Collections;

public class InputTest : MonoBehaviour
{
    void Update()
    {
        if (Input.Getkey("up")) // 使用GetKey监听“↑”按键
        {
            Debug.Log("up arrow Getkey");
        }
        
        if (Input.GetKeyDown(KeyCode.UpArrow))  // 使用GetKeyDown监听“↑”按键
        {
            Debug.Log("up arrow GetKeyDown");
        }
        
        if (Input.GetKeyUp(KeyCode.UpArrow))  // 使用GetKeyUp监听“↑”按键
        {
            Debug.Log("up arrow GetKeyUp");
        }
    }    
}
```
第5行和第8行分别使用了键名和键码两种方式来监听方向上键，其效果是相同的。将上面的脚本挂载到摄像机上运行场景，点击“方向上键”就会看到相应的打印信息，可见GetKey是按住时始终回调的，GetKeyDown和GetKeyUp分别只有按下和抬起时的一帧调用。
    
9.GetMouseButton方法、GetMouseButtonDown方法和GetMouseButtonUp方法

当开发计算机端的游戏时，肯定需要监听鼠标的操控。Input输入对象中包含了GeMouseButton、GetMouseButtonbDown和GetMouseButtonUp三种方法，用它们来监听鼠标按键。在使用时只要在Update方法中传入鼠标按键的索引，就可以对鼠标进行监听了。与前面介绍的方法类似，这3种方法也分别监听了鼠标按键的3个状态。使用方法见如下代码。

```cs
void Update()
{
    if (Input.GetMouseButton(0)) // GetMouseButton监听鼠标左键
    {
        Debug.Log("left mouseButton GetMouseButton"); // 打印信息
    } 
    
    if (Input.GetMouseButtonDown(0)) // GetMouseButtonDown监听鼠标左键
    {
        Debug.Log("left mouseButton GetMouseButtonDown"); // 打印信息
    }  
    
    if (Input.GetMouseButtonUp(0)) // GetMouseButtonUp监听鼠标左键
    {
        Debug.Log("left mouseButton GetMouseButtonUp"); // 打印信息
    }
}
```

这三种方法的参数是一个int类型的索引。常用的鼠标按键索引为0，1，2，分别监听了鼠标的左键、右键、中键。需要使用的时候传入相应的索引就可以监听对应的按键了。

10.GetTouch方法

前面已经介绍过了Touch输入对象，使用其参数时需要获取一个Touch类型的变量。Input.GetTouch方法就是用于获取Touch输入对象的引用，在使用时应传入一个索引值代表要获取的触控索引，使用方式见如下代码片段。
```cs
void Update()
{
    if (Input.touchCount != 0) // 当前发生触控
    {
        Vector3 touchPOS = Input.GetTouch(0).position; // 记录下触控点的位置
    }
}
```
上面的代码片段获取了发生触控时的首个触控点，并将其位置记录了下来。注意这个方法只有在支持触摸的移动设备上运行才会生效。

# 与销毁相关的方法

在游戏的开发过程中，经常会遇到对象、组件、资源等在使用完毕后就失去了作用的情况，放任其不管的话轻则影响项目运行效率，重则可能影响到项目的正常运行。所以必须有一类方法来管理（删除）这些没有用的资源。在本节中将要介绍Unity中的各类删除方法。

Unity中有很多Destroy方法，不同功能的Destroy方法用于销毁不同类型的资源，下面将主要讲解常用的各个类型的Destroy方法的区别以及使用，不同功能的Destroy方法如下表所示。

| 函数  | 功能  |
| :------------ | :------------ |
| Object.Destroy  | 删除游戏对象、组件或资源  |
| NetWork.Destroy  | 销毁网络对象  |
| MonoBehavior.OnDestroy  | 脚本被销毁时间  |


## Object.Destroy方法

Object.Destroy方法可以将对象立即销毁，也可以设置时间后销毁。如果删除的对象是一个组件，则该组件会被移除。下面将通过一个具体的代码片段来说明Object.Destroy的使用方式。
```cs
void Start()
{
    Destroy(ball.GetComponent<Rigidbody>());
    Destroy(ball, 5);
}
```
在这个代码片段中，ball是场景中的一个挂有Rigidbody组件的游戏对象。在Start方法中，首先删除掉ball上挂载的刚体组件，然后在5s后删除ball游戏对象。

## NetWork.Destroy方法

NetWork.Destroy方法可以销毁网络对象，该方法包含了如下两种重载方式。
```cs
static function Destroy(viewID : NetworkViewID) : void
static function Destroy(gameObject : GameObject) : void
```
当使用第一种方法重载方式时，需要给出网络对象的viewID，然后系统会删除所有和该viewID相关的物体。需要注意的是，本地的和远端的物体都会被销毁。使用方法见如下代码片段。
```cs
using UnityEngine;
using System.Collections;

public class example : MonoBehaviour
{
    //通过网络销毁拥有该脚本的物体，必须具备Networkview离性
    public float timer= 0;
    void Awake ()
    {
        timer = Time.time;
    }
    
    // 计时器
    // 记录下开始时间

    void Update()
    {
        if (Time.time-timer>2) // 2s后
        {
            Network.Destroy(GetComponent<NetworkView>().viewID) ; // 删除具有NetworkView的物体
        }  
    }
}
```
NetWork.Destroy方法还可以使用第二种重载方式来销毁网络上的游戏对象，下面将用一段代码片段来说明。
```cs
using UnityEngine;
using System.Collections;

public class example : MonoBehaviour
{
    public float timer = 0; // 声明计时器

    void Awake()
    {
        timer = Time.time; // 记录下开始时间
        
        void Update()
        {
            if (Time.time - timer >2) // 2s后
            {
                Network.Destroy(gameObject); // 删除gameObject
            }
        }
    }
}
```
这段代码的主要功能就是自脚本唤醒后2s删除游戏对象“gameObject”，其中Time.time代表游戏开始后的真实时间。

## MonoBehaviour.OnDestroy方法

MonoBehaviour.OnDestroy方法是MonoBehaviour中的销毁回调方法。类似于脚本中常见的Update()、Start()方法，该方法也由系统自动回调，这个方法的回调条件是当该脚本被移除时系统回调。具体实现方法如下面代码片段所示。
```cs
using UnityEngine;
using System.Collections;

public class DestroyTest : MonoBehaviour
{
    void Start()
    {
        Destroy(this.GetComponent<DestroyTest>(), 5); // 移除该脚本
    }
    
    void OnDestroy()
    {
        Debug.Log("this script has been destroy"); //移除该脚本时回调
    }
}
```
将该脚本挂载到摄像机上后运行场景，在这段代码中首先在第5行指定5s后从摄像机上删除这个脚本，所以等到5后删除脚本时就会看到第8行的打印，这是因为OnDestroy()方法在移除该脚本时被自动回调了。
