---
layout: title
title: 运行时构建NavMesh
date: 2019-06-10 18:17:13
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
1.去动态生成一个场景，看看NavMesh是如何生成和变化的

<!--more-->

导航系统对于动态生成的场景，就不能直接用了。但是Navigation系统有一个官方的扩展，虽然没有直接包含在Unity引擎的安装包里，但是是Unity官方提供的。


点击此链接可以查看对应的组件

在Navigation窗口中可以看到这样的链接字样，也可以打开对应的Github网站(https://github.com/Unity-Technologies/NavMeshComponents)。

# NavMeshComponent

NavMeshComponent给NavMesh的构建和使用提供了很多额外的控制功能，可以在运行时和编辑器中使用。

安装
NavMeshComponent并没有直接包含在Unity中，需要通过这个链接（https://github.com/Unity-Technologies/NavMeshComponents）安装下载。具体的步骤是：

1、打开上面的链接
2、点击图中的Release





3、选择你的Unity版本对应的NavMeshComponent版本。





4、下载解压以后，可以使用Unity打开该工程。如果要在已有的工程中使用，可以将Assets/NavMeshComponents目录复制到已有工程中。另外 Assets/Examples 目录中有示例场景。

组件介绍
NavMeshComponents包含4个组件，分别是：
这里我们介绍导航系统的四个组件：

NavMeshSurface - 基于一种agent类型构建NavMesh。

NavMeshModifier - 基于层次结构影响NavMesh区域类型的NavMesh生成。

NavMeshModifierVolume - 根据体积影响NavMesh区域类型的NavMesh生成。

NavMeshLink - 为一个agent类型连接相同或不同的NavMesh表面。

这些组件包含了构建和使用NavMeshes的高级控制，可以用于运行时和编辑时。

NavMesh Surface
NavMeshSurface组件用来设置针对一种agent类型的可行走的区域，并且在该区域构建NavMesh。在构建NavMesh时，会自动排除NavMeshAgent和NavMeshObstacle物体。

使用方法
点击菜单栏中 GameObject > AI > NavMesh Surface。点击后会创建一个空物体，上面包含NavMeshSurface组件。一个场景可以包含多个NavMeshSurface物体。

也可以给任何物体添加NavMeshSurface组件，这种情况非常适合一个物体的所有子物体来构建NavMesh的情况。





属性详解
Agent Type 使用这个NavMeshSurface的Agent类型。用于NavMesh的生成和寻路。

Collect Objects 用于生成NavMesh的物体。

All 所有active的物体

Volume 选中该选项时，可以在场景中编辑一个盒子，盒子所包围的物体用来生成NavMesh。

Volume选项时的包围盒

Volume选项时的包围盒

Children NavMeshSurface组件所在物体的所有active子物体。

Include Layers 可以选择用于生成NavMesh的layer。

Use Geometry 用于生成NavMesh的几何体

Render Meshes 使用物体的Mesh和Terrain数据

Physics Colliders 使用物体的Collider和Terrain数据（使用这个选项时，agent可以移动到物体边缘更近的地方）

Advanced settings 高级设置
Default Area 设置该NavMesh的Area

Override Voxel Size 设置Voxel尺寸

Override Tile Size 为了使烘焙过程能并行处理并且内存效率更高，场景被分割成多个小块（瓦片）。NavMesh上可见的白线是瓦片的边界。
默认瓦片大小为256个Voxel，这个值在内存使用效率和NavMesh分块之间提供了良好的折衷。
要更改此默认瓦片大小，可以选中此复选框，然后在 Tile Size 字段中输入切片大小的体素数。

瓦片越小，NavMesh就越碎片化。这有时会导致agent走的不是最佳路径。NavMesh雕刻也在瓦片上执行。如果有很多障碍物，通常可以设置更小的瓦片尺寸（例如大约64到128个体素）来加速雕刻。如果计划在运行时烘焙NavMesh，请使用较小的瓦片大小来降低最大内存占用。

Build Height Mesh 暂不支持

NavMesh Modifier
这个组件可以修改该物体及子物体的NavMesh构建属性。这个组件会影响该物体及所有的子物体。如果子物体上也有NavMesh Modifier，则该NavMesh Modifier会重写自己及其子物体的属性。

NavMesh Modifier会影响NavMesh的构建，所以如果NavMesh Modifier的属性进行了修改，需要重新构建NavMesh。





属性详解
Ignore From Build 选中时该物体及其所有子物体不会被构建进NavMesh。

Override Area Type 修改该物体及其所有子物体的Area类型。

Affected Agents 该组件影响的Agents。例如，你可以针对特定的Agent排除特定的障碍物。

NavMesh Modifier Volume
和NavMesh Modifier类似，但是此组件可以通过设置一个包围盒来修改包围盒中的物体的NavMesh属性。

NavMesh Link
NavMesh Link可以在两个NavMesh之间创建链接。可以是点到点的链接，也可以是缝隙之间的链接（会沿着入口边缘最近位置跨越链接）。





属性详解
Agent Type 可以使用这个NavMeshLink的Agent类型。

Start Point 起点。相对于GameObject的位置。
End Point 终点。相对于GameObject的位置。

Swap按钮 交换起点和终点的位置。

Align Transform按钮 将GameObject的位置移到起点和终点的中心，并且朝向终点的位置。

Width 宽度。如果为0则通过两个点连接，如果大于0通过两条线连接。

Cost Modifier 非负值时，通过这个链接的代价是Cost Modifier乘以终点的距离。

Bidirectional 双向。不勾选时只能单向寻路。

Area Type 设置该link的area类型。

# 总结

这些基本上就是NavMesh相关组件的一些介绍和用法。可以更灵活的在运行时创建、使用NavMesh。
