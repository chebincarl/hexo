---
layout: title
title: Unity 3D的物理引擎
date: 2019-03-15 21:03:32
tags: Unity
---
Unity 3D游戏引擎内置了由NVIDIA出品的PhysX物理仿真引擎。该引擎是世界三大引擎之一，具有高效低耗的特点，且仿真程度极高。物理引擎通过为刚性物体赋予真实的物理属性的方式来计算它们的运动、旋转和碰撞反应。在开发过程中只需要简单地操作就可以使物体按照物理运动规律进行运动。

<!--more-->

# 刚体
## 刚体特性
刚体（Rigidbody）是在使用物理引擎的过程中经常用到的一个组件。<label style="color:red">任何一个非角色对象</label>，如果希望能够受作用力及扭矩进行仿真运动，都需要挂载一个刚体组件。刚体设置了许多属性、变量和相关方法，下面将分别进行介绍。

### 刚体属性

为了便于开发人员控制物理系统，Unity 3D提供了多个属性接口（Inspector面板），开发人员可以通过更改这些参数来实现对物体物理状态的控制。在实际的开发过程中，这些参数都被详细地罗列在属性面板中，开发人员可以很方便地对这些属性进行修改。接下来将对这些属性进行详细讲解。
    
(1)质量（Mass）

质量属性表示刚体的质量，其数据类型是float，默认值为1。一般来说，大部分的物体的Mass属性值应该设置为接近0.1且不超过10.0才符合日常生活中的感官感受。刚体的质量并没有单位，在开发过程中读者要通过保持物体与物体之间的质量之比来提高其物理仿真度。

(2)阻力（Drag）

这里的阻力指的是物体的移动阻力，物体进行任意方向的移动都会受到Drag的影响。该属性的数据类型是float，默认值为0。Drag的方向与物体运动的方向相反，对物体的移动起阻碍作用。通过对Drag设置不同的值，可以分别模拟出羽毛和石头掉落的情景。

(3)旋转阻力（Angular Drag）

Angular Drag与Drag类似，也是阻碍物体运动的一个力。该属性的数据类型是float，默认值是0.05，如果读者将该属性设置为0，则物体在因受瞬时力而旋转后，将不会停止旋转运动，此属性值越高，物体的角速度衰减就会越严重。
    
(4)使用重力（Use Gravity）

Use Gravity这个属性是以布尔值的形式存在的，其初始值为true，这一属性设为false时，物体将不受重力的作用，但其他非重力则正常进行计算，可以模拟出物体在外太空等特殊场合的无重力状态，对于某些特殊的场景的物理模拟是非常有用的。

(5)是否遵循运动学（Is Kinematic）

这个属性表示的是该游戏对象是否遵循牛顿运动学物理定律，其数据类型是bool，初始值为false。值得注意的是，该属性值为true时表示该对象的运动只受脚本和动画的影响，作用力、关节和碰撞都不会对其产生任何作用。只有将该属性值设置为false时，才能正常调用物理计算。

另外，虽然当该属性值为true时物体不受物理定律的约束，但是该物体还是会影响其他物体，改变其他物体的运动状态，在游戏开发中此属性经常会被使用到。想像一下，在第一人称射击类游戏中，敌人被击杀后会倒地不动，就是因为这个敌人对象中Rigidbody组件上的Is Kinematic属性被赋为true。
    
(6)插值方式（Interpolate）

该属性表示的是该物体运动的插值模式。默认情况下，Interpolate属性值是空（None），此时物体的物理计算不进行插值，所需的值取最近计算的值。开发人员可以选择内插值（Interpolate）或外插值（Extrapolate）两种模式进行插值。

<label style="color:red">由于在Unity 3D中物理模拟和画面渲染并不同步</label>，如果不进行插值处理，所计算得到的物理数据会是上一个物理模拟时间点的数据，而插值是获取近似当前渲染时间点数据的一种手段，然而，插值得到的值并非真实值，会产生轻微抖动的现象，建议在开发过程中只对主要游戏对象进行插值处理。

(7)碰撞检测模式（Collision Detection）

