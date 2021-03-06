---
layout: title
title: Light Probes（1）
date: 2019-05-11 17:44:14
categories: Unity
tags: 官方文档
---
思考并回答以下问题：
1.光照探针和光照贴图有什么异同点？
2.光照探针两个主要的用途是什么？

<!--more--> 

Light Probes provide a way to capture and use information about light that is passing through the empty space in your scene.

Similar to lightmaps, light probes store “baked” information about lighting in your scene. The difference is that while lightmaps store lighting information about light hitting the surfaces in your scene, light probes store information about light passing through empty space in your scene.

> An extremely simple scene showing light probes placed around two cubes

{% asset_img LightProbes-0.png %}

Light Probes have two main uses:

The primary use of light probes is to provide high quality lighting (including indirect bounced light) on moving objects in your scene.

The secondary use of light probes is to provide the lighting information for static scenery when that scenery is using Unity’s LOD system.

When using light probes for either of these two distinct（不同的，独特的） purposes, many of the techniques you need to use are the same. It’s important to understand how light probes work so that you can choose where to place your probes in the scene.