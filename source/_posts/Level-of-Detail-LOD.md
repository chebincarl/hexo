---
layout: title
title: Level of Detail(LOD)
date: 2019-04-30 11:51:40
categories: Unity
tags: 官方文档
---
When a GameObject
 in the Scene
 is far away from the Camera
, you can’t see very much detail, compared to when the GameObject is close to the Camera. And even though you can’t see the detail on a distant GameObject, Unity uses the same number of triangles to render it at both distances.

<!--more-->

To optimize rendering
, you can use the Level Of Detail (LOD) technique. It allows you to reduce the number of triangles rendered for a GameObject as its distance from the Camera increases. You use several Meshes and optionally a Billboard Asset
, which all represent the same GameObject with decreasing detail in the geometry. Each of the Meshes contains a Mesh Renderer
 component and represents a ‘Mesh
 LOD level’, while the Billboard

 Asset
 has a Billboard Renderer component and represents a ‘Billboard LOD level’.

As long as your GameObjects aren’t all close to the Camera at the same time, LOD reduces the load on the hardware and improves the rendering performance.

Working with LOD levels
In Unity, you use the LOD Group
 component to set up LOD rendering for a GameObject. The images below demonstrate how the LOD levels change according to distance from the Camera.

Image 1: Camera at LOD 0 shows a large number of small triangles in the Mesh
Image 1: Camera at LOD 0 shows a large number of small triangles in the Mesh
Image 1 shows the first level, LOD 0. This level is the closest to the Camera, and therefore the most detailed LOD level. For example, many first-level LODs are active when the GameObject’s height fills 50% or more of the screen’s height.

{% asset_img 1.png %}

Image 2: Camera at LOD 1 shows the Mesh with far fewer triangles and they are much larger in size
Image 2: Camera at LOD 1 shows the Mesh with far fewer triangles and they are much larger in size
Image 2 shows the next level, LOD 1. This level is farther away from the Camera, and therefore is a lower LOD level. For example, many LOD Groups use three levels, where LOD 1 is active when the GameObject fills between 25% and 49% of the screen height, and LOD 2 is active when the GameObject fills less than 25% of the screen height.

Because the arrangement of LOD levels depends on the target platform and available rendering performance, you can set maximum LOD levels and a Lod Bias Quality setting in Unity. The Lod Bias determines whether to favor higher or lower LOD levels at threshold distances.

Naming convention for importing Meshes
When you import a set of LOD Meshes, Unity automatically creates an LOD group for the GameObject with appropriate settings if you follow this naming convention:

Your set of Meshes have file names ending in _LOD and a number ranging from 0 to the total number of LOD levels minus one. For example, if the base name for your Mesh is Player, name your files Player_LOD0, Player_LOD1 and Player_LOD2 to generate a Player GameObject with three LOD levels.
Name the most detailed Mesh file _LOD0.
Make sure the rest of the Mesh file names increment corresponding to decreasing detail.
Set Billboard LOD levels to have the highest numerical prefix (because they always have the least detailed level).
You can use as many LOD levels as you need.