---
layout: title
title: 网格Mesh
date: 2019-04-08 09:27:42
categories: Unity
tags: Unity游戏开发技术详解与典型案例
---
Unity提供了一个Mesh类，允许通过脚本来创建和修改Meshes的类。通过Mesh类生成或修改物体的网格能够做出非常酷炫的物体变形特效。

<!--more-->

## 网格过滤器（Mesh Filter）

网格过滤器从资源中拿出网格并将其传递给网格渲染器（Mesh Renderer），用于在屏幕上渲染。网格过滤器中有一个重要的属性“Mesh”用于储存物体的网格数据。在导入模型资源时，Unity会自动创建一个网格过滤器组件。


# Mesh属性和方法介绍

网格过滤器组件中有一个重要的属性“Mesh”。“Mesh”是网格过滤器实例化的Mesh，Mesh中有一些用于储存物体的网格数据的属性和生成或修改物体网格的方法。

(1)Mesh中有一些用于储存物体的网格数据的属性，这些属性主要用于储存网格各种数据的数组。
属性的详细说明如下表所示。

|  <center>属性</center> |  <center>说明</center> |
| :- | :- |
|  vertices |  网格的顶点数组 |
|  normals | 网格的法线数组  |
|  tangents | 网格的切线数组  |
|  uv |  网格的基础纹理坐标 |
|  uv2 | 如果存在，这是为网格设定的第二个纹理坐标  |
|  bounds | 网格的包围体  |
|  colors |  网格的顶点颜色数组 |
|  triangles | 包含所有三角形顶点索引的数组  |
|  vertexCount |  网格中顶点的数量(只读的) |
|  subMeshCount | 子网格的数量，每种材质都有一个独立的网格列表  |
|  boneWeights |  每个顶点的骨骼权重 |
|  bindposes |  绑定的姿势，每个索引绑定的姿势使用具有相同索引的骨骼 |

(2)Mesh中有生成或修改物体网格的方法，这些方法主要用于设置储存网格各种数据的数组。
方法的详细说明如下表所示。

| <center>方法</center>  | <center>说明</center>  |
| :- | :- |
| Clear  | 清空所有顶点数据和所有三角形索引  |
| RecalculateBounds  | 重新计算从网格包围体的顶点  |
| RecalculateNormals  | 重新计算网格的法线  |
| Optimize  | 显示优化的网格  |
| GetTriangles  |  返回网格的三角形列表 |
| SetTriangles  |  为网格设定三角形列表 |
| CombineMeshes  |  组合多个网格到同一个网格 |

## Mesh的使用

网格包括顶点和多个三角形数组。三角形数组仅仅是顶点的索引数组，每个三角形包含三个索引。每个顶点可以有一条法线、两个纹理坐标以及颜色和切线，显然这些是可选的，但是也可以去掉。所有的顶点信息被储存在单独的同等规格的数组中。

通过为顶点数组赋值并为三角形数组赋值来新建立一个网格，通过获取顶点数组修改这些数据并把这些数据放回网格来改变物体形状。调用Clear函数在赋予新的顶点值和三角形索引值之前是非常重要的，因为Unity总是检查三角形的索引值，判断它们是否超出边界。

## 使用Mesh使物体变形

(1)新建一个场景，命名为“test”。
(2)创建地形，光源的具体参数如图10-11所示。

(3)创建水。选中Assets文件夹，单击鼠标右键选择"Import Package->Water(Pro Only)"导入标准水资源包，然后拖曳Daylight 
Water到场景中。

(4)创建两个空对象，分别命名为“zhang”和“sanjiao”。具体步骤为GameObject->Create Empty。为两个空对象添加网格过滤器，具体步骤为选中对象然后依次单击“Component->Mesh->Mesh Filter”。

(5)为两个空对象的网格过滤器组件设置网格属性。将Assets\Meshes文件夹下的sanjiao.FBX和zhang.FBX模型文件中的网格“Box01”分别拖曳到sanjiao和zhang对象的网格过滤器组件的“Mesh”属性中。如图10-15所示。

(6)创建一个空对象，并且命名为“g1”，调整该对象的位置和大小，具体参数如图10-16所示。为g1对象添加网格过滤器，然后为g1对象添加网格渲染器（具体步骤为选中对象然后依次单击菜单“Component->Mesh->Mesh Renderer”）。

(7)为g1对象添加纹理，将Assets\Textures文件夹下的纹理文件“wenli.tga”拖曳到g1对象上，这时g1对象的网格渲染器的“material”属性就设置为“wenli”材质，如图10-18所示。然后按照相同的方法再创建5个对象。



(8)在Scripts文件夹中，创建脚本并命名为"XiFen.cs"。本脚本主要用于控制物体变形，脚本代码如下：
```cs
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class XiFen : MonoBehaviour
{
	Mesh mesh; // 物体的网格对象
	int time; // 用于记录时间
	public GameObject[] g; // 包含网格的对象数组
	Mesh[] m; // 网格对象数组

	public List<Vector3> vertice; // 网格的顶点数组
	public List<int> triangle; // 包含所有三角形顶点索引的数组 
	public List<Vector2> uv; // 网格的基础纹理坐标 
	public List<Vector3> normal; // 网格的法线数组
	public List<Vector4> tangent; // 网格的切线数组
	bool bian = true; // 物体一次变形是否完成
	int s = 0;  // 物体变形形状标志位 

	void Start()
	{
	}

	void Update() 
	{
	}
}
```
口第5-8行的主要功能是声明变量。主要声明了物体的网格对象、包含网格的对象数组以及网格对象数组等变量。在下面控制物体变形的代码中会用到这些变量。

口第9-15行的主要功能是声明变量。主要声明了用于储存网格数据的各个数组以及物体一次变形是否完成标志位和物体变形形状标志位。

口第16-18行实现了Start方法的重写，该方法在游戏加载时执行。主要功能是游戏加载时细化物体的网格。

口第19-21行实现了Update方法的重写，该方法系统每帧调用一次，主要功能是通过不断改变网格数据使物体不断变形。

(9)在XiFen.cs脚本中通过场景加载时系统调用Start方法来实现细化物体的网格. Start方法的具体代码如下。