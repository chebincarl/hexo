---
layout: title
title: Glossary
date: 2019-04-30 10:20:57
categories: Unity
tags: 官方文档
---
词汇

<!--more-->

| 英文  | 含义  |
| :------------ | :------------ |
| Culling Mask  | Allows you to includes or omit（忽略） objects to be rendered by a Camera, by Layer.  |
| Cubemap  | A collection of six square（正方形）textures that can represent the reflections in an environment or the skybox drawn behind your geometry. The six squares form the faces of an imaginary cube that surrounds an object; each face represents the view along the directions of the world axes (up, down, left, right, forward and back).  |
| GI Cache  | The cached intermediate（中间） files used when Unity precomputes real-time GI, and when baking Static Lightmaps, Light Probes and Reflection Probes
.Unity keeps this cache to speed up computation.  |
| Light Probe  | Light probes store information about how light passes through space in your scene. A collection of light probes arranged within a given space can improve lighting on moving objects and static LOD scenery within that space布置在给定空间内的light probes的集合可以改善移动物体上的照明以及该空间内的静态LOD景物.  |
| Mesh  | The main graphics primitive of Unity. Meshes make up a large part of your 3D worlds. Unity supports triangulated or Quadrangulated polygon meshes. Nurbs, Nurms, Subdiv surfaces must be converted to polygons.  |
| Reflection Probe  | A rendering component that captures a spherical view of its surroundings in all directions, rather like a camera. The captured image is then stored as a Cubemap that can be used by objects with reflective materials一个渲染组件，可以捕捉所有方向的周围环境的球形视图，就像一个摄像头。然后将捕获的图像存储为Cubemap，其可以被具有反射材质的对象使用.  |
| Mesh Filter  | A mesh component that takes a mesh from your assets and passes it to the Mesh  Renderer for rendering on the screen.  |
| Mesh Rendere  | A mesh component that takes the geometry from the Mesh Filter and renders it at the position defined by the object’s Transform component.  |
| Deferred shading  | A rendering path that places no limit on the number of lights that can affect a GameObject. All lights are evaluated per-pixel, which means that they all interact correctly with normal maps and so on所有灯都按像素进行评估，这意味着它们都可以与法线贴图正确交互，依此类推. Additionally, all lights can have cookies and shadows. |