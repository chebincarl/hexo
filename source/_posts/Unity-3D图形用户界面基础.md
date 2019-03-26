---
layout: title
title: Unity 3D图形用户界面基础
date: 2019-02-13 21:05:32
tags: Unity
---

# 旧版GUI
使用时有很多不便，而且没有可视性。GUI代码需要在OnGUI函数中调用才能绘制，GUI的控件一般都需要传入Rect参数来指定屏幕绘制区域，例如Rect(0,10,200,300)，对应的屏幕矩形区域左上角的坐标为（0，10），宽度为200，高度为300，在Unity GUI中，屏幕坐标系以左上角为原点。旧版的核心就是OnGUI函数，都需要在里面绘制，现在已经很少使用了。

<!-- more -->
# UGUI

## 创建UGUI控件
开始介绍UGUI系统前，首先要了解如何去创建一个UGUI控件。按步骤在菜单栏中单击“GameObject->UI”就会出现所有的UGUI控件。选中一个想要的控件（如Button控件），单击它就可以完成创建。

创建好一个Button控件后就会在Hierarchy面板中看到结构。其中，Canvas是画布，在该界面中创建的所有控件都会自动变为Canvas游戏对象的子对象。若是场景中没有Canvas对象，在创建控件时该对象会被自动创建。

创建第一个UGUI控件时，若场景中没有已经存在的Canvas（画布）游戏对象，系统就会自动创建一个，并将UGUI控件设置为Canvas的子对象，同时会自动创建一个名为“EventSystem”的游戏对象，上面挂载了若干事件监听相关的组件可供设置。 

## Canvas 画布
Canvas是一个游戏对象，自带一个“Canvas”游戏组件，所有的UI元素都必须为一个Canvas的子对象。若场景中没有一个Canvas，那么当创建一个新的UI元素时会自动生成一个Canvas游戏对象。

### UI元素的绘制顺序
UI元素在Canvas里的绘制顺序和它们在Hierarchy面板中的排序是一致的，即第一个子对象最先绘制，然后是第二个子对象，以此类推。如果两个UI元素有重叠部分，那么之后绘制的元素会挡在先绘制的元素上面。可以参考下图来帮助理解。

{% asset_img 1.png %}

在图中，A、B两个对象是两个Image控件（用于显示图片）。当A在B上方时，A被后渲染的B挡住。当B在A上方时，A把先渲染的B挡住。这样在Hierarchy面板中简单的拖曳顺序就可以改变出现在最上层的UI元素或控件。

### Render Modes 渲染模式

在Canvas中还可以通过设置渲染模式来确定UI元素是渲染在Screen Space上还是World Space上。在Unity 3D中支持的渲染模式有三种：Screen Space-Overlay、Screen Space-Camera、World Space。

* Screen Space-Overlay
该渲染模式是默认的渲染模式。在该模式下会将所有的UI元素都渲染在场景中的最上层（类似于计算机屏幕上的贴膜，所有的UI元素都在这层贴膜上）。如果屏幕尺寸或者分辨率发生变化，Canvas也会自动去和变化后的尺寸相适应。

* Screen Space-Camera
这个渲染模式和Screen Space-Overlay比较类似。在该模式下Canvas游戏对象放置在一个预先设置好的摄像机的特定距离外，UI元素通过该摄像机进行渲染。所以使用该模式时应该创建一个摄像机并将其指定给Canvas组件下的Render Camera。改变该摄像机的设置时，UI元素的显示效果也会跟着改变。

* World Space
在这种渲染方式下，Canvas类似于一个游戏对象，可以手动改变其RectTransform组件以更改其大小与旋转。在渲染时，UI元素会根据它们在3D场景中的位置被渲染在其他游戏对象之前或之后，使其成为游戏视图中的一个成分。在做动态效果较多的界面时使用该模式比较方便。效果如下图所示。

{% asset_img 2.png %}

在Screen Space的两种渲染模式下UI界面独立于游戏场景，不会和场景中的其他对象进行遮挡，始终保持在最上层。而在World Space渲染模式下UI元素会和场景中的3D游戏物体进行遮挡，并且画布可被旋转缩放等，适合制作一些非常酷炫的UI界面效果。

### Graphic Raycaster

每个Canvas都有一个Graphic Raycaster组件，用于获取用户选中的UGUI控件，多个Canvas之间的事件响应顺序由其显示顺序决定，即在“Hierarchy”面板中越靠上的Canvas越后响应。当Canvas使用World Space或Camera Space渲染模式时，Graphic Raycaster的“Block”选项可以用来设置遮挡目标。

