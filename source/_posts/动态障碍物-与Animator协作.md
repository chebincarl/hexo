---
layout: title
title: 动态障碍物/与Animator协作
date: 2019-06-10 18:03:59
tags:
---
思考并回答以下问题：
1.动手试试动态障碍物和导航与Animator的协作

<!--more-->

NavMesh烘焙只适用于静态的场景。对于动态生成、可移动的障碍物，可以通过NavMeshObstacle组件实现。

# <span style="color:#039BE5;">NavMeshObstacle组件</span>

NavMeshObstacle组件用于动态生成和可移动的障碍物。当障碍物移动时，NavMeshAgent会尽量躲避它。当障碍物静止时，它会在NavMesh上雕刻一个洞，类似烘焙出来的障碍物，此时Agent会重新计算寻路的路线。

{% asset_img 1.png %}

<span style="color:blue;">Shape</span> 障碍物的几何形状，可选项有Box和Capsule。
<span style="color:blue;">Center</span> 障碍物的几何形状中心相当于物体轴心的偏移。
对于Box：
&emsp;<span style="color:blue;">Size</span> 障碍物的几何形状的尺寸。
对于Capsule：
&emsp;<span style="color:blue;">Radius</span> 半径
&emsp;<span style="color:blue;">Height</span> 高度
<span style="color:blue;">Carve</span> 勾选此选项后，Nav Mesh Obstacle会在NavMesh中创建一个洞。

&emsp;<span style="color:blue;">Move Threshold</span> 障碍物移动超过这个阈值设置的值时，Navigation系统才会更新障碍物在NavMesh上雕刻的洞。

&emsp;<span style="color:blue;">Time To Stationary</span> 经过设置的时间后障碍物才会被视为静止。

&emsp;<span style="color:blue;">Carve Only Stationary</span> 只在障碍物静止时才在NavMesh上雕刻一个洞。

## 细节
Nav Mesh Obstacle可以通过两种方式影响Nav Mesh Agent在场景中的导航：

** 阻碍 **
如果未启用Carve，则Nav Mesh Obstacle的默认行为类似于Collider。Agent尝试避免碰撞Nav Mesh Obstacle，当接近时，它们会与Nav Mesh Obstacle碰撞。避障行为非常简单并且半径也很短。因此Agent可能无法在复杂的Nav Mesh Obstacles环境中找到方向。此模式最适用于障碍物不断移动的情况（例如，车辆或角色）。

** 雕刻 **
当Curve启用时，静止的障碍物会在NavMesh上雕刻一个洞，移动的障碍物会是阻挡物。当障碍物在NavMesh上雕刻一个洞时，Agent能在复杂的环境中找到路。对于经常会阻碍玩家移动的障碍物（例如，集装箱或油桶），最好打开Curve，同时这些障碍物可以被玩家或其他游戏事件（如爆炸）移动。

## 移动Nav Mesh Obstacle

当移动的距离超过<span style="color:red;">Carve > Move Threshold</span>设置的距离时，Unity会将Nav Mesh Obstacle视为移动。当Nav Mesh Obstacle移动时，雕刻的洞也会移动。但是，为了减少CPU开销，只在必要时重新计算雕刻的洞。计算的结果可在下一帧更新中使用。重新计算逻辑有两个选项：

* 只有当Nav Mesh障碍物静止时才会雕刻
* 当Nav Mesh障碍物移动时雕刻

** 只有当Nav Mesh Obstacle静止时才雕刻 **
这是默认行为。要启用它，请勾选Nav Mesh Obstacle组件的Carve Only Stationary复选框。在此模式下，当障碍物移动时，雕刻的孔被移除。当障碍物停止移动并且已经静止超过“雕刻静止时间”（Carving Time To Stationary）设置的时间时，它被视为静止，并且更新雕刻的孔。当Nav Mesh障碍物移动时，Nav Mesh Agent会避免使用碰撞躲避，但不会在障碍物周围寻路。

Carve Only Stationery通常是性能方面的最佳选择，并且容易与物理系统一起使用。

** Nav Mesh Obstacle移动时雕刻 **
要启用此模式，需要取消选中Nav Mesh Obstacle组件的Carve Only Stationary复选框。如果未选中此选项，则当障碍物移动的距离超过Carving Move Threshold设置的距离时，会更新雕刻的孔。此模式适用于大型缓慢移动的障碍物（例如，步兵避开的坦克）。

注意：Nav Mesh Obstacle对NavMesh的影响存在一帧延迟。


