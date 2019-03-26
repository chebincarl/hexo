---
layout: title
title: 预制件（Prefab）资源的应用
date: 2019-02-27 22:37:14
tags: Unity
---
在一个项目的开发过程中经常会应用到预制件（prefab）资源。在场景的开发中会同时创建多个完全相同的游戏对象，如果一一创建会耗费大量的时间，并且也会耗费游戏资源，在管理上也会有一定的难度。这时就需要实例化预制件（prefab）来实现。

<!--more-->

## 预制件（prefab）资源的创建

在Assets菜单Create命令的选项中选择创建perfab，就会在资源项目列表中创建一个预制件（perfab）资源。此时的预制件（perfab）只是一个空壳，还需添加一些具体的游戏对象，具体操作步骤如下。

(1).通过菜单创建一个prefab。执行“Assets->Create->Prefab”命令，即可在项目资源列表中创建一个prefab，然后将其名字改为“BallPrefab”。

(2).在场景中创建一个球体。执行Create->3D Object->Sphere命令，即可在游戏组成列表中创建一个Sphere，然后将其改名为“Ball”。

(3).向项目中导入一个球面纹理图片资源。执行”Assets->Import New Asset..”命令，会立刻弹出一个Import New Assets对话框，在对话框中选中需要的球面纹理图片，单击“Import”按钮完成导入。

(4).为创建的Ball添加刚体属性和球体碰撞者属性，即先选中Ball，再执行“Component->Physics->Rigidbody“命令，即可为Ball添加刚体属性，再执行“Component->Physics->Sphere Collider”命令，即可为Ball添加球体碰撞属性。

(5).将导入的球面纹理图片资源添加到创建的Ball上面。选中纹理图片然后将其拖动到创建的Ball上面即可，添加的各个属性都将在属性查看器中显示出来。

(6).为刚创建的BallPrefab添加真实的游戏对象。这个操作跟5中一样，只要选中刚刚创建的Ball对象，然后将其拖到Assets面板中已经创建好的BallPrefab上即可，此时这个空的BallPrefab就具有了与Ball对象完全相同的属性。

### 通过prefab资源进而实例化对象

在实际的开发过程中，若要创建大量的重复化的资源，就需要使用到prefab资源。通过脚本编写程序实例化这些游戏对象，这样可以省去创建过程的时间，以及为各个游戏对象添加相同属性的烦琐操作，并且还会节省大量的游戏资源，提高项目的运行效率。

下面将通过一个实例化篮球的小案例来讲解prefab实例化的具体操作过程。

1.以上面的BallPrefab为例，在此有关prefab的具体创建过程就不再进行讲解。

2.编写脚本，实例化篮球，执行"Assets->Create->C# Script"命令，在项目中创建一个C# Script脚本，在此将脚本的名字改为"BallPrefabScript"，然后双击脚本进入脚本编辑器。对脚本进行编辑。
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
				BP[count++] = (Rigidbody)Instantiate (BallPrefab, new Vector3 (x-2.0f*k*i+4.0f*j*k, 2.0f, z-2.0f*1.75f*k*i), BallPrefab.rotation);
			}
		} 	
	}  
}
```
口第1-13行的主要功能是声明变量，主要声明了整型变量i.j，刚体BallPrefab, x、 y,z的坐标，刚体的行数及计数器，并且对相关的参数进行了赋值。在开发环境下的属性查看器中可以为各个参数指定资源或者取值。

口第14-22行的主要功能是对Sar方法进行了重写，初始化了刚体组数，然后先对i进行循环，再在i的循环中将j进行循环，最后在自定义坐标位置通过实例化刚体数组创建了10个球体，并且对10个球的位置按照一定的规律进行了排列。

3.将编写完的脚本挂载到摄像机上，然后对摄像机脚木属性中的各个变量参数进行设置。设置完毕后单击Unity集成开发环境的运行按钮，在游戏场景中会显示出实例化的效果。本案例中创建了10个BallPefab。