## EventSystem
创建一个UGUI元素后，Unity会同时创建一个游戏对象，其名为“EventSystem”，上面挂载了一系列组件用于控制各类事件。其自带的两个Input Module组件一个用于响应标准输入，一个用于响应触摸输入。在各个Input Module中封装了对Input模块的调用，根据用户操作触发对应的Event Trigger。

{% asset_img 3.png %}

EventSystem组件统一管理多个Input Module和各种Raycaster。该组件每帧调用多个Input Module处理用户的操作，同时还负责调用多个Raycaster用于获取用户点击到的UGUI控件或2D、3D物体。

EventSystem是Unity中的事件管理系统。对于UGUI中的控件点击监听等方法的实现将在下面进行对应的介绍，在本部分中读者只需要了解到EventSystem是一种将基于输入的事件发送到应用程序的对象，包括键盘、鼠标或自定义输入。

## RectTransform组件
UGUI中每个控件包括Canvas都会带一个RectTransform组件。该组件继承自Transform，用于控制UI元素的Transform信息。当读者向一个Empty Object（空对象）添加一个UI Component组件时，Transform组件会自动变为RectTransform，其参数介绍如下表所示。

| 参数名     | 含义  |
| :------   | :-------- |
|  PosX、PosY、PosZ  |   UI元素的位置   |
|    Width、Height    |  UI元素的宽度和高度  | 
|   Anchors    |  相对于父对象的锚点   |
|    Pivot    |   UI元素的中心  | 
|    Rotation    |   按轴旋转  | 
|    Scale    |   按轴缩放  | 

单击RectTransform组件中左上角的准星图标，可以在弹出的Anchor Presets面板中进行快速设置。按住Shift键时能同时设置pivot，这时能发现控件虽然不动但position已经在改变。如果按住Alt键，则能够在设置anchor的同时设置position。如果同时按住Shift键和Alt键，那么就能同时设置anchor、pivot和position。

## Panel控件
执行"GameObject->UI->Panel"命令,就会在Canvas游戏对象下创建一个Panel控件。该件是一个覆盖屏幕的平面,一般可以用来显示UI界面的背景,如图4-55所示。在其Image控件中, Source Image用于放置需要显示的Sprite, Color属性可以更改其颜色以及透明度,读者也而以自行调配材质然后拖拉到Material属性中。

Panel控件在默认情况下会自动根据屏幕的大小来调整自身的大小,所以不用担心其屏幕自适应问题。对于其上挂载的Image组件将在后面的Image控件一节中详细介绍。
    
## Button控件
按钮是每个界面的重要组成元素之一。在UGUI中执行“GameObject->UI->Button”命令，就可以创建一个按钮控件。创建出来的Button控件中包含了一个Text子对象（用于控制Button上显示的字样），若是读者不需要按钮上显示字样也可以将该子对象删除。

### 组件介绍
接下来介绍Button控件挂载的组件。每个按钮上都挂载有Button组件和Image组件，其中Image组件管理按钮的显示图片，Button组件管理单击按钮后的变化以及监听。具体内容如下：

* Image组件
    Button控件上的Image组件和之前介绍的Panel控件上的Image组件没有任何区别，在Source Image中可以放上合适的Sprite图片精灵，通过Color和Material可以设置图片的颜色和材质。

* Button组件
每个按钮上挂载的Button组件实现了按钮的全部功能，包括单击后的特效和点击的事件监听方法挂载。组件中各个参数的含义以及作用如下表所示。

| 参数名     | 含义  |
| :------   | :-------- |
|  Interactable  |   该按钮是否启用   |
|   Transition    |  按钮状态变化模式   |
|    Navigation    |  导航,使用键盘方向键切换选中按钮时的切换顺序  | 
|    Visualize    |   可视化,使Navigation在Scene窗口中可视化  | 

