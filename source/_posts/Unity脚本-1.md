---
layout: title
title: Unity脚本-1
date: 2019-02-28 21:49:47
categories: Unity
tags: Unity5.X游戏开发指南
---
Unity的编程工作都是在脚本里编写的，脚本通过添加到游戏对象的方式运行。

<!--more-->

# MonoBehaviour

所有创建的用于添加到游戏对象上的脚本都必须继承自MonoBehaviour（不添加到游戏对象上就无需继承MonoBehavior），继承自MonoBehaviour的脚本从唤醒到销毁有着完整的生命周期。

* Awake()，脚本唤醒函数。当游戏对象被创建的时候，游戏对象绑定的脚本会在该帧（frame）内执行Awake()函数，无论脚本是否处于激活（enable）状态。

* Start()，该函数在脚本被激活的时候执行，位于Awake()函数之后。该函数的执行同样也是在游戏对象被创建的帧里。不同的是，如果脚本处于不激活状态（MonoBehaviour.enable=false），Start()函数是不会被执行的。

* Update()，只要处于激活状态下的脚本，都会在每一帧里调用Update()函数，该函数也是最为常用的一个函数，用来更新逻辑。

* LateUpdate()，该函数是延迟更新函数，处于激活状态下的脚本在每一帧里都会在Update()函数执行后调用该函数，通常用来调整代码执行的顺序。比如玩家的角色需要一个摄像机来跟随，那么通常角色的移动逻辑会写在Update()里。而摄像机跟随在LateUpdate()里，这样可以确保在角色的位置计算完毕后，再根据角色位置确定摄像机的位置和视角。

* FixedUpdate()，该函数用于固定更新。在游戏运行的过程中，每一帧的处理时间是不固定的，当我们需要固定间隔时间执行某些代码时，就会用到FixedUpdate()函数。在导航栏中，点击Edit->Project Settings->Time菜单项，之后在Inspector视图里出现时间管理器，其中Fixed Timestep选项用于设置FixeUpdate()的更新频率，更新频率默认是0.02秒，如下图所示。固定更新常用于移动物体等操作，因为固定更新每一帧调用的时间间隔是一样的，所以移动速度是均匀的。

{% asset_img 1.png %}

* OnGUI()，绘制界面函数。因为Unity使用最新的UGUI界面系统来创建页面，所以OnGUI()一般作为测试功能使用，如创建测试按钮等。

* OnDestroy()，在当前脚本销毁时调用该函数，我们可以在函数里填写删除时需要处理的逻辑。

* OnEnable()，激活函数，当脚本被激活时调用。

* OnDisable()，当脚本被禁用时调用。

所有继承自MonoBehaviour的脚本都有一个名为enabled的bool值开关，enabled对应脚本名称左侧的勾。<span style="color:red;">enabled决定生命周期的函数是否会被调用。当enabled为true时，生命周期各个阶段对应的函数将会被调用，否则不调用。</span>当enabled变为true时脚本执行OnEnable()，当enabled变为false时脚本执行OnDisable()。

<span style="color:red;">enabled只与生命周期的函数有关，与其他函数和所有变量都没有关系。</span>如果脚本中不包含任何生命周期函数，enabled将没有任何意义，此时在Inspector窗口中，脚本名左侧将不再有勾选框，如下图所示。同理，<span style="color:red;">不管是没有任何生命周期函数还是enabled为false，都不影响其他脚本调用此脚本的变量或者非生命周期函数。</span>

{% asset_img 2.png %}

设置脚本自定义图标。选中任意脚本，在Inspector窗口中点击下图所示的左上角图标，弹出设置脚本图标窗口。可以选择任意一种预制样式，或者点击“Other...”按钮选择图片作为图标。这里选择第一个灰色椭圆图标，效果如下图所示。

{% asset_img 3.png %}

{% asset_img 4.png %}

# UnityGUI

由于效率和可视化程度低等原因，一般用UnityGUI作为测试UI。

