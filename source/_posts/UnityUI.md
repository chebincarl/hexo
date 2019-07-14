---
layout: title
title: UnityUI
date: 2019-03-28 23:48:44
categories: Unity
tags: Unity5权威讲解
---
本章将利用UGUI制作游戏中的主要UI界面，包括如何在游戏场景中实现显示得分、生命条、暂停等按钮的功能。

<!--more-->

首先创建名为scMain的新场景，导入资源包UI_Textures.unitypackage。导入后，将生成的UI Textures文件夹移动到Images。

# Canvas对象

Canvas是Unity提供的游戏对象之一，所有游戏界面的UI元素（纹理、图像、按钮、滑动条等）都必须位于Canvas之下，成为其子对象。Canvas拥有的是Rect Transform组件，而并非其他游戏对象拥有的Transform组件。另外，一个场景中可以有多个Canvas对象，也可以导入其他Canvas对象作为当前Canvas对象的子节点。

生成Canvas对象后，EventSystem对象会自动生成。<span style="color:red">EventSystem对象可将系统中发生的键盘、游戏杆、触摸屏等输入信息传递给Canvas包含的UI元素。</span>

## EventSystem对象

EventSystem对象包含EventSystem组件、Standalone Input Module组件等，其中EventSystem组件的First Selected属性可以设置哪个UI元素将会第一个获得焦点。EventSystem对象很重要，如果没有它，UI元素就无法响应各种输入事件并执行相应动作。

> EventSystem对象包含的处理输入事件相关的组件

{% asset_img 1.png %}

Canvas对象包含Rect Transform、Canvas、Canvas Scaler和Graphic Raycaster这4个组件。UI元素必须拥有Rect Transform组件，它保存着锚点（Anchors）、枢纽（Pivot）、大小（W、H）、位置（Pos X、Pos Y、Pos Z）、旋转以及比例等很多信息。简言之，Rect Transform组件可以理解为专用于UI界面的Transform组件。不可以直接修改Canvas对象的Rect Transform组件的属性，系统会根据画布大小自行设置。

Canvas对象的Rect Transform组件为禁用状态。

> Canvas对象的基本组件 

{% asset_img 2.png %}

## Canvas组件

Canvas组件可将游戏所需的各种UI元素放置到画面，并对其进行渲染。根据渲染模式的不同，UI元素的画面配置方式有如下几种。

** 1.Screen Space-Overlay **
Canvas组件默认设置值，UI元素置于画面最表层，可以根据画面分辨率的设置自动调节位置。

** 2.Screen Space-Camera **
此选项虽然与前一个选项类似，但可以设置渲染UI元素的摄像机。也就是说，可以为其指定一个与（直接朝向游戏全部场景的）主摄像机不同的专用摄像机。将渲染模式属性更改为此选项后，Inspector视图中就会出现可以指定其他摄像机的Render Camera属性。

{% asset_img 3.png %}

如果将渲染UI界面的摄像机的Projection属性设置为Perspective并旋转UI元素的Y轴，则可以使UI界面表现透视感。

<span style="color:red">为了避免因渲染UI元素而添加的摄像机与现有主摄像机冲突，必须适当设置其Clear Flag、Culling Mask、Depth等属性。</span>

** 3.World Space ** 
如果设置为World Space选项，则可以向场景内的其他游戏对象直接添加UI元素，其最具代表性的案例就是制作HUD（Head Up Display，平视显示器）。向特定游戏对象添加Canvas对象后，如果将渲染模式设置为World Space，则该Canvas对象不再受其Rect Transform组件设置的影响，而受相应游戏对象位置的影响。

# Rect Transform组件

UI元素默认包含Rect Transform组件，其作用与之前讲过的Transform组件相同，此处通过Panel元素了解Rect Transform组件的属性。Panel主要用于编组其他UI元素，例如，如果想将用户ID、用户Password以及加入会员等UI元素绑定为1个组时，可以使用Panel。也就是说，制作UI菜单时可以将各个按钮放入1个Panel。

{% asset_img 7.png %}

添加Panel后，Scene视图中会出现矩形面板。Panel对象默认具有Image组件，其4角均有蓝色圆形图标和三角形Gizmos。如果Scene视图当前为2D模式，那么在Rect工具栏中选择如下图所示按钮并使用鼠标即可将Panel移动到想要的位置；3D模式下，拖曳当前3D坐标轴即可移动Panel。

{% asset_img 4.png %}

将光标移动到Panel边框后，可拖曳调节其大小。