Button组件中的Transition过渡选项定义了四种过渡模式,分别为None、Color Tint、Sprite Swap和Animation。在除了None的每个模式中按钮都有4种状态：Normal (正常状态)、Highlight (突出显示), Pressed (按下状态)和Disable (禁用)，用户可以对每个状态的按钮过渡进行自定义。具体区别如下。
    (1) Color Tint过渡模式。当使用该模式时，可以分别通过Color属性对按钮的4个状态进行设置，按钮的颜色在对应的状态下就会变成设置的颜色，与正常状态产生区别。
    (2) Sprite Swap过渡模式。该过渡模式为精灵换图，同样的有4个状态可以设置，用户可以为每个状态下的按钮设置一个图片Sprite，设置完毕后当按钮处于对应状态下时就会显示出对应的图片。注意在设置各个状态图片的时候，图片也应当是一个Sprite，设置步骤可以参考Button控件Image组件部分中的说明。
    (3) Animation过渡模式。这个过渡模式是UGUI的特色,可以使UGUI界面系统和Unity中的动画系统进行完美的结合，使用动画状态机可以对不同状态下的按钮的位置、大小、旋转、图片等大量参数进行设置,功能非常全面。接下来将介绍一个使用Animation过渡模式的例子。口首先按步骤"GameObject->UI->Button"创建一个按钮,将其Button组件中的Transition选为"Animation",然后单击下方的Auto Generate Animation按钮,在弹出的对话框中找 合i的目录创建一个动画控制器。如图4-58所示。
    口创建好动画控制器后,按步骤"Window-Animation”打开动画编辑器窗口，然后单击Hierarchy面板中的上一步创建的Button控件，在Animation窗口中单击左上角的下拉框就可以选择想要编辑的按钮状态。如图4-59所示。

 2.按钮点击监听挂载
    以下将主要介绍如何给创建好的按钮挂载点击监听。当然,为按钮挂载点击监听的方法有很多,读者可以根据需要进行自主的选择,在本书中将要介绍的是通过Button组件中的"On Click()"事件参数添加按钮点击监听。具体步骤如下。
    (1)首先在"Project"面板中单击鼠标右键,执行"Create-C# Script"命令的创建一个C#脚本,将其命名为"UGUIOnClick.cs"后挂载到Canvas游戏对象上,双击打开即可开始编辑脚本。本脚本的功能相当简单,声明一个返回类型为空的方法,里面加上打印信息即可,脚本代码如下。
    using UnityEngine;
    using System. Collections;
    public class UGUIOnClick: MonoBehaviour
    {
        public void Onbticlick() //监听方法
        { 
            Debug. Log ("This is bt1");
        }
    }
    OnbtlClick方法就是场景中的按钮的点击事件监听方法。在Unity中将该方法,添加到按钮的点击事件列表中后,点击按钮就会自动回调该方法。
    (2)单击Button组件On ClickO下方的“+”按钮,给监听列表添加一个事件,如图4-62所示。将挂载有UGUIOnClick.cs脚本的游戏对象Canvas拖到图4-63所示的选框中,展开有NoFunction字样的下拉框,选择UGUIOnClick/ OnbtIClick即可。
    
        (3)运行场景,单击按钮后就会在"Console"面板中看到打印的"This is bl"字样。在指定监听方法时还可以进行传参。将上述代码进行修改,给OnbtIClick方法添加一个int类型的参数,然后保存。代码如下。

        using UnityEngine;
        using System.Collections;
        public class UGUIOnclick MonoBehaviour
        {
            public void Onbticiick (int index) //声明方法
            {
                Debug. Log ("This is bt"+index);
            }
        }

        在Unity中按钮的事件列表中为OnbtIClick设置参数后,单击该按钮,就会调用该方法,并且传入设置好的参数。在本案例中传入的参数类型是int型,Ae方法中就可以收到相应的参数。
        
        (4)重新在Button组件中的OnClickO列表中指定方法后,下方会多出一个输入框。在其中输入对应的int类型参数即可,如图4-64所示。设置完毕后运行游戏场景,单击按钮后就可以看到打印信息"This is bts".

