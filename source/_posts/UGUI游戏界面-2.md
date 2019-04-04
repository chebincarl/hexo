---
layout: title
title: UGUI游戏界面-2
date: 2019-04-04 15:32:41
categories: Unity
tags: Unity3D游戏开发第2版-宣雨松
---
事件系统

<!--more-->

UGUI所有的事件系统都是依赖EventSystem组件来完成的，Unity新版的事件系统已经全面代替了之前的SendMessage系统了。操作的事件是非常庞大的。鼠标、键盘和手势能产生太多事件了，而且不同的UI操作事件还不太一样。就拿按钮来说，点击事件、按下事件和抬起事件都各不相同。可想而知，所有的UI系统加在一起能产生的事件将有多少。需要在全局添加一个EventSystem对象，运行起来后，在面板中还可以查看操作的一些详细信息，便于日后调试。

事件系统不仅会抛出点击一类的事件，还可以取到最基本的操作信息，例如鼠标在屏幕中的坐标、滑动开始的坐标以及滑动结束的坐标等。新版的EventSystem不仅供UI使用，3D游戏对象也可以使用它，并且使用方法都比较接近。

## UI事件

UI事件依赖于Graphic（图解的） Raycaster组件，如下图所示，它必须绑定在Canvas组件上，表示这个Canvas下所有UI元素支持的事件。比如游戏中同时有很多Canvas，如果想让游戏中某些UI不可接收点击事件，那么可以考虑把部分Canvas上的Graphic Raycaster组件设置成enable=false，或者直接删掉Graphic Raycaster组件即可。

{% asset_img 1.png %}

其实UGUI已经帮我们封装好了一些UI元素的事件，比如前面提到的Button、Toggle、Slider等，但像Image、Text这种特别基础的UI元素是没有事件封装的，如果非要监听的话，只能手动添加监听方法。首先，来看看UGUI有多少事件监听方法。
* IPointerEnterHandler - OnPointerEnter：进入该区域时调用。 
* IPointerExitHandler - OnPointerExit：离开该区域时调用。  
* IPointerDownHandler - OnPointerDown：按下时调用。  
* IPointerUpHandler - OnPointerUp：抬起时调用。 
* IPointerClickHandler - OnPointerClick：按下并且抬起时调用，好比按钮的点击。
* InitializePotentialDragHandler - OnInitializePotentialDrag：拖动初始化。  
* IBeginDragHandler - OnBeginDrag：拖动开始时调用，并且可以取到拖动的方向，而OnInitializePotentialDrag只表示滑动初始化，无法取到方向。 
* IDragHandler - OnDrag：滑动持续时调用。  
* IEndDragHandler - OnEndDrag：滑动结束时调用。  
* IDropHandler - OnDrop：落下时调用。  
* IScrollHandler - OnScroll：鼠标滚轮持续时调用。
* IUpdateselectedHandler - OnUpdateselected：选择时续调用，只针对Selectab起作用  
* ISelectHandler - OnSelect：选择后调用，只针对Selectable起作用。 
* IDeselectHandler - OnDeselect：取消选择，由于只能选择一个Selectable，当选择新的后，之前选择的就会回调取消选择事件。  
* IMoveHandler - OnMove：选择后，可监听上下左右WSAD方向键。如果访问eventData.moveDir，可以取到具体移动的方向。  
* ISubmitHandler - OnSubmit：按钮按下事件。  
* ICancelHandler - OnCancel：按钮取消事件，按下时按Esc键可取消。

## UI事件管理
## UnityAction和UnityEvent
## RaycastTarget优化
## 渗透UI事件