---
layout: title
title: UGUI系统
date: 2019-02-13 21:05:32
categories: Unity
tags: Unity游戏开发技术详解与典型案例
---
旧版GUI使用时有很多不便，而且没有可视性。GUI代码需要在OnGUI函数中调用才能绘制，GUI的控件一般都需要传入Rect参数来指定屏幕绘制区域，例如Rect(0, 10, 200, 300)，对应的屏幕矩形区域左上角的坐标为（0，10），宽度为200，高度为300。在Unity GUI中，屏幕坐标系以左上角为原点。旧版的核心就是OnGUI函数，都需要在里面绘制，现在已经很少使用了。

<!-- more -->

# UGUI

## 创建UGUI控件

开始介绍UGUI系统前，首先要了解如何去创建一个UGUI控件。选择GameObject->UI就会出现所有的UGUI控件。选中一个想要的控件，如Button控件，单击它就可以完成创建。

创建好一个Button控件后就会在Hierarchy面板中看到结构。其中，Canvas是画布，在该界面中创建的所有控件都会自动变为Canvas游戏对象的子对象。若是场景中没有Canvas对象，在创建控件时Canvas对象会被自动创建。

创建第一个UGUI控件时，若场景中没有已经存在的Canvas游戏对象，就会自动创建一个，并将UGUI控件设置为Canvas的子对象，同时会自动创建一个名为EventSystem的游戏对象，上面挂载了若干可供设置的事件监听的相关组件。

## Canvas 画布

Canvas是一个游戏对象，自带一个Canvas游戏组件，所有的UI元素都必须是Canvas的子对象。若场景中没有一个Canvas，那么当创建一个新的UI元素时会自动生成一个Canvas游戏对象。

### UI元素的绘制顺序

UI元素在Canvas里的绘制顺序和它们在Hierarchy面板中的排序是一致的，即第一个子对象最先绘制，然后是第二个子对象，以此类推。如果两个UI元素有重叠部分，那么之后绘制的元素会挡在先绘制的元素上面。可以参考下图来帮助理解。

{% asset_img 1-1.png %}

{% asset_img 1-2.png %}

在图中，A、B两个对象是两个Image控件，用于显示图片。当A在B上方时，A被后渲染的B挡住；当B在A上方时，A把先渲染的B挡住。这样在Hierarchy面板中简单的拖曳就可以改变出现在最上层的UI元素或控件。

### Render Modes（渲染模式）

在Canvas中还可以通过设置渲染模式来确定UI元素在Screen Space上还是World Space上渲染。在Unity 3D中支持的渲染模式有3种：Screen Space-Overlay、Screen Space-Camera和World Space。

* Screen Space-Overlay
该渲染模式是默认的渲染模式。在该模式下会将所有的UI元素都渲染在场景中的最上层（类似于计算机屏幕上的贴膜，所有的UI元素都在这层贴膜上）。如果屏幕尺寸或者分辨率发生变化，Canvas也会自动去和变化后的尺寸相适应。

* Screen Space-Camera
该渲染模式和Screen Space-Overlay类似。在该模式下，Canvas游戏对象放置在一个预先设置好的摄像机的特定距离外，UI元素通过该摄像机进行渲染。所以使用该模式时应该创建一个摄像机并将其指定给Canvas组件下的Render Camera。改变该摄像机的设置，UI元素的显示效果也会跟着改变。

* World Space
该渲染方式使得Canvas更像一个游戏对象，可以手动改变其Rect Transform组件，从而更改其大小与旋转。在渲染时UI元素会根据它们在3D场景中的位置被渲染在其他游戏对象之前或之后，使其成为游戏视图中的一个成分。在做动态效果较多的界面时使用该模式比较方便。效果如下图所示。

{% asset_img 2.png %}

在Screen Space的两种渲染模式下，UI独立于游戏场景，不会被场景中的其他对象遮挡，始终保持在最上层；而在World Space渲染模式下，UI元素会被场景中的3D游戏物体遮挡，并且Canvas可旋转缩放等，适合制作一些非常酷炫的UI效果。

### Graphic Raycaster