假设一个高速运动的物体，其两个相邻物理模拟时间点所进行的位移大于被碰撞物体的厚度，且本身厚度足够小，则该物体将有可能直接穿过被碰撞物体，这种现象称为碰撞检测的穿透。为防止这种现象的出现，Unity 3D提供了三种不同的碰撞检测模式，用于应对不同情况下的碰撞检测。

本属性默认使用占用资源较少的离散模式（Discrete），对于静止或运动速度较慢的物体建议使用该模式；而对于高速运动或体积较小的物体建议使用连续模式（Continuous）；被使用了连续检测模式的物体所撞击的物体，则应该使用动态连续模式（Continuous Dynamic）。

(8)约束条件（Constraints）

该属性表示的是该物体的位移或旋转是否受到物理定律的约束。默认状态下，物体的任意方向的位移和任意轴的旋转都是受物理定律的约束的，开发人员通过设置指定方向的位移和指定轴的旋转，可以灵活地设置物体的状态，达到自己想要的效果。

以上属性在属性面板中的位置如下图所示。在实际开发过程中，除了可以在代码中对这些属性进行修改，还可以在属性面板中直接对其进行修改，以提高开发效率。

{% asset_img 1.png %}

## 刚体变量
为了获取和更改物体的运动状态，Unity 3D还预留了多个变量接口，这些接口简化了对物体运动的处理，使得开发人员能够轻易地对物体的运动状态进行干预。接下来将具体地介绍这些变量。

(1)角速度（angularVelocity）

此变量表示刚体的角速度向量，其数据类型为Vector3；该向量的方向即为刚体旋转轴的方向，旋转方向遵循左手定则；该角速度的大小为向量的模，单位为rad/s。非必要情况下，不建议对此变量进行过多的干预，直接修改该值会造成一定的模拟失真。下面的代码可以实现一个静止物体的旋转，具体代码如下。
```cs
void Start()
{
    GetComponent<Rigidbody>().angularVelocity = Vector3.up; // 使物体以y轴为旋转轴进行旋转，一直在旋转
}
```

(2)位移速度（velocity）

此变量表示物体的位移速度值，在Unity 3D中单位1表示现实生活中的1m。在开发过程中，直接修改此变量的值并不是一种明智的做法，因为Unity 3D中物体的运动要经过非常复杂的计算才能使得物体的运动自然平滑，如果有外加干预，会使得物体的运动模拟失真。
下面的代码可以实现一个物体的速度骤增，以实现瞬移效果。具体代码如下。
```cs
void Start()
{
    GetComponent<Rigidbody>().velocity = Vector3.up; //给物体赋以向上的速度    
}
```

(3)重心（centerOfMass）

通过调低物体的重心，可以使物体不易因其他物体的碰撞或作用力而倒下。若不对重心进行设置，Unity 3D会对重心位置自动进行计算，其计算基础为物体所挂载的碰撞器。值得注意的是，物体的重心的坐标以模型坐标系为准，而不是世界坐标系。
下面的代码可以实现修改一个物体的重心，具体代码如下。
```cs
void Start()
{
    GetComponent<Rigidbody>().centerOfMass = Vector3.up; // 修改物体的重心位置
}
```
(4)碰撞检测开关（detectCollisions）

这个变量是一个非常有用的变量，该变量默认是true，在必要的时候可以关闭。在实际开发过程中，有一些物体并不是时刻都需要进行碰撞检测的，此时，通过设置本属性而不是移除刚体组件进行实现，对提高程序的运行效率有明显的作用。

下面的代码演示了如何关闭物体碰撞检测，具体代码如下。
```cs
void Start()
{
    GetComponent<Rigidbody>().detectCollisions = false; // 关闭碰撞检测    
}
```

(5)惯性张量（inertiaTensor）

该变量用来描述物体的转动惯量，其数据类型为Vector3。如果不对该值进行设置和干预，它将通过挂载在物体对象上的碰撞器组件自动进行计算。
下面的代码可以实现给物体赋一个自定义的惯性张量，具体代码如下。

```cs
void Start()
{
    GetComponent<Rigidbody>().inertiaTensor = Vector3.one; // 修改惯性张量    
}
```
(6)惯性张量旋转（inertiaTensorRotation）

