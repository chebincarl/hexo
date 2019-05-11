---
layout: title
title: Reflection Probes（3）
date: 2019-05-11 11:51:27
categories: Unity
tags: 官方文档
---
思考并回答以下问题：
1.

<!--more-->

CG films and animations commonly feature highly realistic reflections（通常具有高度逼真的反射）, which are important for giving a sense of “connectedness” among the objects in the scene. However, the accuracy（准确性） of these reflections comes with a high cost in processor time and while this is not a problem for films, it severely limits the use of reflective objects in realtime games.

只使用一张反射贴图的缺点。
Traditionally, games have used a technique called reflection mapping to simulate reflections from objects while keeping the processing overhead（开销） to an acceptable level. This technique assumes that all reflective（反光） objects in the scene can “see” (and therefore reflect) the exact same surroundings. This works quite well for the game’s main character (a shiny car, say) if it is in open space but is unconvincing（不能令人信服的） when the character passes into different surroundings; it looks strange if a car drives into a tunnel but the sky is still visibly（明显的） reflected in its windows.

Unity improves on basic reflection mapping through the use of Reflection Probes, which allow the visual environment to be sampled（采样） at strategic points（关键点） in the scene. You should generally place them at every point where the appearance of a reflective object would change noticeably（明显地） (eg, tunnels, areas near buildings and places where the ground colour changes). When a reflective object passes near to a probe, the reflection sampled by the probe can be used for the object’s reflection map. Furthermore, when several probes are nearby, Unity can interpolate between them to allow for gradual（逐渐的） changes in reflections. Thus, the use of reflection probes can create quite convincing reflections with an acceptable processing overhead.

** How Reflection Probes Work **

The visual（视觉） environment for a point in the scene can be represented by a cubemap. This is conceptually（概念） like a box with flat images of the view from six directions (up, down, left, right, forward and backward) painted on its interior（内部） surfaces.

> Inside surfaces of a skybox cubemap (front face removed)

{% asset_img CubemapDiagram.svg %}

For an object to show the reflections, its shader must have access to the images representing the cubemap. Each point of the object’s surface can “see” a small area of cubemap in the direction the surface faces (ie, the direction of the surface normal vector). The shader uses the colour of the cubemap at this point in calculating what colour the object’s surface should be; a mirror material might reflect the colour exactly while a shiny car might fade and tint it somewhat.
对于显示反射的对象，其着色器必须能够访问表示立方体贴图的图像。物体表面的每个点都可以在表面朝向的方向（即表面法线向量的方向）上“看到”立方体图的一小块区域。着色器在此时使用立方体贴图的颜色来计算对象表面应该是什么颜色；镜面材料可能会完全反映颜色，而闪亮的汽车可能会褪色并略微着色

As mentioned above, traditional reflection mapping makes use of only a single cubemap to represent the surroundings for the whole scene. The cubemap can be painted by an artist or it can be obtained by taking six “snapshots” from a point in the scene, with one shot for each cube face. Reflection probes improve on this by allowing you to set up many predefined points in the scene where cubemap snapshots can be taken. You can therefore record the surrounding view at any point in the scene where the reflections differ noticeably.

In addition to its view point, a probe also has a zone of effect defined by an invisible box shape in the scene. A reflective object that passes within a probe’s zone has its reflection cubemap supplied temporarily by that probe. As the object moves from one zone to another, the cubemap changes accordingly.