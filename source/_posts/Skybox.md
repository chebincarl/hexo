---
layout: title
title: Skybox
date: 2019-03-24 23:53:14
tags: Unity
---
Skyboxes are a wrapper around your entire scene that shows what the world looks like beyond your geometry.

<!--more-->

{% asset_img 1.png %}

## Properties

| Property  | Function  |
| :------------ | :------------ |
| Tint Color  | The tint color  |
| Exposure  | Adjusts the brightness of the skybox.  |
| Rotation  | Changes the rotation of the skybox around the positive y axis.  |
| Front,etc  | The textures used for each face of the cube used to store the skybox. Note that it is important to get these textures into the correct slot.  |

## Details
Skyboxes are rendered around the whole scene in order to give the impression of complex scenery at the horizon.
Skyboxes围绕整个场景渲染，以给人以地平线上复杂风景的印象。

Internally skyboxes are rendered after all opaque objects; 
在所有不透明对象之后渲染内部天空盒

and the mesh 
used to render them is either a box with six textures, or a tessellated（棋盘格） sphere.

To implement a Skybox create a skybox material. Then add it to the scene by using the Window-> Rendering->Lighting Settings menu item and specifying your skybox material as the Skybox on the Scene tab.

Adding the Skybox Component
to a Camera is useful if you want to override the default Skybox. E.g. You might have a split screen game using two Cameras, and want the Second camera to use a different Skybox. To add a Skybox Component to a Camera, click to highlight the Camera and go to Component->Rendering->Skybox.

{% asset_img 2.png %}

Hints（提示）
If you have a Skybox assigned to a Camera
, make sure to set the Camera’s Clear mode to Skybox.
It’s a good idea to match your Fog color to the color of the skybox. Fog color can be set in the Lighting window.