每个Canvas都有一个Graphic Raycaster组件，用于获取用户选中的UGUI控件。多个Canvas之间的事件响应顺序由其显示顺序决定，在Hierarchy面板中越靠上的Canvas越后响应。当Canvas使用World Space或Camera Space渲染模式时，Graphic Raycaster的Block选项可以用来设置遮挡目标。

{% asset_img 4.png %}

## EventSystem组件

创建一个UGUI元素后，Unity会创建一个游戏对象，其名为EventSystem，上面挂载了一系列用于控制各类事件的组件，如下图所示。其自带的Input Module组件用于响应标准输入。在Input Module中封装了对Input模块的调用，用于根据用户操作触发对应的Event Trigger事件。

{% asset_img 3.png %}

EventSystem组件统一管理Input Module和各种Raycaster。该组件每帧调用多个Input Module处理用户的操作，同时还调用多个Raycaster用于获取用户单击到的UGUI控件或2D、3D物体。

EventSystem是Unity中的事件管理系统，对于UGUI中控件的单击监听等方法的实现将在下面进行介绍，在此只需要了解到EventSystem是一种将基于输入的事件发送到应用程序的对象，包括键盘、鼠标或自定义输入。

## Rect Transform组件

UGUI中每个控件都包括Canvas，Canvas会带一个Rect Transform组件。该组件继承自Transform，用于控制UI元素的Transform信息。当向一个Empty Object（空对象）添加UI Component组件时，Transform组件会自动变为Rect Transform。其参数介绍如下表所示。

{% asset_img 5.png %}

| 参数     | 含义  |
| :------   | :-------- |
|  PosX、PosY、PosZ  |   UI元素的位置   |
|    Width、Height    |  UI元素的长度和高度  | 
|   Anchors    |  相对于父对象的锚点   |
|    Pivot    |   UI元素的中心  | 
|    Rotation    |   按轴旋转  | 
|    Scale    |   按轴缩放  | 

单击Rect Transform组件中左上角的准星图标，可以在打开的Anchor Presets面板中进行快速设置。按住Shift键能同时设置Pivot，这时控件虽然不动，但Position已经改变。如果按住Alt键，则在设置Anchor的同时设置Position。如果同时按住Shift键和Alt键，那么就能同时设置Anchor、Pivot和Position。

{% asset_img 6.png %}

## Panel控件

执行GameObject->UI->Panel，就会在Canvas游戏对象下创建一个Panel控件。该控件是一个覆盖屏幕的平面，一般可以用来显示UI的背景，如下图所示。在其Image控件中，Source Image用于放置需要显示的Sprite，Color属性可以更改其颜色及透明度。读者也可以自行调配材质，然后拖曳到Material属性中。

{% asset_img 7.png %}

Panel控件在默认情况下会自动根据屏幕的大小来调整自身的大小，所以不用担心其屏幕自适应问题。
    
## Button控件

按钮是每个界面的重要组成元素之一。在UGUI中执行GameObject->UI->Button，就可以创建一个按钮控件。创建出来的Button控件中包含一个Text子对象，控制Button上显示的字样，若不需要按钮上显示字样，也可以将该子对象删除。

### 组件介绍

{% asset_img 8.png %}

接下来介绍Button控件挂载的组件。每个按钮都挂载有Button组件和Image组件，其中Image组件用于管理按钮的显示图片，Button组件用于管理按钮被单击后的变化以及监听。具体内容如下：

* Image组件
Button控件上的Image组件和之前介绍的Panel控件上的Image组件没有任何区别，在Source Image中可以放上合适的Sprite图片精灵，Color和Material可以设置图片的颜色和材质。

* Button组件
按钮上挂载的Button组件实现了按钮的全部功能，包括单击后的特效、单击的事件监听方法挂载。Button组件中各个参数的含义如下表所示。

| 参数     | 含义  |
| :------   | :-------- |
|  Interactable  |   该按钮是否启用   |
|   Transition    |  按钮状态变化模式   |
|    Navigation    |  导航，使用键盘方向键切换选中按钮时的切换顺序  | 
|    Visualize    |   可视化，使Navigation顺序在Scene面板中可视化  | 

