---
layout: title
title: Player Settings
date: 2019-03-25 00:02:47
tags: Unity
---
Player Settings
Use the Player settings to set various options for the final game built by Unity.
使用player setting为Unity最后构建游戏设置各种选项

<!--more-->

There are a few general settings that are the same regardless of the build target. These are covered below.
无论构建目标如何，都有一些相同的常规设置。

Most settings, however, are platform-specific and divided into the following sections:
但是，大多数设置都是特定于平台的，并分为以下几个部分：

* Icon: the game icon(s) as shown on the desktop. 桌面上显示的游戏图标

* Resolution and Presentation: settings for screen resolution and other presentation details such as whether the game should default to fullscreen mode. 屏幕分辨率和其他显示详细信息的设置，例如游戏是否应默认为全屏模式。

* Splash Image: the image shown while the game is launching. This section also includes common settings for creating a Splash Screen, which are documented in the Splash Screen section.
游戏启动时显示的图像。此部分还包括用于创建启动画面的常用设置，这些设置在“启动画面”部分中进行了介绍。

* Other Settings: any remaining settings specific to the platform.

* Publishing Settings: details of how the built application is prepared for delivery from the app store or host webpage.

* XR Settings: settings specific to Virtual Reality, Augmented Reality, and Mixed Reality applications. 特定于虚拟现实，增强现实和混合现实应用程序的设置。

You can find information about the settings specific to individual platforms in the platform’s own manual section. Not all sections are supported on every platform. 
您可以在平台自己的手册部分中找到有关各个平台特定设置的信息。 并非每个平台都支持所有部分。

This table provides a breakdown of which platform support these sections:
此表提供了哪些平台支持这些部分的细分：

{% asset_img 1.png %}

# General Settings

{% asset_img 2.png %}

Player settings that are available for all platforms

| Property  | Function  |
| :------------ | :------------ |
| Company Name  | Enter the name of your company. This is used to locate the preferences file. 这用于找到配置文件 |
| Product Name  | Enter the name that appears on the menu bar when your game is running. Unity also uses this to locate the preferences file. 输入游戏运行时菜单栏上显示的名称 | 
| Version  | Enter the version number of your application.  |
| Default Icon  | Pick the the Texture 2D file that you want to use as a default icon for the application on every platform. You can override this for specific platforms. |
| Default Cursor（光标）  | Pick the the Texture 2D file that you want to use as a default cursor for the application on every supported platform.  |
| Cursor Hotspot（热点）  | Set the offset value (in pixels) from the top left of the default cursor to the location of the cursor hotspot. 将偏移值（以像素为单位）从默认光标的左上角设置为光标热点的位置 |

# Player settings for the Android platform

{% asset_img 3.png %}

automatic 自动
## Icon

{% asset_img 4.png %}

Property    Function
Adaptive    Set up textures for the Android Adaptive icons in your app.
Round   Set up textures for the Android Round icons in your app.
Legacy  Set up textures for the Android Legacy icons in your app.
Enable Android Banner   Enables a custom banner for Android TV builds.


## Resolution（分辨率） and Presentation
Use the Resolution and Presentation section to customize aspects of the screen’s appearance.
（自定义屏幕外观的各个方面）

{% asset_img 5.png %}

splash screen 启动画面

| Setting  | Function  |
| :------------ | :------------ |
| Start in fullscreen mode  | Hide the navigation bar while the splash screen or first scene
 loads. When not set, the navigation bar appears while the splash screen or first scene loads. 在启动画面或第一个场景加载时隐藏导航栏。未设置时，导航栏会在启动画面或第一个场景加载时出现。 |
| Preserve framebuffer alpha  | Enable this option if you want Unity to render on top of native the Android UI. The camera’s Clear Flags have to be set to Solid color with an alpha less than 1 for this to have any effect. (OpenGL ES only).  |