拖曳Panel的4角蓝色圆形图标，也可以向相应方向调节其大小。点击鼠标并按Alt键拖曳，可以同时调节Panel的4边大小。

{% asset_img 1.gif %}

Panel中间的圆形为其正中心，是布局、排列UI元素时参考的基准位置。另外，这个中心圆圈也起到旋转轴心的作用。将鼠标光标移动到Panel外侧就会出现旋转图标，此时按住鼠标拖曳即可旋转Panel。

{% asset_img 2.gif %}

其他UI元素的移动、大小调节、旋转等操作与上述方式一致。如下图所示，Panel周围有4个小的白色三角形，以Panel的父对象Canvas为基准，称为定位点（Anchor Point），其与Panel的布局排列和大小调节均相关。定位点通常以Panel的父节点的Rect Transform组件属性值为基准，所以请牢记，选择某个UI元素时，其定位点不是以自身Rect Transform为基准，而是以其父对象的Rect Transform为基准。

{% asset_img 8.png %}

** Anchor Preset **
Anchor Preset可将各UI元素的排列与大小等设置定义为预设，这是UGUI的基本概念，必须熟记。

{% asset_img 5.png %}

现在，Panel的锚点预设功能支持Stretch模式。该模式下，即使屏幕分辨率发生变化，Unity也会自动调节大小以适应新的分辨率。

> 横竖均可自由伸缩的Anchor Preset

{% asset_img 6.png %}

Stretch模式下，如果将游戏视图从Unity IDE中分离并调整视图大小，可以发现Panel的大小也会随之变化。

{% asset_img 3.gif %}

点击Anchor Preset按钮，出现事先定义的Anchor Preset。

> 默认Anchor Preset（只点击鼠标时）

{% asset_img 9.png %}

> 按Alt键时

{% asset_img 10.png %}

> 按Shift键时

{% asset_img 11.png %}

> Alt + Shift

{% asset_img 12.png %}

锚点（定位点）由4个三角形组成，每个都可以分别移动。上图为这4种设置界面下Panel在Canvas对象内排列方式的缩略示意图。

** 1.默认锚点预设 **
点击鼠标可更改锚点位置

** 2.点击鼠标并按Alt键进行设置 **
打开Anchor Preset界面并按Alt键，此模式可将当前选定的UI元素移动到Anchor Preset中选择的预设位置并进行排列。

** 3.点击鼠标并按Shift键进行设置 **
按Shift键并点击Anchor Preset之后出现设置界面，此处只会更改锚点和所选UI元素的中心点位置，不会移动UI元素位置。

** 4.点击鼠标后同时按Alt + Shift键进行设置 **
将UI元素中心点移动到与锚点相同的位置。

# anchoredPosition属性

RectTransform属性顶端的Pos X、Pos Y、Pos Z以相应UI元素锚点为基准，显示UI元素当前位置，在Unity中称为anchoredPosition。下图的Panel的位置表示，以Anchor Point为基准时，X轴方向-130、Y轴方向+60的位置。

> 设置锚点为middle、right时的anchoredPosition属性值

{% asset_img 13.png %}

anchoredPosition的属性为Vector2类型，不涉及Z轴，所以Pos Z值为0。Width和Height属性则是相应UI元素的宽和高的值。Pos X、Pos Y、Width、Height这些属性值表示当前UI元素的中心点相对锚点位置的偏移量。

下图是将锚点设置为左边排列与宽高自适应后的界面截图。因为选择了宽高自适应，所以UI元素的高度可以根据场景当前分辨率改变，故Inspector视图中的Pos Y值和Height值分别变为Top值和Bottom值，二者分别表示当前UI元素的顶端留空值（Top Margin）与下端留空值（Bottom Margin）。

** Anchor属性 **

Unity UI用4个小箭头表示Anchor属性，通过它可设置并调整UI元素的大小和对齐方式。RectTransform组件的Anchor属性中的Min(X, Y)、Max(X, Y)值是锚点位置，取值范围均为0.0f ~ 1.0f，例如，0.5f表示50%。

UI元素四周均设置锚点后，该UI元素就只能在锚点设置的范围内调整大小。下图中，锚点的Min值设置为(0.2, 0.2)，Max值设置为(0.8, 0.8)，这样可以使Panel大小随画面分辨率的变化而变化。

上面设置的锚点值表示从画面左边20%、画面下端20%到画面左边80%、画面下端80%的范围。也就是说，画面按照比例分割为20%，60%，20%，如下图所示。

# Image组件

Panel对象默认带有Image组件，Image组件可为画面附上纹理，在Unity中必须将纹理先转换为Sprite格式后才能使用。现在将Panel的Image组件的Source Image属性设为之前下载的SF Window文件。