UnityGUI使用一个特殊的OnGUI()函数，在该函数中加入实现UI的脚本。

它一共有两种类型的接口：GUI.xxx()和GUILayout.xxx()。第一种需要自己手动填写处于屏幕上的位置。第二种Unity会为我们自动排版，我们只需要关心内容即可。本文所有的测试UI只用到以下3个函数。

* GUILayout.Label(string str)：标签，用于显示文本。输入的参数为需要显示的文本。

* GUILayout.Button(string str)：按钮，用于触发事件。输入的参数为在按钮上显示的文字。它返回一个bool值，表示是否按下了按钮。

* GUILayout.TextField(string str)：文本区域，用于输入文本。输入的参数为显示的文本，返回经过用户输入改动后的文本。一般来说，用参数变量接收返回值即可。如
```cs
string str="";
str = GUILayout.TextField(str);
```
新建一个场景，将场景里的对象全部删除。再新建一个空的游戏对象，重命名对象为Manager。然后新建一个脚本GUIDemo.cs。
```cs
using UnityEngine;
using System.Collections;

public class GUIDemo : MonoBehaviour 
{
	string text = "";
	string myName = "";

	void OnGUI()
	{
		// 用标签显示文本
		GUILayout.Label("请输入你的名字：");
		// 用文本区域输入名字
		text = GUILayout.TextField(text);
		if(GUILayout.Button("提交"))
		{
			myName = text;	
		}
		
		// 当myName不为空的时候，说明我们已经提交了名字，则显示名字
		if( !string.IsNullOrEmpty(myName))
		{
			GUILayout.Label("提交成功，名字：" + myName);
		}
	}
}
```
{% asset_img 5.png %}

# 调试

我们可以在脚本里输出调试信息，并在控制台中显示，主要是在脚本中使用以下几个接口。
* Debug.Log：向控制台输出普通信息（白色）。
* Debug.LogWarning：向控制台输出警告信息（黄色）。
* Debug.LogError：向控制台输出错误信息（红色）。

需要注意的是，Unity本身在运行场景时会向控制台输出警告信息和错误信息，但不会输出普通信息，所以一般情况下我们输出普通信息即可。
```cs
using UnityEngine;
using System.Collections;

public class LogDemo : MonoBehaviour 
{
	void Start () 
	{
		Debug.Log ("普通信息");
		Debug.LogWarning("警告信息");
		Debug.LogError("错误信息");
	}
}
```
运行场景后，在控制台窗口中可以看到输出的信息，如下图所示。

{% asset_img 6.png %}

# 游戏对象的操作

Unity中的所有实体都属于游戏对象，比如Unity自带的立方体、球体以及场景中使用的模型等，而对游戏对象的操作以及游戏对象之间的一切交互都需要使用脚本来完成。

## 创建游戏对象

创建游戏对象的方式有以下两种：
* 第一种为将导入工程后的模型放入Hierarchy视图或者Scene视图中，好处是完全可视化的操作。
* 第二种则是在代码里动态地创建和删除游戏对象，这种处理方式灵活性比较高。

本例在游戏视图中添加了两个按钮：“创建立方体”和“创建球体”。点击其中一个按钮，将在游戏中动态添加立方体对象或者球体对象。为了让创建的立方体对象与球体对象具有物理属性，比如质量、重力和碰撞等，我们需要为其添加刚体组件（Rigidbody）。

