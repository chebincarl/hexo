---
layout: title
title: Unity常用英文
date: 2019-04-04 09:48:12
tags: Unity
---
常用英文的翻译

<!--more-->

cast 投射
gizmo 可视化辅助工具。
trigger 触发
overlap 重叠
culling 剔除
mask 面具，遮罩
Threshold 阈值
entry 条目 可以用来命名List变量，如logEntries
tint 着色
delta 增量
Axis 轴
Magnitude 大小
mapped 映射 
Conventional 常规 
arrow keys 方向键 
linear 线性
Additive 添加物
Vertex 顶点
duration 持续时间
Lerp 线性插值
Frustum 视锥体
cutscene 过场动画
obligation  义务
PostProcessing 后期处理
tiling 平铺
unlit 无灯光的
elapsed  过去
Slerp 球面线性插值
rect 矩形
Projectile 抛射体
formula 公式，方案，方法
host 托管
compound 复合，合成
polygon 多边形
Portals 入口
stretch 伸展
solid color 纯色
Rotate和rotation是两种东西
Obstacle 障碍
Agent 代理
mute 静音
embedded 嵌入的，内含的
cascade 级联
Volume 体积
Portals 入口
Visualization 可视化
toraue 扭矩

## 刚体选项设置

| 英文  | 中文  | 含义  |
| :------------ | :------------ | :------------ |
| Mass  | 质量  | 设置游戏对象的质量大小  |
| Drag  | 阻力  | 设置游戏对象在运动时受到空气阻力的大小。数值0表示无空气阻力，阻力越大游戏对象运动越慢，阻力极大时游戏对象则会停止运动  |
| Angular Drag  | 角阻力  | 设置游戏对象受到扭矩力时受到的空气阻力的大小。数值0表示无空"风
    力,阻力越大游戏对象运动总慢。用力极大时游戏对象则会停止运动  |
| Use Giavity  | 使用重力  | 表示游戏对象会受到重力的影响  |
| Is Kinematic  | 是否开启动力学  | 开启动力学后,游戏对象将只能通过Transform属性对其进行控制,1不A受物群居
    抗的影响  |
| Interpolate  | 插值  | 此选项用于控制刚体运动的抖动情况,共包括三个子选项,
1. Nonc:无差值:
2. Interpolate,内考值,表示将基于上一触的Transform来平滑此次的Transform:3, Extrapolate:外梦值,表示将基于下一触的Transform来平滑北次的Transfonn  |
| Collision Delection  | 碰操检测  | 用于难,较高运动速度的游戏对象无法与其他游戏对象发生碰撞,共包括三个子去项:
1. Discrete:离散碰接检测,此选项为被选中的游戏对象与场景中其他所有的碰操体进行醒撞检测:
2 Continuous连按碳接检测,该选项用于检测被选中游观对象与动态碰撞体的碰掉,使用连续碰障检测模式来检测与网络继操体的碰撑;
3. Continuots, Dynamic:连校动态碳推检测,此选项用于检测连续碳潍模式成是连续动态碳擦模式对象的检测  |
| Constraints  | 约束  | 此选项用于约束M体的运动,共包括两个子选项;
1. Freere Position,冻结位置
2. Freere Rotation:冻结膜转  |

# 碰撞体

在游戏制作的过程中，游戏对象要根据游戏的需要进行物理属性的交互。为此，Unity的物理组件为游戏开发者提供了碰撞体组件，碰撞体是物理组件的一类，它与刚体一起，促使碰撞的发生。

## 使用碰撞体

在Unity的物理组件的使用过程中，碰撞体需要与刚体一起添加到游戏对象上才能触发碰撞。因此，在游戏制作的过程中，没有添加刚体组件的碰撞体也会相互穿过。

为游戏对象添加碰撞体后，其使用需要注意以下两点:

(1)碰撞体与刚体

游戏对象物理效果的产生，是碰撞体与刚体共同作用的结果。值得注意的是，刚体一定要绑定在一个被碰撞到的对象上才能够产生碰撞效果，而碰撞体则不需要一定绑定刚体。

(2)物理材质

Unity中内置了多种物理材质供游戏开发者进行选择，如下图所示，物理材质的参数设置如下表所示。