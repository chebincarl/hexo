---
layout: title
title: Rendering Paths
date: 2019-05-04 15:15:33
categories: Unity
tags: 官方文档
---
Unity supports different Rendering Paths. You should choose which one you use depending on your game content and target platform / hardware.Different rendering paths have different performance characteristics（特点） that mostly affect Lights and Shadows. See render pipeline for technical details.

<!--more-->

思考并回答以下问题：
1.渲染路径是干嘛用的？
2.有哪些渲染路径？ 
3.区别是什么？

The rendering path used by your project is chosen on the Graphics window. Additionally, you can override it for each Camera.

If the graphics card can’t handle a selected rendering path, Unity automatically uses a lower fidelity（保真度） one. For example, on a GPU that can’t handle Deferred Shading, Forward Rendering will be used.

** Deferred Shading **

Deferred Shading is the rendering path with the most lighting and shadow fidelity, and is best suited if you have many realtime lights. It requires a certain level of hardware support.

For more details see the Deferred Shading page.

** Forward Rendering **

Forward is the traditional rendering path. It supports all the typical Unity graphics features (normal maps, per-pixel lights, shadows etc.).However under default settings, only a small number of the brightest lights are rendered in per-pixel lighting mode. The rest of the lights are calculated at object vertices or per-object.

For more details see the Forward Rendering page.

** Legacy Deferred **

Legacy Deferred (light prepass) is similar to Deferred Shading, just using a different technique with different trade-offs. It does not support the Unity 5 physically based standard shader.

For more details see the Deferred Lighting page.

** Legacy Vertex Lit **

Legacy Vertex Lit is the rendering path with the lowest lighting fidelity and no support for realtime shadows. It is a subset of Forward rendering path.

For more details see the Vertex Lit page.

NOTE: Deferred rendering is not supported when using Orthographic projection. If the camera’s projection mode is set to Orthographic, these values are overridden, and the camera will always use Forward rendering.