## Toggle控件
            游戏的设置界面中经常能见到各种开关，在UGUI中开关控件Toggle就实现了开关的功能。执行“Game Object->UI->Toggle”命令就可以创建一个开关。
            Toggle控件的子对象中包含: 
            Background，即为一个Image控件，作为开关的背景；
            Checkmark，也是一个Image控件，用于显示选中后显示的图案；
            Lable是一个Text控件，可以用来显示开关的信息。在游戏中若是用不到Lable可以将其删除。
            Toggle控件的参数及其含义如下：
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
1.首先在Hierarchy窗口中选中Canvas游戏对象，单击鼠标右键选择“Create Empty”命令创建一个空对象以方便管理。然后依次创建3个Toggle控件，将其设置为GameObject（刚才创建的空对象）的子对象。在这里需要将创建的3个Toggle控件中的Is On设置为关闭状态。
2.选中上一步中创建的空对象GameObject，执行“Component->UI->Toggle Group”添加一个Toggle Group组件。其中的“Allow Switch Off”参数决定是否可以把打开的开关取消选中。
3.依次选中第一步创建的3个Toggle控件，将其中的Group参数选为挂载有Toggle Group组件的GameObject游戏对象。这样3个Toggle控件就成组了，最多只能同时选中其中的一个。

 ## Scroll View的制作
    所谓Scroll View就是滚动视图，在游戏中非常常见，比如游戏中的背包内容太多而无法一次显示完毕时就需要一个滚动视图，让玩家可以上下拖动以显示更多内容。
    UGUI也可以通过各个组件与控件的配合实现这一功能。下面介绍如何制作一个Scroll View。
    1.在Hierarchy面板中单击鼠标右键，执行"UI->Panel"命令创建一个Panel控件，并将其命名为“ScrollView“。更改其锚点值为"middle/center"并调整其大小为Width=280，Height=200。
    2.接下来选中ScrollView游戏对象，在Hierarchy面板中单击鼠标右键，选择”Create Empty“命令创建一个空对象，并将其命名为”Grid“，将其大小设置为280 X 370，选中该空对象后，创建12个Image组件，并将其设置为Grid的子对象。
    3.为游戏对象Grid添加Grid Layout Group组件。组件内部参数设置如下图所示。这时，其内部的12个Image子对象会自动根据布局管理器的设置进行摆放，调整Grid的高度即可得到下图所示的效果。

    4.经过上一步的设置后，可以看到12个Image组件已经超出了ScrollView的大小。接下来，为Scrollview添加Mask组件。这时，超出Scrollview大小范围的Image已经不进行显示了。

    5.接下来需要让ScrollView中的Image控件组可以被拖动。选中ScrollView游戏对象，添加Scroll Rect组件，并将游戏对象Grid拖到Scroll Rect组件中的“Content”选框中。

    6.经过步骤5，ScrollView中的控件组已经可以被拖动了。为了直观，可以在滚动视图右边创建一个Scrollbar控件，将其选择为垂直拖动模式。

    7.下面将滚动视图ScrollView和滚动条挂接。将ScrollView游戏对象上的Scroll Rect组件中的Vertical Scrollar选为上一步创建的“Scrollbar”，至此，滚动视图创建完毕，为Image组件赋子贴图后，运行场景，效果如下。

## UGUI中不规则形状的按钮的碰撞检测
    UGUI中自带的按钮是标准的矩形，虽然可以由玩家任意换图，但是其碰撞检测区域始终是矩形的。在有些时候，可能会用到特殊形状的按钮，当然其碰撞检测区域也要是符合按钮形状的。下面就使用UGUI中的知识来创建一个不规则形状的按钮。具体步骤如下。
    1.首先创建一个Button控件，命名为“bt1”，由于这里不需要它的Text子对象，所以可以将其删除。选中bt1后执行“Component->Physics2D->Polygon（多边形） Colloder（对撞机） 2D”命令，为bt1添加一个多边形碰撞器组件。
    2.单击Polygon Colloder 2D组件中的Edit Collider按钮，在Scene界面中将想要的碰撞检测区域勾选出来，如图中五边形边上的亮线所示。勾选出来的五边形即为该按钮的碰撞检测区域。
    3.接下来要实现将按钮的碰撞区域和Polygon Colloder 2D组件勾选区域的挂钩，这一步要重写Image类。新建一个C#脚本，将其命名为"UGUIImagePlus.cs"，该脚本需要引用UnityEngne.UI命名空间，并维承Image类，脚本代码如下。
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

 本脚本继承自UnityEngine.UI.Image类，并重写了该类的IsRaycastLocationValid说明 方法。这个方法用于返回触摸点（screenPoint）是否在图片范围内。在该方法中使用Collider2D.OverlapPoint方法判断点是否在多边形区城内。

    4.接下来将bt1上面挂载的Image组件移除，单击Inspector面板中的Image组件右边的设置按钮，选择Remove Component。将UGUIImagePlus.cs类拖拉到bt1上形成UGUIImagePlus组件以代替。
    5.挂载好UGUIImagePlus组件后，重新为该组件指定好纹理图。这样，图片已经可以检测到由Polygon Colloder 2D组件勾选出来的区域内的触摸了。

