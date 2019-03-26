---
layout: title
title: Layers
date: 2019-03-12 22:30:19
tags: Unity
---
Layers are most commonly used by Cameras to render only a part of the scene, and by Lights to illuminate（照亮） only parts of the scene. But they can also be used by raycasting（光线投射） to selectively（有选择地） ignore（忽略） colliders（碰撞器） or to create collisions（n碰撞）.
<!--more-->

Creating Layers

The first step is to create a new layer, which we can then assign to（分配给） a GameObject. To create a new layer, open the Edit menu and select Project Settings->Tags and Layers.

We create a new layer in one of the empty User Layers. We choose layer 8.

Assigning（分配） Layers
Now that you have created a new layer, you have to（必须） assign the layer to one of the game objects.

In the tag manager we assigned the Player layer to be in layer 8.

Drawing only a part of the scene with the camera’s culling mask
Using the camera’s culling mask, you can selectively render objects which are in one particular layer. To do this, select the camera that should selectively render objects.
Modify the culling mask by checking or unchecking layers in the culling mask property.


Be aware that UI elements aren’t culled. Screen space canvas children do not respect the camera’s culling mask.


Casting Rays Selectively

Using layers you can cast（投射） rays（射线） and ignore colliders in specific layers. For example you might want to cast a ray only against the player layer and ignore all other colliders.
The Physics.Raycast function takes a bitmask（位掩码）, where each bit determines if a layer will be ignored or not. If all bits in the layerMask are on, we will collide against（相撞） all colliders. If the layerMask = 0, we will never find any collisions with the ray.

// C# example.
```cs
int layerMask = 1 << 8;
        
// Does the ray intersect（相交） any objects which are in the player layer.
if (Physics.Raycast(transform.position, Vector3.forward, Mathf.Infinity（无穷大）, layerMask))
    Debug.Log("The ray hit the player");
```


In the real world you want to do the inverse（相反的） of that however. We want to cast a ray against all colliders except those in the Player layer.

// C# example.
```cs
void Update () 
{
    // Bit shift（ [计算机] 使（数据）位移） the index of the layer (8) to get a bit mask
    int layerMask = 1 << 8;
        
    // This would cast rays only against colliders in layer 8.
    // But instead we want to collide against everything except layer 8. The ~ operator does this, it inverts（使倒置，使反转） a bitmask.
    layerMask = ~layerMask;
    
    RaycastHit hit;
    // Does the ray intersect any objects excluding the player layer
    if (Physics.Raycast(transform.position, transform.TransformDirection (Vector3.forward), out hit, Mathf.Infinity, layerMask)) 
{
        Debug.DrawRay(transform.position, transform.TransformDirection (Vector3.forward) * hit.distance, Color.yellow);
        Debug.Log("Did Hit");
    } 
else
 {
        Debug.DrawRay(transform.position, transform.TransformDirection (Vector3.forward) *1000, Color.white);
        Debug.Log("Did not Hit");
    }
}
```

When you don’t pass a layerMask to the Raycast function, it will only ignore colliders that use the IgnoreRaycast layer. This is the easiest way to ignore some colliders when casting a ray.
Note: Layer 31 is used internally（内部的） by the Editor’s Preview window mechanics. To prevent clashes（冲突）, do not use this layer.