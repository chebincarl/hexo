---
layout: titlle
title: GI cache
date: 2019-05-15 10:40:45
categories: Unity
tags: 官方文档
---
思考并回答以下问题：


<!--more-->

The GI cache is used by the Global Illumination (GI) system to store intermediate files when precomputing real-time GI, and when baking Static Lightmaps, Light Probes and Reflection Probes. The cache is shared between all Unity projects on the computer, so projects with the same content and same version of the lighting system (Enlighten) can share the files and speed up subsequent builds.

You can manage the size, location, and compression of the cache using the GI Cache preferences. For more information, see the Preferences documentation.

** GI cache and lighting **

To ensure that the the lighting data loads from the GI cache in a very short amount of time when you reload your Scene, open the Lighting window (menu: Window > Lighting) and enable the Auto option next to the build button. This makes lightmap baking automatic, meaning that the lightmap data is stored in the GI cache.

In the Lighting window, you can clear the baked data in a Scene (disable the Auto option and choose Clear Baked Data from the Build button drop-down menu). This does not clear the GI Cache, because this would increase bake time afterwards.

You can share the GiCache folder among different machines. This can make your lighting build faster, because the files are downloaded from the GiCache folder instead of computed locally. Note that the build process isn’t optimized for slow network-attached storage (NAS), so test if your bake times are severely affected before moving the cache to NAS.