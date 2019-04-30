---
layout: title
title: Character Controller
date: 2019-03-24 23:57:34
categories: Unity
tags: 官方文档
---
The Character Controller is mainly used for third-person or first-person player control that does not make use of Rigidbody physics.
角色控制器组件主要用于不使用刚体物理的第三人称或第一人称控制器。

<!--more-->

是一个组件，和Rigidbody一样。
添加之后会自带胶囊碰撞器。

{% asset_img 1.png %}

slope 坡度
steep  陡
indicated 指示
degrees 度
essentially 基本相同
jitter 抖动

## Property
| Property  | Function  |
| :------------ | :------------ |
| Slope Limit  | Limits the collider to only climb slopes that are less steep (in degrees) than the indicated value.  | 
| Step Offset  | The character will step up a stair only if it is closer to the ground than the indicated value. This should not be greater than the Character Controller’s height or it will generate an error.   角色控制器的能爬上的台阶高度   |
| Skin width  | Two colliders can penetrate each other as deep as their Skin Width. Larger Skin Widths reduce jitter. Low Skin Width can cause the character to get stuck. A good setting is to make this value 10% of the Radius. 两个对撞机可以在它们的皮肤宽度上穿透彼此。较大的皮肤宽度可减少抖动。低皮肤宽度可能导致角色卡住。 一个好的设置是将此值设为Radius的10％ |
| Min Move Distance  | If the character tries to move below the indicated value, it will not move at all. This can be used to reduce jitter. In most situations this value should be left at 0.  |
| Center  | This will offset the Capsule Collider in world space, and won’t affect how the Character pivots.  角色控制器的中心位置 |
| Radius  | Length of the Capsule Collider’s radius. This is essentially the width of the collider. 胶囊碰撞器半径的长度。 这基本上是对撞机的宽度 |
| Height  | The Character’s Capsule Collider height. Changing this will scale the collider along the Y axis in both positive and negative directions. 角色的胶囊碰撞器的高度。更改此设置将沿Y轴在正方向和负方向缩放碰撞器 |


{% asset_img 2.png %}
The Character Controller

halt 暂停
constrained 受限
Fine tuning 微调
turn on a dime 灵活

## Details
The traditional Doom-style first person controls are not physically realistic. The character runs 90 miles per hour, comes to a halt immediately and turns on a dime. Because it is so unrealistic, use of Rigidbodies and physics to create this behavior is impractical and will feel wrong. 

The solution is the specialized Character Controller. It is simply a capsule shaped Collider which can be told to move in some direction from a script. The Controller will then carry out the movement but be constrained by collisions. 
解决方案是专门的角色控制器。 它只是一个胶囊形状的碰撞器，可以通过脚本被告知向某个方向移动。然后，控制器将执行运动但受到碰撞的约束。


It will slide along walls, walk up stairs (if they are lower than the Step Offset) and walk on slopes within the Slope Limit.
它将沿着墙壁滑下来，走上楼梯（如果它们低于Step Offset）并在Slope Limit内的斜坡上行走

The Controller does not react to forces on its own and it does not automatically push Rigidbodies away.
控制器不会对施加在其身上的力作出反应，也不会自动推动刚体。

If you want to push Rigidbodies or objects with the Character Controller, you can apply forces to any object that it collides with via the OnControllerColliderHit() function through scripting.
如果要使用角色控制器推动刚体或对象，可以使用脚本通过OnControllerColliderHit()函数将力施加到它碰撞的任何对象。

On the other hand, if you want your player character to be affected by physics then you might be better off using a Rigidbody instead of the Character Controller.
另一方面，如果你想让你的玩家角色受到物理影响，那么你可能最好使用Rigidbody而不是Character Controller。

Fine-tuning your character
You can modify the Height and Radius to fit your Character’s mesh. It is recommended to always use around 2 meters for a human-like character. You can also modify the Center of the capsule in case your pivot point is not at the exact center of the Character.
您可以修改高度和半径以适合角色的网格。 建议总是使用大约2米的人类角色。如果您的枢轴点不在角色的正中心，您也可以修改胶囊的中心。

Step Offset can affect this too, make sure that this value is between 0.1 and 0.4 for a 2 meter sized human.
Step Offset也会对此产生影响，对于2米大小的人，请确保此值介于0.1和0.4之间。

Slope Limit should not be too small. Often using a value of 90 degrees works best. The Character Controller will not be able to climb up walls due to the capsule shape.
坡度限制不应太小。通常使用90度的值最好。由于胶囊形状，角色控制器将无法爬上墙壁。

## Don’t get stuck
critical 关键的
penetrate 穿透

The Skin Width is one of the most critical properties to get right when tuning your Character Controller. 
Skin Width是调整Character Controller时最关键的属性之一。

If your character gets stuck it is most likely because your Skin Width is too small. The Skin Width will let objects slightly penetrate the Controller but it removes jitter and prevents it from getting stuck.

皮肤宽度会让物体稍微穿透控制器，但它会消除抖动并防止卡住。

It’s good practice to keep your Skin Width at least greater than 0.01 and more than 10% of the Radius.
保持皮肤宽度至少大于0.01且大于半径的10％是一种很好的做法。

We recommend keeping Min Move Distance at 0.

See the Character Controller script reference here

You can download an example project showing pre-setup animated and moving character controllers from the Resources area on our website.

## Hints 提示
Try adjusting your Skin Width if you find your character getting stuck frequently.

The Character Controller can affect objects using physics if you write your own scripts.
如果编写自己的脚本，Character Controller可以使用物理影响对象。

The Character Controller can not be affected by objects through physics.
角色控制器不会受到物理对象的影响。

Note that changing Character Controller properties in the inspector will recreate the controller in the scene, so any existing Trigger contacts will get lost, and you will not get any OnTriggerEntered messages until the controller is moved again.
请注意，在inspecto视图中更改“Character Controller”的属性将在场景中重新创建控制器，因此任何现有的触发器联系都将丢失，并且在再次移动控制器之前，您不会收到任何OnTriggerEntered消息。


下面是脚本
CharacterController<-Collider<-Component<-Object

继承自Collider，其实就一个碰撞器。

Description
A CharacterController allows you to easily do movement constrained by collisions without having to deal with a rigidbody.
CharacterController允许您轻松地进行受碰撞约束的运动，而无需处理刚体

A CharacterController is not affected by forces and will only move when you call the Move function. It will then carry out the movement but be constrained by collisions.

| Properties  |   |
| :------------ | :------------ |
| center  | The center of the character's capsule relative to the transform's position.  |
| collisionFlags  | 在最后一次CharacterController.Move调用期间，胶囊的哪个部分与环境相撞。  |
| detectCollisions  | Determines whether other rigidbodies or character controllers collide with this character controller (by default this is always enabled).  |
| enableOverlapRecovery  | Enables or disables overlap recovery. Enables or disables overlap recovery. Used to depenetrate character controllers from static objects when an overlap is detected.  |
| height  | The height of the character's capsule.  |
| isGrounded  | Was the CharacterController touching the ground during the last move? |
|   |   |
| minMoveDistance  | Gets or sets the minimum move distance of the character controller.  |
| radius  | The radius of the character's capsule.  |
| skinWidth  | The character's collision skin width.  |
| slopeLimit  | The character controllers slope limit in degrees.  |
| stepOffset  | The character controllers step offset in meters.  |
| velocity  | The current relative velocity of the Character (see notes).  |