-----------------------------

## UGUI布局管理的使用及相关组件介绍
        前面的几节中介绍了UGUI中的控件的相关知识，在介绍完控件知识后，接下来将讲解如何去管理、排布多个控件。这部分知识的运用常见于游戏中的奖励窗口，由于预先不知道获得奖励的数量，但是依旧需要让获得的奖励道具按照一定的布局整齐地出现在界面中，这时就需要用到布局的知识了。
    Unity中自带的布局模式分为“Horizontal Layout Group”、“Vertical Layout Group”和“Grid Layout Group”三种，分别为“水平布局”、“垂直布局”和“网格布局”。首先创建5个Image控件，并将其放在一个空对象”UIMain“下成为其子对象。
    1.Horizontal Layout Group水平布局
    选中UIMain空对象，执行"Component->Layout->Horizontal Layout Group"命令，即可为该游戏对象添加一个水平布局管理器组件。顾名思义，在该组件的作用下，UIMain的子对象将按照一定的要求进行水平排列。

| 参数 | 含义 |
| :------------ |:------------|
| Padding  |   布局的边缓填充（即偏移） |
| Spacing    |   布局内的元素间距 |
| Child Alignment    |  对齐方式 |
| Child Force Expand     |  自适应宽和高 |


为UIMain添加Horizontal Layout Group组件后，其所有UI元素子对象都会根据对Horizontal Layout Group组件的设置进行水平的自动排列。

2.Vertical Layout Group（垂直布局）
    选中UIMain游戏对象，执行“Component->Layout->Vertical Layout Group”，就可以给该游戏对象添加一个垂直布局管理器。垂直布局管理器的功能为将UI元素按照一定的规则进行整齐的垂直排列。其内部参数和Horizontal Layout Group的参数完全一样，所以不再赞述。

    3. Grid Layout Group（网格布局）
    Grid Layout Group是网格布局管理器组件，这个组件会自动地将其管理下的UI元素进行网格状排列，其实现了自动换行等功能，常见于各个游戏中的背包，内部的储物格成网格状排列。组件的内部参数如下表所示。

|  参数名  | 含义  |
| :------------ | :------------ |
| Padding  | 偏移  |
| CellSize  | 内部元素的大小  |
| Spacing  | 每个元素间的水平间距和垂直间距  |
| Start Corner  | 第一个元素的位置  |
| Start Axis  | 元素的主轴线  |
| Horizontal  | 在填满一行后启用一个新行  |
| Vertical  | 在填满一列后启用一个新列  |
| Child Alignment  | 对齐方式  |
| Constraint  | 指定网格布局的行或列  |

以上即为Unity中自带的3种布局管理模式，在大部分情况下可以满足开发的需要。在游戏运行时随时将新实例化的UI控件或者游戏对象设置为挂载有Layout Group组件（3种任意一个皆可）的游戏对象的子对象，Layout Group组件便会对其进行自动布局排列。
using UnityEngine;
using System.Collections;

public class UGUILayout : MonoBehaviour 
{
	public GameObject UIMain;  //挂载有Layout group组件的游戏对象
	public GameObject items; //需要实例化的UI控件或者游戏对象的预制件

	void Start ()
	{
		GameObject item = (GameObject)Instantiate(items); //实例化items
    	item.transform.parent = UIMain.transtorm;//将实例化的游戏对象设置为UIMain的子对象	
	} 
}



在Start方法中新实例化出了一个items预制件，将其设置为挂载有布局管理器组件的UIMain的子对象，然后观察场景，就会发现新实例化的预制件已经被自动排列好了。这在游戏开发中非常方便，可以随时实例化UI元素而不用再三考虑排布问题。

2.需要注意的是，导入的图片类型是".PNG"类型的,在使用前需要将导入的图片在Inspector面板中设置为精灵模式"Sprite(2D and UI)",这样才能在Image控件中正常显示出来。然后将Format选为"Truecolor".如图4-103所示。

        3.接下来开始搭建UI界面。在本部分中所使用的控件较多,各种控件的创建方法在前面的介绍中都有所讲解,所以在本部分中不再赘述。接下来将用表格的形式将场景中Canvas (画布)下的游戏对象Musi Plaer及其子对象进行介绍,读者可以按照表4-16所法中的内及层级关系依次进行创建。