该变量指物体惯性张量的旋转值，其数据类型为Quaternion，即四元数。如果不对该值进行设置和干预，它将通过挂载在物体对象上的碰撞器组件自动进行计算。
下面的代码可以实现给物体赋一个原始惯性张量旋转值，具体代码如下：
```cs
void Start()
{
    GetComponent<Rigidbody>().inertiaTensorRotation = Quaternion.identity; // 修改惯性张量旋转   
}
```

(7)最大角速度（maxAngularVelocity）

该变量用于设置物体的最大角速度，其数据类型为float，单位为rad/s，只能为非负数，且数值可无限大，默认为7。最大角速度用来限制物体的旋转速度，使物体的旋转速度不至于过大。当物体的旋转向量的模大于该值时，则使物体旋转速度等于该值。该值在一些指定的情况下会特别有用。
下面的代码可以实现对物体最大角速度的修改，具体代码如下：
```cs
void Start()
{
    GetComponent<Rigidbody>().maxAngularVelocity = 1.9f; // 修改最大角速度    
}
```

(8)最大穿透速度（maxDepenetration Veloctiy）

当一个物体穿透其他碰撞器时，物体的速度会变得非常不稳定，此时通过设置本变量可以限制物体的速度，从而使得物体的运动变得平滑。该值的数据类型为float，只能为非负数，且数值可无限大，默认情况下为无限大。
下面的代码可以实现对物体最大穿透速度的修改，具体代码如下。
```cs
void Start()
{
    GetComponent<Rigidbody>().maxDepenetrationVelocity = 1.9f; // 修改最大穿透速度 
}  
```
(9)坐标（position）

该变量表示的是刚体在世界坐标系中的坐标，数据类型为Vector3。该变量与transform.postion具有完全不同的意义（切不可混淆乱用），前者代表物理模拟中的坐标，而后者指绘制场景中的坐标。两者的数值会尽量保持一致，但在高速运动的过程中，这两个变量的数值会有细微的差别。
下面的代码用于打印物体的刚体坐标位置值，具体代码如下。
```cs
void Start()
{
    Debug.Log(GetComponent<Rigidbody>().position);  // 打印刚体位置   
}
```
    
(10)旋转（rotation）

该变量表示的是刚体在世界坐标系中的旋转，数据类型为Quaterion。该变量与transform.rotation也具有完全不同的意义，前者代表物理模拟中的旋转值，而后者指绘制场景中的旋转值。两者的数值会尽量保持一致，但在高速旋转的过程中，这两个变量的数值会有细微的差别。
下面的代码用于打印物体的刚体旋转值，具体代码如下。
```cs
void Start()
{
    Debug.Log(GetComponent<Rigidbody>().rotation);  // 打印刚体旋转
}
```

## 刚体常用方法
在讲解了刚体的属性与变量之后，有必要讲解一下Unity提供的相关方法。

(1)给刚体施加力（AddForce）

此方法的方法签名为“public void AddForce(Vector3 force, ForceMode mode)”。此方法被调用时，将会向刚体施加一个沿着force方向的力，该力的类型为mode。ForceMode的类型包括计算重力的连续力、忽略重力的连续加速力、计算重力的瞬时力、忽略重力的瞬时力四种。
下面的代码实现了对物体施加竖直向上的力，具体代码如下。
```cs
void Start()
{
    GetComponent<Rigidbody>().AddForce(Vector3.up); // 缺省情况下，默认为Force
} 
```

(2)移动刚体（MovePosition）

此方法的方法签名为“public void MovePosition(Vector3 position)”。此方法被调用时，系统会根据指定的参数，将刚体移动到相对应的位置，其效果是物体的位置会因为刚体的移动而随之移动。该方法经常用于FixedUpdate方法中。
下面的代码实现了对物体的匀速平移操作，具体代码如下。
```cs
void FixedUpdate()
{
    GetComponent<Rigidbody>().MovePosition (transform.position + Vector3.right * Time.deltaTime); // 平移刚体
}
```

(3)旋转刚体(MoveRotation)