# <span style="color:#039BE5;">寻路与Animator</span>

敌人的动画怎么和寻路结合呢？现在角色只会飘来飘去的，看着好吓人

{% asset_img 2.gif %}
<center><font color="gray">人物漂移</font></center>

这个问题其实要协调NavMeshAgent和Animator，如果同时使用NavMeshAgent和开启RootMotiond的Animator，会有冲突，因为两个组件都会更新物体的transform。一般有两个解决方案：

* Animation跟随Agent
* Agent跟随Animation

需要注意的是只能选取一种，让信息单向流动，否则可能会造成信息的混乱，很难调试。

## <span style="color:#00ACC1;">Animation跟随Agent</span>

NavMeshAgent组件中有一个属性velocity，是角色移动的速度。可以用这个速度作为Animator的输入，来粗略地控制动画。这种方式比较简单，但是如果速度不匹配的话可能会造成角色的滑步。示例代码如下：
```cs
using UnityEngine;
using UnityEngine.AI;

[RequireComponent(typeof(NavMeshAgent))]
public class EnemyMovement : MonoBehaviour
{
    public Transform Player;

    Animator _animator;
    private NavMeshAgent _agent;

    void Start () {
        _animator = GetComponent<Animator>();
        _agent = GetComponent<NavMeshAgent>();
    }

    void Update ()
    {
        _agent.SetDestination(Player.position);

        var velocity = _agent.desiredVelocity;
        // 将世界坐标的速度转换为角色本地坐标系的速度
        velocity = transform.InverseTransformDirection(velocity);

        _animator.SetFloat('speedX', velocity.x);
        _animator.SetFloat('speedZ', velocity.z);
    }
}
```

## <span style="color:#00ACC1;">Agent跟随Animation</span>

这种方式控制比较精确，但是相对复杂一些。首先要禁用掉 <span style="color:blue;">NavMeshAgent.updatePosition</span> 和 <span style="color:blue;">NavMeshAgent.updateRotation</span>，这样NavMeshAgent就不会控制角色的Transform进行移动。然后使用模拟的agent下一帧的位置(<span style="color:blue;">NavMeshAgent.nextPosition</span>) 和当前位置<span style="color:blue;">animation root (Animator.rootPosition</span>) 来计算Animator的输入参数。

示例代码如下：
```cs
using UnityEngine;
using UnityEngine.AI;

[RequireComponent(typeof(NavMeshAgent))]
[RequireComponent(typeof(Animator))]
public class LocomotionSimpleAgent : MonoBehaviour
{
    Animator anim;
    NavMeshAgent agent;
    Vector2 smoothDeltaPosition = Vector2.zero;
    Vector2 velocity = Vector2.zero;

    void Start()
    {
        anim = GetComponent<Animator>();
        agent = GetComponent<NavMeshAgent>();
        // 设置agent不会更新角色的位置
        agent.updatePosition = false;
    }

    void Update()
    {
        // 根据agent模拟的下一帧的位置nextPosition，计算变化
        Vector3 worldDeltaPosition = agent.nextPosition - transform.position;

        // 将世界位置的变化转换到物体的本地坐标系
        float dx = Vector3.Dot(transform.right, worldDeltaPosition);
        float dy = Vector3.Dot(transform.forward, worldDeltaPosition);
        Vector2 deltaPosition = new Vector2(dx, dy);

        // 平滑处理
        float smooth = Mathf.Min(1.0f, Time.deltaTime / 0.15f);
        smoothDeltaPosition = Vector2.Lerp(smoothDeltaPosition, deltaPosition, smooth);

        // 计算移动的速度
        if (Time.deltaTime > 1e-5f)
            velocity = smoothDeltaPosition / Time.deltaTime;

        bool shouldMove = velocity.magnitude > 0.5f && agent.remainingDistance > agent.radius;

        // 更新animator的参数
        anim.SetBool('move', shouldMove);
        anim.SetFloat('velx', velocity.x);
        anim.SetFloat('vely', velocity.y);

        // agent可能和角色出现分离的现象，通过下面的代码进行修正
        if (worldDeltaPosition.magnitude > agent.radius)
            transform.position = agent.nextPosition - 0.9f * worldDeltaPosition;
    }

    void OnAnimatorMove()
    {
        // 根据navmesh的高度更新角色的高度
        Vector3 position = anim.rootPosition;
        position.y = agent.nextPosition.y;
        transform.position = position;
    }
}
```
# 总结

一般情况下用第一种方式会比较简单，第二种方式更精确一些，也更难理解