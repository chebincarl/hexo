---
layout: title
title: Light Probes（2）
date: 2019-05-11 17:44:18
categories: Unity
tags: 官方文档
---
思考并回答以下问题：
1.光照探针是干嘛用的？不使用会怎么样？

<!--more-->

Lightmapping adds greatly to the realism（真实感） of a scene by capturing realistic（逼真的） bounced light as textures which are “baked” onto the surface of static objects. However, due to the nature（性质） of lightmapping, it can only be applied to non-moving objects marked as Lightmap Static.

<!--more-->

<span style="color:red;">While realtime and mixed mode lights can cast direct light on moving objects, moving objects do not receive bounced light from your static environment unless you use light probes.（移动物体不主动接收间接光照） </span>Light probes store information about how light is bouncing around in your scene.Therefore as objects move through the spaces in your game environment, they can use the information stored in your light probes to show an approximation（近似） of the bounced light at their current position.

> A simple scene showing bounced light from static scenery.

{% asset_img LightProbes-MovingObjects-1.jpg %}

In the above scene, as the directional light hits the red and green buildings, which are static scenery, bounced light is cast into the scene. The bounced light is visible（可见） as a red and green tint on the ground directly in front of each building. Because all these models are static, all this lighting is stored in lightmaps.

<span style="color:red;">When you introduce（采用） moving objects into your scene, they do not automatically receive bounced light. </span>In the below image, you can see the ambulance (a dynamic moving object) is not affected by the bounced red light coming off（从...反射出来） the building. Instead, its side is a flat grey color. This is because the ambulance is a dynamic object which can move around in the game, and therefore cannot use lightmaps, because they are static by nature. The scene needs Light Probes so that the moving ambulance can receive bounced light.

> The side of the ambulance is a flat grey color, even though it should be receiving some bounced red light from the front of the building.

{% asset_img LightProbes-MovingObjects-2.png %}

To use the light probe feature to cast bounced light onto dynamic moving objects, you must position light probes throughout（到处，各处） your scene, so that they cover the areas of space that moving objects in your game might pass through.

The probes you place in your scene define a 3D volume. The lighting at any position within this volume is then approximated on moving objects by interpolating between the information baked into the nearest probes.

> Light probes placed around the static scenery in a simple scene. The light probes are shown as yellow dots. They are shown connected by magenta（品红色） lines, to visualise the volume that they define.

{% asset_img LightProbes-MovingObjects-3.png %}

Once you have added probes, and baked the light in your scene, your dynamic moving objects will receive bounced light based on the nearest probes in the scene. Using the same example as above, the dynamic object (the ambulance) now receives bounced light from the static scenery, giving the side of the vehicle a red tint, because it is in front of the red building which is casting bounced light.

> The side of the ambulance now has a red tint because it is receiving bounced red light from the front of the building, via the light probes in the scene.

{% asset_img LightProbes-MovingObjects-4.png %}

When a dynamic object is selected, the Scene view will draw a visualisation of which light probes are being used for the interpolated bounced light. The nearest probes to the dynamic object are used to form a tetrahedral（四面体）volume, and the dynamic object’s light is interpolated from the four points of this tetrahedron.

> The light probes that are being used to light a dynamic object are revealed in the scene view when the object is selected, connected by yellow lines to show the tetrahedral volume.

{% asset_img LightProbes-MovingObjects-5.jpg %}

As an object passes through the scene, it moves from one tetrahedral volume to another, and the lighting is calculated based on its position within the current tetrahedron.

> A dynamic object moving through a scene with light probes, showing how it passes from one tetrahedral light probe volume to another.

{% asset_img LightProbes-MovingObjects-6.gif %}