设置纹理后，如下图所示，设置Panel的位置和锚点预设。

Image组件是制作游戏UI界面时最常用的UI元素，仔细观察其属性，尤其要注意Image Type选项不同导致的图像变化。

| 属性  | 功能  |
| :------------ | :------------ |
| Source Image  | Panel要使用的图像（仅允许Sprite格式的文件）  |
| Color  | 指定图像的颜色（RGBA）  |
| Material  | 渲染图像时使用的材质（使用法线贴图时可用）  |
| Image Type  | 显示图像的方式有4种。<br>Simple：不需要重复显示图像或需要使图像长宽比例固定；<br>Sliced：调整图像大小也不会使其周围图像变形；<br>Tiled：可以平铺图像；<br>Filled：可以只显示部分图像。  |

下面讲解Image组件的Image Type属性提供的4种选项的不同之处。

## Simple

用于固定图像长宽比例，主要适用于装饰画面的图片。选择Simple时，Inspector视图中会出现Preserve Aspet选项，勾选后，图像即可按照其原有长宽比例进行调整。通过Set Native Size按钮可设置原版图像大小。

按住Shift键后，可按照既有长宽比例对所有UI元素进行等比调整。

## Sliced


将Panel中的Image组件的Image Type设置为Sliced后，即使调整图像大小，其边框外围部分的图像也不会变形，只有中间切片的图像才会随着调整的大小而缩放。要想使用Sliced选项设置图像，需要在Sprite Editor中事先设置要使用的图像的九宫格线。选择项目视图的Images/UI Textures/Textures and Sprites/SF Window，点击Inspector视图的Sprite Editor按钮打开Sprite Editor视图，如下图所示。SF Window图像是背景透明的白色图像，所以为了便于查看，点击控制栏的RGB/Alpha按钮选择Alpha。

调整4条绿色实线的位置，即可设置九宫格大小。

按照边线将图像切分为9块后，即使调整整个图像的大小，4条边附近的图像也不会变形，如下图所示。设置为Sliced类型的图像常用于制作登录对话框等窗口。

选择Sliced选项后，Inspector视图会出现Fill Center属性。如果勾选，则显示原图像切片的九宫格中间图像；如果不勾选，则最终图像只显示中间轮廓。如下图所示，原始图像切片后的中间9号区域是透明的，没有图像。

## Tiled

Tiled选项可以使设置的图像不断重复，平铺整个画面，不受Image大小影响，如下图所示。

## Filed

选择Filed选项可以使图像沿着特定方向逐渐显示，直至填满整个Panel。在Fill Method属性中设置填充方式，Fill Origin属性可以选择从哪个位置开始填充。

> Fill Method属性

| 选项  | 说明  | Clockwise属性  |
| :------------ | :------------ | :------------ |
| Horizontal  | 横向填满图像  | 无  |
| Vertical  | 纵向填满图像  | 无  |
| Radial90  | 图像以90°填满  | 有  |
| Radial180  | 图像以180°填满  | 有  |
| Radial360  | 图像以360°填满  | 有  |

选择Radial90、Radial180或者Radial360选项时，可以继续设置Clockwise属性，Clockwise决定图像以顺时针方向还是逆时针方向进行填充。另外，设置填充比例的选项为Fill Amount，其取值范围是0.0f ~ 1.0f，可以看出，Filed选项在游戏开发中可用于表现生命条或技能冷却时间。

# RawImage组件

RawImage组件与Image组件相似，但其主要用于设置UI界面的静态背景。可以在Raw Image组件的Texture属性中设置纹理文件或者Sprite文件。

选择Hierarchy视图的Canvas对象，点击鼠标右键后，在上下文菜单中选择RawImage，Scene视图中会生成RawImage对象。将Project视图Images/SkyBox Volume 2/中用于Skybox的一个纹理设置到RawImage组件的Textuue属性，然后按住Shift键，利用鼠标拖曳等比调整RawImage，使其比Scene视图的Canvas区域更大，如下图所示。

UV Rect选项的X、Y属性可以设置图像位置的偏移量，而W，H属性设置其宽和高相对于原始大小的缩放比例。

如下图所示，之前添加的Panel的图像会被RawImage覆盖，所以需要调整。Unity UI系统各元素的Z-Order值由其各自在Hierarchy视图中的顺序决定，故将Hierarchy视图的排列选项更改为TransformSort，并将Panel对象拖曳至RawImage下方。这样，Panel的Z-Order值将比后者更高，从而使Panel能够在RawImage之上正常显示。