此方法的方法签名为“public void MoveRotation(Quaternion rot)”。此方法被调用时，系统会根据指定的参数，将刚体旋转到相对应的角度，其效果是物体的角度会因为刚体的旋转而随之变化。该方法经常用于FixedUpdate方法。
下面的代码实现了对物体的匀速旋转操作，具体代码如下。
```CS
void FixedUpdate()
{
	GetComponent<Rigidbody>().MoveRotation(transform.rotation * Quaternion. Euler(new Vector3(0, 100, 0) * Time.deltaTime));  //旋转刚体
}
```

(4)添加爆炸力(AddExplosionForce)

此方法的方法签名为“public void AddExplosionForce(float explosionForce, Vector3 explosionPosition, float explosionRadius, float upwardsModifier, ForceMode mode)”。其将在“explosionPosition”处产生模式为“mode”，大小为“explosionForce”的爆炸力，半径为“explosionRadius”，并在物体下方upwardsModifier向上施力。
下面的代码实现了产生一个爆炸力的操作，具体代码如下。
```cs
void Start()
{
	GetComponent<Rigidbody>().AddExplosionForce(19.0f, transform.position, 10, 1.5f, ForceMode.Force); // 添加爆炸力
}
```
如果将爆炸力的值设置为负数，则该方法可以模拟出引力的效果，使在半径之内的物体因爆炸力的作用，向中心点靠拢。

(5)在指定点施加力（AddForceAtPosition）

此方法的方法签名为“public void AddForceAtPosition(Vector3 force, Vector3 position, ForceMode mode)”。该方法被调用时，将在“position”处添加一个“mode”模式、“force”大小的力。这里的position是基于世界体系的坐标，读者应使position在物体之内，否则将会很难进行控制。
下面的代码实现了向物体施加一个力的操作，具体代码如下。
```cs
void FixedUpdate()
{
	GetComponent<Rigidbody>().AddForceAtPosition(Vector3.up, transform.position, ForceMode.Force); // 施加作用力
}
```

(6)施加相对力（AddRelativeForce）

此方法的方法签名为“public void AddRelativeForce(Vector3 force, ForceMode mode)”。调用此方法时，将向刚体施加一个沿着“force”方向的力，该力的模式为“mode”。在本方法中，force基于物体的模型坐标，与基于世界坐标的AddForce方法略有不同。
下面的代码实现了向刚体施加一个相对力的操作，具体代码如下。
```cs
void Start()
{
	GetComponent<Rigidbody>().AddRelativeForce(Vector3.up, ForceMode.Force); //施加相对力
}
```

(7)施加力矩（AddTorque）

此方法的方法签名为“public void AddTorque(Vector3 torque, ForceMode mode)”。该方法被调用时，将向刚体施加一个“torque”的力矩，其力模式为“mode”。通过此方法可以使物体受力矩的作用而进行运动。
下面的代码实现了向刚体施加一个力矩的操作，具体代码如下。
```cs
void Start()
{
	GetComponent<Rigidbody>().AddTorque(Vector3.up, ForceMode.Force); // 施加力矩
}
```
(8)施加相对力矩（AddRelativeTorque）

此方法的方法签名为“public void AddRelativeTorque(Vector3 torque, ForceMode mode)”。调用此方法时，将向刚体施加一个沿着“torque”方向的力矩，该力的模式为“mode”。在本方法中，torque基于物体的模型坐标，与基于世界坐标的AddTorque方法略有不同。
下面的代码实现了向刚体施加一个相对力矩的操作，具体代码如下。
```cs
void Start()
{
	GetComponent<Rigidbody>().AddRelativeTorque(Vector3.up, ForceMode.Force); // 施加相对力矩
}	
```

(9)计算相对刚体的最近点（ClosestPointOnBounds）

此方法的方法签名为“public Vector3 ClosestPointOnBounds(Vector3 position)”。调用此方法可以计算出在刚体所包含的三维空间内，与“position”距离最短的点的坐标。通过计算与刚体的最近点，可以在一些特殊场合实现所需效果。
下面的代码实现了打印相对刚体最近点的操作，具体代码如下。
```cs
void Update()
{
	Debug.Log(GetComponent<Rigidbody>().ClosestPointOnBounds(Vector3.zero)); // 打印最近点
}
```