```cs
using UnityEngine;
using System.Collections;

public class ObjDemo : MonoBehaviour
{
	void OnGUI()
	{
		if (GUILayout.Button("创建立方体", GUILayout.Height(50)))
		{
			// 设置该模型默认为立方体
			GameObject obj = GameObject.CreatePrimitive(PrimitiveType.Cube);
			// 为对象添加一个刚体，赋予物理属性
			obj.AddComponent<Rigidbody>();
			//	赋予对象的材质红色
			obj.GetComponent<Renderer>().material.color = Color.red;
			// 设置对象的名称
			obj.name = "Cube";
			//	设置此模型的位置坐标
			obj.transform.position = new Vector3(0, 5f, 0);
		}
		if(GUILayout.Button("创建球体", GUILayout.Height(50)))
		{
			// 设置该模型默认为球体
			GameObject obj = GameObject.CreatePrimitive(PrimitiveType.Sphere);
			// 为对象添加一个刚体，赋予物理属性
			obj.AddComponent<Rigidbody>();
			// 赋予对象的材质绿色
			obj.GetComponent<Renderer>().material.color = Color.green;
			// 设置对象的名称
			obj.name = "Sphere";
			// 设置此模型的位置坐标
			obj.transform.position = new Vector3(0, 5f, 0);
		}
	}
}
```
运行游戏后点击按钮，立方体或球体将被创建。物体由于被添加了刚体具有物理属性，将受到重力的作用自由落体。

{% asset_img 7.png %}

* GameObject.CreatePrimitive()函数：创建一个游戏对象并指定一个Unity自带的模型，如立方体、球体以及圆柱体等。

* AddComponent()函数：用于给该游戏对象添加一个组件，也就是所添加的脚本必须继承自Component类。<span style="color:red;">值得注意的是，有些组件是依赖于其他组件的，当添加组件时，其依赖的组件会被自动添加，例如关节HingeJoint依赖刚体Rigidbody，当添加HingeJoint时，如果游戏对象没有Rigidbody，将会自动添加Rigidbody到游戏对象上。</span>

* renderer.material.color：设置渲染材质的颜色。

* transform.position：设置该游戏对象的位置，这个属性设置的是位于世界坐标系下的位置。如果需要设置物体坐标系下的位置，即相对于父节点下的位置，则用到的是transform.localPosition。

## 获取游戏对象

在脚本中获取游戏对象的方式一共有两种：
* 第一种为在代码里声明对象，在Inspector属性栏里指定游戏对象；
* 第二种是通过对象名称获取对象。

### 通过指定

主要是通过在代码里声明一个公开的游戏对象，然后在Inspector属性栏里指定游戏对象。这种方法获取的游戏对象一般是预制体或者是场景中已经存在的对象。

```cs
using UnityEngine;
using System.Collections;

public class GetObjDemo : MonoBehaviour 
{
	// 声明名为obj的游戏对象
	public GameObject obj;
}
```

### 通过对象名称获取对象

在代码中使用CameObject.Find(string name)即可找到对应名称的游戏对象。只需要名字即可，不需要知道游戏对象的路径信息，使用方法如代码所示。
```cs
using UnityEngine;
using System.Collections;

public class GetObjByName : MonoBehaviour 
{
	private GameObject obj;

	void Start()
	{
		// 寻找整个场景中名为Cube的游戏对象并赋予obj变量
		obj = GameObject.Find("Cube");
	}
}
```

## 添加组件与修改组件

新创建的游戏对象本身不具备任何属性，自然没有功能作用。为了让它具备一些功能，就必须给它添加游戏组件。常用的组件有物理类、网格类、粒子类等。

添加游戏组件时，需要使用AddComponent()方法。而删除组件的时候需要使用Object.Destroy()方法，参数为需要删除的游戏对象或游戏组件。如果删除的是某一游戏对象，对象中所有的组件都会被一并删除。

本例中，我们首先在Scene视图中创建一个空的立方体对象，然后为其添加渲染组件。运行游戏后，在Game视图中点击“添加颜色”按钮或者“添加贴图”按钮将为该立方体对象添加颜色或贴图。

```cs
using UnityEngine;
using System.Collections;

public class AddComponentDemo : MonoBehaviour 
{
	public Texture texture; // 需要在Inspector指定贴图
	private GameObject obj;
	private Renderer render;

	void Start()
	{
		// 获取游戏对象
		obj = GameObject.Find("Cube");
		// 获取该对象的渲染器
		render = obj.GetComponent<Renderer>();
	}

	void OnGUI()
	{
		if (GUILayout.Button("添加颜色",GUILayout.width(100), GUILayout.Height(50)))
		{
			// 修改渲染颜色为红色
			render.material.color = Color.red;
		}
		if(GUILayout.Button("添加贴图", GUILayout.Width(100), GUILayout.Height(50)))
		{
			// 添加组件贴图
			render.material.mainTexture = texture;
		}
	}
}
```

