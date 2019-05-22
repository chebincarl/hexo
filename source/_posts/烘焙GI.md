---
layout: title
title: 烘焙GI
date: 2019-05-22 15:07:44
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：

<!--more-->

采用Progressive Lightmapper烘焙中的场景

除了预计算实时GI这种照明技术外，我们还知道有一种照明方案叫Baked lightmaps（烘焙光照贴图）

什么是烘焙光照贴图？
烘焙光照贴图是将场景的光影信息提前计算，将计算的结果写入光照贴图中。这个计算是在游戏发布前进行的，在游戏运行时的光影信息通过一张或多张光照贴图直接读取，除了贴图占用的内存外，几乎不产生其他的计算资源消耗。

这是一种用空间换时间的技术，在保证场景图形效果的基础上，还能有非常高的运行效率，非常适合计算能力较低的平台，比如移动平台。

优点
通过光照贴图可以生成非常高质量的静态物体的阴影，并且运行时性能好，除了贴图占用的内存外没有额外开销

支持间接光照（从其他物体表面反射的光）

缺点
没有实时光照，这意味着无法在运行时修改光源的属性（颜色、强度、位置、旋转等）

从静态物体到动态物体的阴影只能使用Light Probe技术得到，而且分辨率很低

会增加打包体积大小和运行时的内存。相对于实时GI，光照贴图会包含更多的光照细节信息，会占用更多的空间。

烘焙光照贴图的流程
烘焙的整个流程大致如下：
1、场景环境设置
2、对场景打光
3、设置静态物体的Lightmap Static标签和Renderer参数
4、设置烘焙的参数
5、烘焙，根据烘焙结果调整灯光、烘焙的参数重新烘焙直到达到满意的效果

其中前三步已经在上周的27、28课学习，本节课我们从第4步开始学习。

4 烘焙参数设置
要想启用烘焙GI，首先需要在Lighting面板勾选Baked Global Illumination。

Lighting Mode 光照模式，该参数决定了Mixed模式的灯光对物体阴影的使用的模式。默认值是Shadowmask，在下方会详细解释。特别注意，如果修改了这个值，场景全部的光照贴图需要重新烘焙，如果Auto Generate勾选，重新烘焙会自动开始。

Lighting Mode
该选项只对Light组件的Mode设置为Mixed时生效。

Shadowmask
Lighting Mode的默认值。
选中Shadowmask时，可以从菜单栏Edit > Project Settings > Quality 中选择Shadowmask的模式。

Distance Shadowmask 在Shadow Distance距离内使用实时渲染的阴影，超过距离的地方使用烘焙阴影（动态物体无法被投射阴影，可用Light Probe解决，后续会讲到）。这样在Shadow Distance内静态物体可以对动态物体产生实时阴影。
适用情景举例：
开放的大场景，阴影几乎都是投射到地面上，阴影也可以投射到移动的角色身上

Shadowmask 动态物体可以投射实时阴影到静态物体上，但是静态物体无法投射阴影到动态物体上（可用Light Probe解决，后续会讲到）。静态物体之间的阴影通过烘焙贴图渲染。
适用情景举例：
几乎全静态物体的场景

Shadow Distance
渲染阴影是非常消耗性能的一件事。Shadow Distance表示距离摄像机在这个距离范围内的物体才会产生实时阴影，以优化实时运行的效率。

Baked Indirect
这种模式下，Unity只烘焙间接光到光照贴图，不烘焙阴影。阴影在Shadow Distance内会采用实时渲染的方式，超出Shadow Distance的阴影不渲染。

适用情景举例
场景是房间内的室内射击或冒险游戏，玩家观看距离有限，所有可见的内容通常在阴影距离内；此模式对于有雾的室外场景也很有用，可以使用雾来隐藏距离外缺失的阴影。

Subtractive
此模式下，直接光会被烘焙到光照贴图中，因此在运行时不会进行任何直接光照运算。静态的物体因此没有高光、反射效果。

Realtime Shadow Color 该属性只在Subtractive模式下显示。可以用来设置实时阴影的颜色。

适用情景举例
块状渲染的场景如卡通渲染风格的场景，并且几乎没有动态物体。

Lightmapping Settings
Lightmapper的两个选择

Unity的烘焙GI由子系统提供支持，有两个选择，默认是Progressive，需要配置的参数略微不同。

两种系统没有明显的优劣，但是Progressive Lightmapper是较新的系统，据非官方比较结果，效果比Enlighten好。一般建议使用Progressive Lightmapper。

Progressive Lightmapper（渐进式光照贴图器）
这个子系统仅用于烘焙GI。顾名思义，这个系统以渐进的方式烘焙光照贴图，在烘焙过程中可以看到过程中的变化，右下角也会显示预估的时间。

Progressive的默认值

使用Progressive Lightmapper时，主要配置的参数是：

Lightmap Resolution 光照贴图分辨率，决定了光影的精细程度。越大的值精细度越高，相应烘焙时间越长，生成的光照贴图也越多。

Direct Samples 直接光采样的数量。提高该值可以提高光照贴图的质量，但是会增加烘焙时间。

Indirect Samples 间接光采样的数量。对于室外场景，一般100就足够了。对于室内复杂场景，可以增加这个值直到达到你预期的效果。提高该值可以提高光照贴图的质量，但是会增加烘焙时间。

Bounces 间接光反弹的次数。对于大多数场景，默认值2次就足够了。但是对于复杂室内场景，可能需要更多的次数。

其他参数可以在下方查看详细解释。

Enlighten
Enlighten可以用于烘焙GI，也是实时GI所使用的子系统(实时GI只能用这个子系统)。