(10)获取基于点坐标系的速度（GetPointVelocity）

此方法的方法签名为“public Vector3 GetPointVelocity(Vector3 worldPoint)”。给定一个基于世界坐标的点“worldPoint”，调用此方法可以计算出刚体在以“worldPoint”为原点的坐标系中的速度。
下面的代码实现了打印基于点坐标系的速度的操作，具体代码如下。
```cs
void Update()
{
	Debug.Log(GetComponent<Rigidbody>().GetPointVelocity(Vector3.up)); // 打印速度	
}
```
(11)获取基于相对点坐标系的速度（GetRelativePointVelocity）

此方法的方法签名为“public Vector3 GetRelativePointVelocity(Vector3 relativePoint)”。给定一个基于刚体模型坐标系的点“relativePoint”，调用此方法可以计算出刚体在以“relativePoint”为原点的坐标系中的速度。
下面的代码实现了打印基于相对点坐标系的速度的操作，具体代码如下。
```cs

```

	(12)确定是否处理休眠(IsSleeping),此方法的方法签名为"public bool IsSleeping()",调用此方法将返回一个bool类型的值,表示该刚体是否处于休眠状态。
下面的代码实现了打印休眠状态的操作,具体代码如下。
void Update () {
Debug. Log (GetComponent<Rigidbody> ().Issleeping);  //打印休眠状态
N。
	(13)设置密度(SetDensity),此方法的方法签名为"public void SetDensity(float density)",调用此方法将给刚体设置一个密度值,该密度值基于碰撞器的体积,而不是物体的体积。
下面的代码实现了设置刚体密度的操作,具体代码如下。

(14)强制休眠(Sleep)。
	此方法的方法签名为"public void Sleep()",调用此方法可将对应刚体强制进行休眠,不参与物理模拟计算。通过将不重要的物体进行强制休眠,可以节约大量的资源,以提高程序的运,行效率。
下面的代码实现了将刚体进行强制休眠,具体代码如下。
1 void Start () t
2 GetComponent<Riqidbody>().sleep();  //将刚体强制休眠,
	(15)唤醒(WakeUp)。此方法的方法签名为"public void WakeUpO",调用此方法可将处于休眠状态的刚体进行唤醒,使其重新加入物理模拟计算。下面的代码实现了使刚体不被休眠,具体代码如下。

	(16)扫描检测(SweepTest)。此方法的方法签名为"public bool SweepTest(Vector3 direction, outRaycastHit hitInfo, float maxDistance)",调用该方法时,将沿着direction的方向产生一条长度为maxDistance"的射线"hitInfo",若该射线碰撞到其他刚体,则返回true,否则返回false。第一个被检测到的刚体信息储存在"hitInfo"上。
	下面的代码实现了扫描检测的功能,具体代码如下。
23
RaycastHit rh = new RaycastHit();
void Update () {
	Debug. Log (GetComponent<Rigidbody> () . SweepTest (Vector3. forward, out rh, 10.0f) );//扫描结果
	(17)扫描检测所有(SweepTestAll)。此方法的方法签名为"public RaycastHit] SweepTestAll致(Vector3 direction, float maxDistance)",该方法与SweepTest类似,不同的是将会返回RaycastHit类型的数组,其中储存了"direction"方向所有检测到的刚体的信息。该数组的最大长度不超过128.
	下面的代码实现了扫描所有刚体的功能,具体代码如下。

	SweepTest和SweepTestAll方法都只能扫描到简单类型的碰撞器(sphere、cube.capsule ),而网格碰撞器则不适用使用本方法。关闭碰撞器的知识将在后面进行详细讲解。
6.1.2物理管理器:
	前面简单介绍了刚体的属性、变量和方法,接下来这一部分将讲解一下物理管理器(PhysicsManager)的相关内容。
Unity 3D作为一个优秀的游戏开发平台,其出色的管理模式是令人称赞的。在Unity 3D中不仅可以对单个分组进行属性设置,还可以对场景全局进行设置。这一部分的内容将会向读者详细讲解Unity 3D中场景的全局物理参数是如何设置的。
	1,物理管理器预览
	(1)打开Unity 3D开发平台,在菜单栏中选择"Edit"选项,会弹出图6-2所示的编辑列表。在编辑列表中,选择"Project Settings 选项,将弹出图6-3所示的全局设置列表。在全局设置列表中选择"Physics",即可进入物理管理器界面。
	(2)按照步骤1进行操作后,会在属性查看器中呈现出物理管理器的面板,如图6-4所示。读者可以在该面板中对当前项目的全局物理参数进行设置。
	2、物理管理器参数
	在前面的内容中,讲解了物理管理器的打开操作,读者应该能够熟练地调出物理管理器面板。接下来介绍的是物理管理器中的相关参数的含义和用法。
	(1)重力(Gravity)。该属性表示的是当前项目中的重力加速度,该参数将被应用于所有刚体。该属性3个数值分别指在x、y、2方向上的重力加速度,一般重力加速度是竖直向下的,所以只有y轴上有一个负值。默认情况下, y轴上的值大小为-9.81, x轴和2轴方向的值为0。
	(2)默认材质(Default Material)。该属性表示当物体没有被指定物理材质的时候,该物体的默认材质。默认状态下该属性是没有指定值的,也就是说在默认状态下创建的物体都是没有指定材质的,这是因为在Unity 3D中,每个物体的物理材质可能会有很大的不同,有时候指定默认材质并没有太大的意义。
	(3)反弹阈值(Bounce Threshold),该属性表示的是项目中的反弹阈值,该参数被应用于所有刚体。如果两个相互碰撞的物体的相对速度低于反弹阈值,则将不会进行反弹计算。通过合理设计该属性可以有效减少物理模拟过程中的抖动。这里所有的相对速度是指以其中一个刚体为参照物,另外一个刚体的速度值。
	(4)休眠阈值(Sleep Threshold )。该属性用来代替之前版本中的SleepVelocity.SleepAngularVelocity等值,实为刚体的能量值,其大小受刚体的平移速度和旋转速度所影响。设刚体能量为E,平移速度大小为v,角速度大小为A,则刚体的能量计算公式为E(V+VA) ×0.5.当刚体能量低于该阈值时,则进行休眠操作,
	(5)默认接触偏差(Default Contact Offset),当两个刚体的表面距离低于该值时,则认为两个刚体已经接触,并对其进行物理模拟计算。该属性只能为正数,不可为负数或0。在实际的物理模拟计算中,很难使两个刚体刚好无缝贴合,如果想对其进行碰撞检测,就必须有一个容差值,

使两个刚体能够进行物理模拟计算。
	(6)求解迭代次数(Solver Iteration Count)。该属性表示的是项目中的关节和连接计算过程中的迭代次数。该属性值决定了关节和连接的计算精度。求解迭代次数越高,其计算精度越高,但是过高的迭代次数会浪费过多的资源,不利于项目的流畅运行,一般来说此属性值设置为7就可以适用于大多数情况。
	(7)射线检测命中触发器(Raycasts Hit Triggers)。在Unity 3D中,有碰撞器和触发器之分,Unity 3D通过射线实现拾取。若一个物体上附加的是碰撞器,则碰撞拾取功能可一直正常使用;若一个物体上挂载的是触发器,则其碰撞拾取功能受本属性的限制,只有本属性值为true时,其拾取功能才能正常运行。
	(8)允许自适应力(Enable Adaptive Force),自适应力是PhysX所使用的一项特殊技术,它主要用于修正PhysX在模拟动态状况时不可避免的数值偏差。Unity 3D在5.0版本之后采用了PhysXSDK3版本的物理引擎,其在经典PhysXSDK 2.x的基础上进行了重新设计。并将Physx的自适应力设置为可切换的,而且在默认状态下是关闭状态。
	(9)层碰撞矩阵(Layer Collision Maxtrix),将碰撞检测进行分层是一个非常有用的功能。同一个物理层内的所有刚体可以相互进行碰撞检测,同时,开发人员可以指定不同的层与层之间进行碰撞检测,以适应不同的开发需求。而对于物理层的设置,则是通过层碰撞矩阵实现的。两个层交叉处便是设置这两个层是否可碰撞的标志位。