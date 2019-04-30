---
layout: title
title: Input
date: 2019-03-27 16:22:38
categories: Unity
tags: 官方文档
---
class in UnityEngine/Implemented in:UnityEngine.CoreModule

<!--more-->

Description
Interface into the Input system.

Use this class to read the axes set up in the Conventional Game Input, and to access multi-touch/accelerometer data on mobile devices.

To read an axis use Input.GetAxis with one of the following default axes: "Horizontal" and "Vertical" are mapped to joystick, A, W, S, D and the arrow keys. "Mouse X" and "Mouse Y" are mapped to the mouse delta. "Fire1", "Fire2" "Fire3" are mapped to Ctrl, Alt, Cmd keys and three mouse or joystick buttons. New input axes can be added. See Input Manager for this.

If you are using input for any kind of movement behaviour use Input.GetAxis. It gives you smoothed and configurable input that can be mapped to keyboard, joystick or mouse. Use Input.GetButton for action like events only. Do not use it for movement. Input.GetAxis will make the script code more small and simple.

Note also that the Input flags are not reset until Update. It is suggested you make all the Input calls in the Update Loop.

See Also: KeyCode which lists all of the key press, mouse and joystick options.

Mobile Devices:

iOS and Android devices are capable of tracking multiple fingers touching the screen simultaneously.
iOS和Android设备能够跟踪多个手指同时触摸屏幕。 You can access data on the status of each finger touching screen during the last frame by accessing the Input.touches property array.
通过访问Input.touches属性数组，您可以访问最后一帧期间每个手指触摸屏状态的数据

As a device moves, its accelerometer hardware reports linear acceleration changes along the three primary axes in three-dimensional space. You can use this data to detect both the current orientation of the device (relative to the ground) and any immediate changes to that orientation.

当设备移动时，其加速计硬件报告沿三维空间中的三个主轴的线性加速度变化。 您可以使用此数据来检测设备的当前方向（相对于地面）以及对该方向的任何即时更改。