Enlighten的默认值

使用Enlighten时，主要配置的参数是：

Lightmap Resolution 光照贴图分辨率，决定了光影的精细程度。越大的值精细度越高，相应烘焙时间越长，生成的光照贴图也越多。

Indirect Resolution 间接光计算中，每个单位使用几个像素表示。增加该值可以提高间接光的视觉质量，但也会增加烘焙时间。默认值是2。

其他参数可以在下方查看详细解释。

【选读】Progressive Lightmapper参数详解
Prioritize View 优先计算Scene窗口中可见物体的光照，再计算窗口外不可见物体

Direct Samples 直接光采样的数量。提高该值可以提高光照贴图的质量，但是会增加烘焙时间。

Indirect Samples 间接光采样的数量。对于室外场景，一般100就足够了。对于室内复杂场景，可以增加这个值直到达到你预期的效果。提高该值可以提高光照贴图的质量，但是会增加烘焙时间。

Bounces 间接光反弹的次数。对于大多数场景，默认值2次就足够了。但是对于复杂室内场景，可能需要更多的次数。

Filtering 在烘焙完成后，控制间接光噪点的设置。可以设置None，Auto或Advanced。一般保持默认值Auto即可。如果以后你需要详细调节，可以阅读官方文档中该部分的内容。

Advanced中的选项：

Gaussian 阴影更平滑

A-Trous 阴影边缘更锐利

Lightmap Resolution 设置世界中每单位对应光照贴图中多少个像素。提高该值可以提高光照贴图的质量，但也会增加烘焙时间。需要注意的是，提高该值，使用的总像素数量会呈2次方增加。比如原来是30，则1平单位使用3030=900个像素；提高到40后，则1平方单位使用4040=1600个像素。

Lightmap Padding 不同物体的光照贴图块在光照贴图中的间隙（避免块之间发生像素交叉的现象），默认值是2。

Lightmap Size 设置单张光照贴图的最大尺寸。默认值是1024。

Compress Lightmaps 是否压缩光照贴图。默认选中。选中时可以压缩贴图，减少存储空间占用，但是可能造成光照贴图出现不正常。

Ambient Occlusion 是否开启AO，默认开启。只影响间接光。

Max Distance 物体之间会产生AO的最大距离。物体之间的距离超出最大距离则不会产生AO。0代表不限制最大距离。

Indirect Contribution 间接光贡献度。提高该值可以增加参与AO的间接光的亮度，减少该值可以降低简介光的亮度。

Direct Contribution 直接光贡献度。提高该值可以增加参与AO的直接光的亮度，减少该值可以降低简介光的亮度。

前面我们提高过AO（Ambient Occlision，环境光遮蔽）。AO模拟环境光（不是来自特定方向的照明）能够照亮物体的程度。它会使弯折的物体、小孔、距离很近的表面变暗。因为这些区域遮挡了环境光线，因此它们看起来较暗。

以下4个选项同时适用于预计算实时GI和烘焙GI

Directional Mode 设置光照贴图存储入射光的哪些信息。

Directional 此模式下，Unity会生成次光照贴图来存储主要入射光的方向，用于材质的法线贴图。比如需要10张主光照贴图，使用此模式时，会相应生成10张次光照贴图。相对于Non-directional模式，此模式会占用2倍的存储空间。

Non-directional 此模式只会生成主光照贴图。

Indirect Intensity 控制实时或烘焙光照贴图中间接光的亮度。大于1的值可以提高亮度，小于1的值可以降低亮度。范围是0-5，默认值是1。

Albedo Boost 控制物体表面反射的光线的数量，范围是1-10。增加这个值可以让间接光更亮。默认值是1，模拟了真实世界中光照。

Lightmap Parameters 设置光照贴图的参数，我们已经在28课的选读部分进行了讲解，现在你也可以回去重新学习一遍。

【选读】Enlighten参数详解
Indirect Resolution 间接光计算中，每个单位使用几个像素表示。增加该值可以提高间接光的视觉质量，但也会增加烘焙时间。默认值是2。

Final Gather 如果你想要GI计算时最终的光线反弹和烘焙贴图有相同的分辨率，选中这个选项。这可以增加视觉效果，但是会增加额外的烘焙时间。

Ray Count 每个Final Gather（最终聚集）点发射的射线数量。默认是256。

Denoising 是否开启反噪点。默认开启。

以下参数和Progressive Lightmapper中的作用相同，请在上面查阅。
Lightmap Resolution
Lightmap Padding
Lightmap Size
Compress Lightmaps
Ambient Occlusion
Directional Mode
Indirect Intensity
Albedo Boost
Lightmap Parameters

5 开始烘焙
即使不修改第4步周的任何参数，所有参数保持默认值，也可以开始烘焙。你可以先烘焙一次看看效果，再去修改参数进行尝试。

使用Progressive Lightmapper烘焙时，右下角会显示预计剩余时间。

烘焙的时间视场景大小、参数设置的不同时间可能从几分钟到几十小时不等。通常一个正常的场景烘焙几个小时是正常的，会比预计算实时GI的时间长很多。后面我们会有专门的一课讲解场景及参数的优化。

烘焙完成后，你就能看到最终场景的效果。

总结
今天学习了使用Unity中的光照系统中的烘焙GI，希望你能记住使用的流程：
1、场景环境设置
2、对场景打光
3、设置静态物体的Lightmap Static标签和Renderer参数
4、设置烘焙的参数
5、烘焙，根据烘焙结果调整灯光、烘焙的参数重新烘焙直到达到满意的效果

今日思考题
构建一个简单场景，试试使用烘焙GI和预计算实时GI的区别~