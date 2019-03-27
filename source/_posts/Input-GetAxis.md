---
layout: title
title: Input.GetAxis
date: 2019-03-27 17:16:35
categories: Unity
tags: Unity
---
public static float GetAxis(string axisName); static 类名.方法名
Description
Returns the value of the virtual axis identified by axisName.

The value will be in the range -1...1 for keyboard and joystick input. If the axis is setup to be delta mouse movement, the mouse delta is multiplied by the axis sensitivity and the range is not -1...1.

This is frame-rate independent; you do not need to be concerned about varying frame-rates when using this value.

To set up your input or view the options for axisName, go to Edit > Project Settings > Input. This brings up the Input Manager. Expand Axis to see the list of your current inputs. You can use one of these as the axisName. To rename the input or change the positive button etc., expand one of the options, and change the name in the Name field or Positive Button field. Also, change the Type to Joystick Axis. To add a new input, add 1 to the number in the Size field.