在上述代码中,render.material引用为当前脚本绑定对象的材质，直接为其赋值即可修改对象材质。render.material.color为材质的颜色，render.material.mainTexture为材质的主要贴图。运行效果如图所示。

{% asset_img 8.png %}

## 发送广播与消息

在游戏对象之间使用广播传递消息是游戏对象之间互动的一种快捷的方式。

主要是通过GameObject.SendMessage(string methodName, object value = null, SendMessageOptions options = SendMessageOptions.RequireReceiver)函数发送的。方法是向该游戏对象上的所有MonoBehaviour脚本发送消息。第一个参数是消息的名称，所有MonoBehaviour脚本里与该名称同名的方法将被调用；第二个参数是向该方法传递的参数；第三个参数是是否必须有接收方法的选项，一般选不要求接收方法即可。

新建一个场景，然后新建一个游戏对象并命名为Sender，添加脚本。新建一个游戏对象，命名为Receiver并添加脚本，最后将Sender脚本的receiver指定为Receiver对象。

```cs
using UnityEngine;
using System.Collections;

public class SendDemo : MonoBehaviour
{
	public GameObject receiver;

	void Start () 
	{
		// 向本脚本所属的游戏对象发送ShowNumber消息并传递参数100
		receiver.SendMessage("ShowNumber", 100,  SendMessageOptions.DontRequireReceiver);
	}
}

```

```cs
using UnityEngine;
using System.Collections;

public class ReceiveDemo : MonoBehaviour 
{
	// 消息发送后，ShowNumber函数被自动调用
	void ShowNumber(int number)
	{
		Debug.Log("收到的数字是" + number);
	}
}
```
运行场景，Sender成功向Receiver发送消息并输出至控制台。

## 克隆游戏对象

克隆游戏对象与创建游戏对象不同，创建游戏对象是创建一个全新的游戏对象，需要另外添加组件来赋予功能。而克隆游戏对象通常是克隆具有一定功能的现成的对象。如果这个现成对象已经保存为文件的话，则称之为预制体。<span style="color:red;">一般来说，克隆的执行效率较高。</span>比如游戏中发射的子弹，每颗子弹对象是一样的，所以每次发射子弹直接克隆一个子弹，然后赋予新的位置速度等信息即可。在代码中，<span style="color:red;">需要使用Instantiate()函数克隆游戏对象。</span>
	
首先创建一个Cube立方体，然后添加Rigidbody组件。下一步，将Cube立方体从Hierarchy视图拖曳至Project视图。一个预制体就创建好了。对着预制体点击鼠标右键“Show in Explorer”，会在文件夹中显示该文件，名称是“Cube.prefab”。prefab就是Unity中所有预制体的后缀名。我们已经创建好了预制体，接着选中Scene场景视图中的Cube对象，右键“Delete”将其删除。

接着在游戏中通过代码动态克隆这个预制体，如代码所示。
```cs
using UnityEngine;
using System.Collections;

public class CloneDemo : MonoBehaviour
{
	public GameObject prefab;

	void Start()
	{
		// 克隆预制体
		GameObject obj = Instantiate(prefab) as  GameObject;
		// 设置游戏对象obj的位置
		obj.transform.position = new Vector3(0, 3, 0);
	}
}
```
创建一个新的空游戏对象，命名为“Manager”，添加该脚本。

添加后，在Inspector窗口里CloneDemo栏目下可以看到Prefab指定条，将之前创建好的Cube预制体拖曳上去。

运行游戏，在屏幕中间出现我们通过代码克隆出的Cube游戏对象。Cube对象因为添加了刚体而向下自由落体。