Button组件中的Transition过渡选项定义了4种过渡模式，分别为None、Color Tint、Sprite Swap和Animation。除了None模式，其他过渡模式中每个按钮都有4种状态：Normal (正常状态)、Highlight (突出显示)、Pressed (按下状态）和Disable（禁用），可以对每个状态的按钮过渡进行自定义，具体区别如下。

(1) Color Tint。当使用该模式时，可以分别通过Color属性对按钮的4种状态进行设置，在对应的状态下时按钮的颜色就会变成设置的颜色，与正常状态产生区别。

{% asset_img 9.png %}

(2) Sprite Swap。该过渡模式为精灵换图，同样地，其按钮有4种状态可以设置，用户可以为每种状态下的按钮设置一个图片Sprite，设置完毕后，当按钮处于对应状态下时就会显示出对应的图片。注意，在各种状态设置图片时，图片也应当是一个Sprite，设置步骤可以参考Button控件Image组件部分的说明。

{% asset_img 10.png %}

(3) Animation。该过渡模式是UGUI的特色，该功能可以使UGUI系统和Unity中的动画系统完美地结合，使用动画状态机可以对不同状态下的按钮的位置、大小、旋转、图片等参数进行设置，功能非常全面。接下来将介绍一个使用Animation过渡模式的例子。

①选择GameObject->UI->Button，创建一个按钮，将其Button组件中的Transition选为Animation。单击Auto Generate Animation按钮，在弹出的对话框中找到合适的目录创建一个动画控制器。如下图所示。

{% asset_img 11.png %}

{% asset_img 15.png %}

②创建好动画控制器后，按步骤Window->Animation，打开动画编辑器窗口，单击Hierarchy面板中的上一步创建的Button组件，在Animation面板中单击左上角的下拉列表就可以选择想要编辑的按钮状态。如下图所示。

{% asset_img 12.png %}

③想要达到的效果是单击按钮后按钮进行弹性缩放，所以将当前编辑的按钮状态设置为Pressed，然后单击Add Property按钮。展开Rect Transform，单击Scale右边的“+”按钮，操作如下图所示。

{% asset_img 13.png %}

④单击Animation面板中的Curves按钮，进入曲线编辑模式，在该模式下可以对按钮Scale中X、Y、Z这3个参数进行设置。本示例的动画曲线如下图所示。

{% asset_img 14.png %}

⑤到此动画设置结束，关闭Animation面板，运行场景。当玩家单击按钮时，按钮会进行弹性缩放。

### 按钮点击监听挂载

本部分将介绍如何给创建好的按钮挂载单击监听。当然，为按钮挂载单击监听的方法有很多，这里介绍的是通过Button组件中的On Click()事件参数添加按钮单击监听，具体步骤如下。

(1)首先创建UGUIOnClick.cs脚本，然后挂载到Canvas游戏对象上。声明一个返回类型为空的方法，里面加上输出信息即可。
```cs
using UnityEngine;
using System.Collections;

public class UGUIOnClick: MonoBehaviour
{
    public void Onbt1Click() // 监听方法
    { 
        Debug.Log("This is bt1"); // 输出
    }
}
```
Onbt1Click方法就是场景中的按钮单击事件监听方法。在Unity中将该方法添加到按钮的单击事件列表中后，单击按钮就会自动回调该方法。

(2)单击Button组件On Click()下方的“+”按钮，为监听列表添加一个事件。将挂载有UGUIOnClick.cs脚本的游戏对象Canvas拖到选框中，展开有No Function字样的下拉列表，选择UGUIOnClick.Onbt1Click即可。如下图所示。

{% asset_img 1.gif %}

(3)运行场景，单击按钮后就会在Console面板中看到输出的This is bt1字样。在指定监听方法时还可以传递参数，修改上述代码，为Onbt1Click方法添加一个int类型的参数，然后保存，具体代码如下。
```cs
using UnityEngine;
using System.Collections;

public class UGUIOnclick : MonoBehaviour
{
    public void Onbt1Click(int index) // 声明方法
    {
        Debug.Log("This is bt" + index); // 输出信息
    }
}
```
在按钮的事件列表中为Onbt1Click设置参数，单击该按钮，就会调用该方法，并且传入设置好的参数。在示例中传入的参数类型是int型，这样在方法中就可以收到相应的参数了。

(4)重新在Button组件的OnClick()列表中指定方法后，下方会多出一个文本框。在其中输入对应的int类型参数即可，如下图所示。设置完毕后运行游戏场景，单击按钮就可以看到输出信息This is bt5。

{% asset_img 16.png %}

## Text控件

Unity的控件中有一个名为Text的控件，该控件的主要功能是在对应的区域内显示相应的文本。虽然在游戏中大部分文本为了美观需要使用Image来代替，但是Text控件依旧可以在开发中省却很多步骤。该控件中包含的参数如下表所示。

| 参数  | 含义  |
| :------------ | :------------ |
| Best Fit  | 最佳匹配方式（字体大小会根据内容多少和Text控件大小自动更改）  |

接下来的代码用于更改Text控件中的显示内容及字体颜色，可以将该代码挂载到Canvas游戏对象上，并将新建的Text控件指定给该脚本中对应的变量。具体代码如下。
```cs
using UnityEngine;
using System.Collections;
using UnityEngine.UI;

public class UGUIText : MonoBehaviour
{
	public Text tt;

	void Start()
	{
		tt.color = Color.red; // 设置Text的颜色
		tt.text = "this is text"; // 设置显示文本
	}
}
```
在Unity中支持导入外带的字体包，TTF格式的字体一般都可以使用。具体导入方法是将下载好的TTF文件放在项目目录下的Assets/Font目录下（没有请自己创建），在字体的Font参数中就可以找到导入的字体了。

## Image控件
Image控件即图片控件，该控件用于显示一个非交互式的图片精灵Sprite。作为游戏中最常用的控件之一，Image控件可以用于装饰界面、图标等。

在Image组件中，Source Image用于显示Sprite，所以当需要使用自己的图片时可以将其设置为Sprite格式。具体步骤为单击图片，在Inspector面板中将Texture Type选为Sprite(2D and UI)，单击Apply按钮即可。

## Raw Image控件

Raw Image控件用于显示一个非交互式的图像，这点与Image控件非常类似，区别在于Image控件只能显示Sprite（图片精灵），而Raw Image控件可以显示任何纹理。

| 参数  | 含义  |
| :------------ | :------------ |
| Raycast Target  | 是否接受射线事件  |
| UV Rect  | 图片在控件矩形中显示的偏移和大小  |

由于Raw Image控件不需要精灵纹理Sprite，因此它可以用于显示在游戏中使用WWW类从某个URL下载的图像或渲染纹理，也可以使用场景中某个特定摄像机的渲染图在UI中呈现出该摄像机拍摄到的画面。下面将介绍如何使用Raw Image控件呈现出场景中的摄像机Camera1所拍摄的画面。

(1)在Project面板中右击，执行Create->RenderTexture命令，创建一个渲染图片，并将其命名为Camera1RT。在场景中创建一个名为Camera1的摄像机，并将Camera组件中的Target Texture设置为Camera1RT。

{% asset_img 17.png %}
	
(2)在Raw Image控件上的RawImage组件中将Texture设置为Camera1RT。设置完毕后运行该游戏场景。这时Raw Image控件显示的就是Camera1拍摄到的画面。

{% asset_img 18.png %}

{% asset_img 19.png %}

## Slider控件

选择GameObject->UI->Slider，即可创建一个Slider控件（滑块控件），如图所示。其子对象结构如下图所示。该控件可由玩家滑动以操控其值的大小，可以用来制作游戏中的音量大小滑块等。

Slider控件的子对象中，Background是滑块主题背景，本身为一个Image控件；Fill Area下的子对象Fill代表已经被选中的部分，它会随着滑块的左右滑动而改变长度；Handle子对象是玩家单击的滑块按钮。Slider控件中的参数含义如下表所示。

{% asset_img 20.png %}

| 参数  | 含义  |
| :------------ | :------------ |
| Interactable  | 是否启用该控件  |
| Transition  | 过渡模式  |
| Navigation  | 导航，使用键盘方向键切换选中按钮时的切换顺序  |
| Visualize  | 可视化，使Navigation顺序在Scene面板中可视化  |
| FillRect  | Fill子对象的RectTransform组件的引用   |
| HandleRect  | Handle子对象的RectTransform组件的引用  |
| Direction  | 滑块的方向，默认是从左到右  |
| Min Value  | 滑块的最小值  |
| Max Value  | 滑块的最大值  |
| Whole Number  | 滑块的值是否只能是整数  |
| Value  | 滑块当前的值  |

在Slider控件最下方的On Value Changed(Single)中还可以为Slider控件绑定事件监听方法，该控件发出的事件是“值发生改变”，所以绑定的监听方法就会在滑块值发生变化时回调。具体设置步骤如下。

(1)创建一个脚本，命名为UGUISlider.cs，并将其挂载到Canvas游戏对象上，将其sd参数指定为Slider，脚本代码如下。
```cs
using UnityEngine;
using System.Collections;
using UnityEngine.UI;

public class UGUISlider : MonoBehaviour
{
	public Slider sd;
	public void OnsdValueChange() // 值发生改变后回调的方法
	{
		Debug.Log(sd.value); // 输出变化后的值
	}
}
```
该脚本较为简单，里面仅有一个OnsdValueChange方法，在Unity中将该脚本中的监听方法挂载给对应的滑块后，若滑块的值发生改变，系统就会自动回调OnsdValueChange方法。

{% asset_img 24.png %}

(2)单击On Value Changed(Single)下方的“+”按钮添加一个事件监听，将挂载有脚本UGUISlider.cs的游戏对象Canvas拖曳到Runtime下的GameObject选框中。在右边选择其监听方法为UGUISlider.OnsdValueChange，如下图所示。这时运行场景，若滑块的值变化，就会打印出变化后的滑块值。

{% asset_img 23.png %}

## Scrollbar控件

Scrollbar控件即滚动条控件，执行GameObject->UI->Scrollbar，即可以创建出一个Scrollbar控件，如下图所示。其子对象结构如图所示。Scrollbar控件和Slider控件功能相似，其具体参数如下表所示。

{% asset_img 25.png %}

{% asset_img 26.png %}

{% asset_img 27.png %}

| 参数  | 含义  |
| :------------ | :------------ |
| Interactable | 是否启用控件  |
| Transition  | 过渡模式  |
| Navigation  | 导航，使用键盘方向键切换选中按钮时的切换顺序  |
| Handle Rect  | Handle子对象的Rect Transform组件  |
| Direction  | Scrollbar的方向，默认从左到右  |
| Visualize  | 可视化，使Navigation顺序在Scene窗口中可视化  |
| Value  | Scrollbar的值  |
| Size  | 滑块的大小  |
| Number of Steps  | 进行分段，滚动条的显示分段  |

## Toggle控件

游戏的设置界面中经常能见到各种开关，在UGUI中开关控件Toggle就实现了开关的功能。执行GameObject->UI->Toggle，即可创建一个Toggle控件，其内部结构如下图所示。

{% asset_img 28.png %}

{% asset_img 29.png %}

Toggle控件的子对象中包含Background，它是一个Image控件，作为开关的背景；Checkmark也是一个Image控件，用于显示选中后的图案，如上图中的“√”图样；Lable是一个Text控件，可用来显示开关的信息，如上图中的Toggle字样。在游戏中若是用不到Lable可以将其删除。Toggle控件的参数及其含义如下表所示。

{% asset_img 30.png %}

| 参数 | 含义 |
| :------------ |:------------|
| Interactable  |   是否启用该控件 |
| Navigation    |   导航，确认控件的顺序 |
| Transition    |  过渡模式 |
| Visualize     |  使导航顺序在Scene窗口中可视化 |
| Togele Transition |  开关的消隐模式，有none和Fade（褪色消隐）两种模式|
| Group | 成组（将一组开关变成多选一开关）|
| Is On | 开关的状态（“开”或“关"）|
| Graphic | Checkmark（复选标记）子对象的引用 |

接下来将介绍如何使用开关的Group参数。

(1)在Hierarchy面板中选中Canvas游戏对象，右击，选择Create Empty，创建一个空对象，方便管理。依次创建3个Toggle控件，将其设置为GameObject（刚才创建的空对象）的子对象，如下图所示。在这里需要将创建的3个Toggle控件中的Is On设置为关闭状态。

{% asset_img 31.png %}

(2)选中上一步中创建的GameObject空对象，执行Component->UI->Toggle Group，添加一个Toggle Group组件，如下图所示。其中的Allow Switch Off参数决定是否可以取消选中打开的开关。

{% asset_img 32.png %}

(3)依次选中第（1）步创建的3个Toggle控件，将其Toggle控件中的Group参数选为挂载有Toggle Group组件的GameObject游戏对象。这样3个Toggle控件就成组了，最多只能同时选中其中的一个。

## Input Field控件

Input Field控件是UGUI中的文本框控件，用户在移动设备上单击到该控件时，就会弹出用于输入的键盘，常见于各个游戏中给游戏人物取名等地方。在文本框没有输入时，会显示默认的提示文本，其内部结构如下图所示。

{% asset_img 43.png %}

{% asset_img 44.png %}

在Input Field控件的子对象中，Placeholder用于显示默认提示信息的文本框，如下图中的Enter text字样；Text用于显示用户输入的文本，若想改变默认提示文本，直接改变Placeholder的Text属性即可。Input Field控件包含的参数如下表所示。

| 参数  | 含义  |
| :------------ | :------------ |
| Interactable  | 是否启用该控件  |
| Transition  | 过渡模式  |
| Navigation  | 导航  |
| Visualize  | 使导航顺序在Scene面板中可视化  |
| Text Component  | 用于用户输入的文本框的引用  |
| Text  | 用户输入文本框中的内容  |
| Character Limit  | 可以输入到文本框中的最多文字数  |
| Content Type  | 指定文本框的类型  |
| Line Type  | 换行方式，包括单行显示、自动换行、自定义换行  |
| Placeholder  | 提示文本框的引用  |
| Caret Blink Rate  | 光标的闪烁速度  |
| Caret Width  | 光标的宽度  |
| Custom Caret Color  | 光标的颜色  |
| Selection Color  | 选中文本框中文本时文本框的颜色  |
| Hide Mobile Input  | 在移动设备上输入时是否隐藏  |
| Read Only  | 是否只读  |

Input Field控件可以发出两个事件：OnValueChanged和OnEndEdit，分别在当值发生改变时发出和结束编辑时发出。用户可以自行单击事件下方的“+”按钮添加事件监听方法，两个事件监听的添加方法完全相同。

## DropDown控件

在游戏的设置界面中有时能见到各种下拉菜单，UGUI的DropDown控件就实现了下拉菜单的功能。选择GameObject->UI->DropDown，即可创建一个下拉DropDown控件，其内部结构如下图所示。

{% asset_img 45.png %}

{% asset_img 46.png %}

DropDown控件的子对象中，Label是一个Text控件，用于显示下拉菜单的信息；Arrow是一个Image控件，用于显示下拉箭头的图案，如上图所示的箭头图样；Template是一个下拉列表控件，用于显示下拉菜单的信息。DropDown控件包含的参数如下表所示。

{% asset_img 47.png %}

| 参数  | 含义  |
| :------------ | :------------ |
| Interactable  | 是否启用该控件  |
| Transition  | 过渡模式  |
| Navigation  | 导航，确认控件的顺序  |
| Visualize  | 使导航顺序在Scene面板中可视化  |
| Template  | 下拉列表模板的Rect Transform  |
| Caption Text  | 保存当前选定选项文本的文本组件  |
| Caption Image  | 用于保存当前选定选项图像的图像组件  |
| Item Text  | 用于保存项目文本的文本组件  |
| Item Image  | 用于保存项目文本的图像组件  |
| Value  | 当前选择选项的索引。0是第一个选项，1是第二个选项，依此类推  |
| Options  | 可以为每个选项指定文本字符串和图像  |

DropDown控件可以发出一个事件：OnValueChanged，具体意义是当值发生改变时发出此事件。可以自行单击事件下方的“+”按钮添加事件监听方法。

## Scroll View控件

Scroll View（滚动视图）控件在游戏中非常常见，比如游戏中的背包内容太多而无法一次显示完毕时就需要一个滚动视图，让玩家可以上下拖动以显示更多内容。选择GameObject->UI->Scroll View，即可创建一个Scroll View控件，其内部结构如下图所示。

{% asset_img 48.png %}

{% asset_img 49.png %}

用一个小示例介绍Scroll View控件，步骤如下。
(1)在Hierarchy面板中选中Content对象，选择Add Component->Layout->Grid Layout Group，为Content对象添加一个网格布局组件，参数如下图所示。在Content对象下创建12个Image控件，以此充当滚动视图的内容，如下图所示。

{% asset_img 50.png %}

{% asset_img 51.png %}

(2)修改Scroll Rect的参数，取消选中Horizontal复选框，表示滚动视图不支持横向滑动，如下图所示。为了使滚动视图纵向适配视图内容，需要在Content对象上添加一个内容尺寸适配器，参数设置如下图所示。

{% asset_img 52.png %}

(3)至此，滚动视图创建完毕。最后为Image赋予贴图后，运行场景，便可上下拖动滚动视图，运行效果如下图所示。

{% asset_img 53.png %}

## UGUI布局管理的使用及相关组件介绍

接下来讲解如何管理、布局多个控件。这部分知识的运用常见于游戏的奖励窗口，由于预先不知道获得奖励的数量，但是依旧需要让获得的奖励道具按照一定的布局整齐地出现在界面中，这时就需要用到布局的知识了。

Unity自带的布局组分为3种，分别为水平布局、垂直布局和网格布局，还有其他一些适配器、布局元素等组件。首先创建5个Image控件，并将其放在空对象UIMain下成为其子对象，如下图所示。

{% asset_img 33.png %}

** 1.Horizontal Layout Group（水平布局） **

选中UIMain空对象，选择Add Component->Layout->Horizontal Layout Group，即可为该游戏对象添加一个水平布局管理组件。顾名思义，在该组件的作用下，UIMain的子对象将按照一定的要求进行水平排列。该组件如下图所示，组件包含的参数如下表所示。

{% asset_img 34.png %}

| 参数 | 含义 |
| :------------ |:------------|
| Padding  |   布局的边缓填充（即偏移） |
| Spacing    |   布局内的元素间距 |
| Child Alignment    |  对齐方式 |
| Child Force Expand     |  自适应宽和高 |
| Control Child Size     |  是否控制子物体缩放 |

为UIMain添加Horizontal Layout Group组件后，它所有UI元素子对象都会根据对Horizontal Layout Group组件的设置进行水平自动排列。如下图所示。

{% asset_img 35.png %}

** 2.Vertical Layout Group（垂直布局） **

选中UIMain游戏对象，选择Add Component->Layout->Vertical Layout Group，即可给该游戏对象添加一个垂直布局管理组件。如下图所示。该组件的功能是将UI元素按照一定的规则进行整齐的垂直排列，其内部参数和Horizontal Layout Group的参数基本一样。

{% asset_img 36.png %}

{% asset_img 37.png %}

** 3.Grid Layout Group（网格布局） **

Grid Layout Group是网格布局管理器组件，该组件会将其管理下的UI元素进行自动的网格型的排列，如下图所示。此外，它还实现了自动换行等功能。该组件常见于各个游戏中的背包内部的储物格。Grid Layout Group组件包含的参数如下表所示。

{% asset_img 39.png %}

{% asset_img 38.png %}

|  参数名  | 含义  |
| :------------ | :------------ |
| Padding  | 偏移  |
| Cell Size  | 内部元素的大小  |
| Spacing  | 每个元素间的水平间距和垂直间距  |
| Start Corner  | 第一个元素的位置  |
| Start Axis  | 元素的主轴线  |
| Child Alignment  | 对齐方式  |
| Constraint  | 指定网格布局的行或列  |

以上即为Unity中自带的3种布局管理模式，大部分情况下可以满足开发的需要。在游戏运行时随时将新实例化的UI控件或者游戏对象设置为挂载有Layout Group组件（3种中任意一个皆可）的游戏对象的子对象，Layout Group组件便会对其进行自动布局排列。
```cs
using UnityEngine;
using System.Collections;

public class UGUILayout : MonoBehaviour 
{
	public GameObject UIMain;  // 挂载有Layout group组件的游戏对象
	public GameObject items; // 需要实例化的UI控件或者游戏对象的预制件

	void Start ()
	{
		GameObject item = (GameObject)Instantiate(items); // 实例化items
    	item.transform.parent = UIMain.transtorm; // 将实例化的游戏对象设置为UIMain的子对象	
	} 
}
```
在Start方法中新实例化出了一个items预制件，将其设置为挂载有布局管理器组件的UIMain的子对象，然后观察场景，就会发现新实例化的预制件已经被自动排列好了。这在游戏开发中非常方便，可以随时实例化UI元素而不用再三考虑排列布局问题。

** 4.Layout Element（布局元素） **

Layout Element是布局元素组件，该组件常用于管理带有布局组对象的子物体。选择Add Component->Layout->Layout Element，即可给该游戏对象添加一个布局元素组件，如下图所示。Layout Element组件包含的参数如下表所示。

{% asset_img 40.png %}

{% asset_img 41.png %}

{% asset_img 42.png %}

| 参数  | 含义  |
| :------------ | :------------ |
| Ignore Layout | 是否受布局组影响  |
| Min Width  | 布局元素的最小宽度  |
| Min Height  | 布局元素的最小高度  |
| Preferred Width  | 布局元素的最大宽度  |
| Preferred Height  | 布局元素的最小高度  |
| Flexible Width  | 宽度拉伸布局比例  |
| Flexible Height  | 高度拉伸布局比例  |


5.Content Size Fitter（内容尺寸适配器）

6.Aspect Ratio Fitter（宽高比适配器）

## UGUI中不规则形状的按钮的碰撞检测

UGUI中自带的按钮是标准的矩形，虽然可以由玩家任意换图，但是其碰撞检测区域始终是矩形的。在有些时候，可能会用到特殊形状的按钮，当然其碰撞检测区域也要是符合按钮形状的。下面就使用UGUI中的知识来创建一个不规则形状的按钮。具体步骤如下。

1.首先创建一个Button控件，命名为“bt1”，由于这里不需要它的Text子对象，所以可以将其删除。选中bt1后执行“Component->Physics2D->Polygon（多边形） Colloder（对撞机） 2D”命令，为bt1添加一个多边形碰撞器组件。

2.单击Polygon Colloder 2D组件中的Edit Collider按钮，在Scene界面中将想要的碰撞检测区域勾选出来，如图中五边形边上的亮线所示。勾选出来的五边形即为该按钮的碰撞检测区域。

3.接下来要实现将按钮的碰撞区域和Polygon Colloder 2D组件勾选区域的挂钩，这一步要重写Image类。新建一个C#脚本，将其命名为"UGUIImagePlus.cs"，该脚本需要引用UnityEngne.UI命名空间，并维承Image类，脚本代码如下。
```cs
using UnityEngine;
using System.Collections;
using UnityEngine.UI;

public class UGUIImagePlus : Image 
{
	PolygonCollider2D collider;  //多边形碰撞器组件

	void Awake()
	{
		collider = GetComponent<PolygonCollider2D>(); //获取2D多边形碰撞器组件
	}

	public override bool IsRaycastLocationValid(Vector2 screenPoint, Camera eventCanera) 
	{
		bool inside = collider.OverlapPoint(screenPoint); //判断触摸是否在围出的多边形区域内
		return inside;  //返问是否在多边形内
	}
}
```
本脚本继承自UnityEngine.UI.Image类，并重写了该类的IsRaycastLocationValid说明 方法。这个方法用于返回触摸点（screenPoint）是否在图片范围内。在该方法中使用Collider2D.OverlapPoint方法判断点是否在多边形区城内。

4.接下来将bt1上面挂载的Image组件移除，单击Inspector面板中的Image组件右边的设置按钮，选择Remove Component。将UGUIImagePlus.cs类拖拉到bt1上形成UGUIImagePlus组件以代替。

5.挂载好UGUIImagePlus组件后，重新为该组件指定好纹理图。这样，图片已经可以检测到由Polygon Colloder 2D组件勾选出来的区域内的触摸了。