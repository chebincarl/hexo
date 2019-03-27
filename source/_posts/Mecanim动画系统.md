---
layout: title
title: Mecanim动画系统
date: 2019-02-17 14:14:41
categories: Unity
tags: Unity
---
旧版动画系统中，游戏开发人员只能通过代码操控角色动画的播放，随着动画个数的增多，其代码复杂度也随之增加。同时，动画的过渡需要烦琐的代码控制，这就使得缺乏编程经验的游戏动画师很难对动画效果进行处理。
Mecanim动画系统就是为了解决这个问题，该动画系统使游戏动画师能够参与到游戏的开发中来。
<!-- more -->

# Unity与建模软件单位的比例关系

Unity默认的系统单位为m，例如在Unity中新建一个Cube游戏对象，其长宽高都是一个单位，即1m。但3D建模软件默认的系统单位并不都是m。为了让模型可以按照理想的尺寸导入Unity，就需要调整建模软件的系统单位或者尺寸。在3D建模软件中，应尽量使用“米”制单位。

# 导入模型

将模型导入Unity中，依次单击菜单栏的“Assets->Import New Assets..."立刻弹出导入资源对话框，找到并选中模型，单击“Import”按钮就可完成导入。此时Unity的Assets面板里就会出现此模型。

# 角色动画的配置

导入了角色动画资源之后，需要对角色动画进行适当的配置才能被Mecanim动画系统所识别和使用。Mecanim动画系统非常适合用于对人形角色动画的控制。

1.创建骨骼结构映射--Avatar

把带动画的模型文件拖曳进Unity 3D中时，系统会自动为模型文件生成一个Avatar文件作为其子对象。Avatar是Mecanim动画系统自带的人形骨骼结构与模型文件中的骨骼结构间的映射，但此时选中Avatar文件时只会出现空白视口，且无法对其进行配置。

选中人形角色模型文件，在右边的Inspector视口中单击“Rig”按钮，出现视图，点击Animation Type下拉按钮，选择Humanoid选项，并单击右下角的“Apply”按钮应用该选择。至此，该模型文件已经被指定为人形角色模型，同时，系统重新为其创建了Avatar文件。

Animtion Type下拉列表有四个选项，分别为“None”、“Legacy”、“Generic”和“Humanoid”，分别对应无模式、旧版动画模式、其他动画模式和人形角色动画模式。运用于Mecanim动画系统中的人形角色动离都要选择“Humanoid”。

2.配置Avatar

(1)首先选择模型文件下的Avatar文件，在Inspector视口中就会出现一个“Configure Avatar”按钮。只需单击该按钮即可进入Avatar的配置窗口。

(2)同时，系统会弹出提示窗口，用于提示是否保存场景中的所有信息。这是由于在配置Avatar的时候，系统会关闭原场景窗口，并开启一个临时Scene视口作为配置Avatar的实时显示窗口，并在配置结束后关闭该临时窗口。

(3)单击“Configure Avatar”按钮之后Inspector就会变成下图的视口。同时，Scene视口会出现骨骼，Inspector中参数的改变会实时显示在Scene视口中，可以在Scene视口中实时地看到Avatar的效果，而不必再重建场景验证其准确性。

{% asset_img 1.png %}

(4)可分别单击Avatar配置视口中的“Body”、“Head”、“Left Hand”和“Right Hand”等按钮进行Avatar不同层次的配置。可以在不同的窗口进行不同部位的骨骼配置，这样做的好处是各个骨骼层次配置互不影响，并能同时播放。

(5)一般情况下Uniy 3D都会正确地对Avatar初始化，但有时候会因为骨骼的名字不规范等原因，Unity 3D不能准确地识别到相应的骨骼，此时就需要使用系统自带的工具手动对其进行校正。

(6)当遇到这种情况的时候，可以在Hierarchy视口中找到正确的骨骼，然后将正确的骨骼拖曳到Inspector视口中Optional Bone下的指定位置中。若所有骨骼都变成绿色，则代表Avalar已经配置完成。

3.Muscle的配置

实际的开发过程中，开发人员可能会遇到一些骨骼动画动作过于夸张的情况，如果使用的是旧版动画，就需要重新制作该动画，而Mecanim动画系统则为其提供了一套解决方案，读者可以通过设置Avatar中的Muscle参数，来限制角色模型各个部位的运动范围，防止某些骨骼运动范围超过合理值。

(1)单击Avatar视口中的“Muscles”按钮进入Muscle的配置窗口。该窗口由预览窗口、设置窗口及附加配置窗口组成。

{% asset_img 2.png %}

(2)下面以左脚骨骼为例对其进行调整。选中设置窗口中的Left Leg参数，其附带的所有子参数也会随之展开，可以通过拖动参数左边的拖拉条，观察指定的骨骼的运动范围，同时Scene视口会在对应的骨骼上生成若干个扇形，代表骨骼旋转的范围。

{% asset_img 3.png %}

(3)单击Upper Leg Front-Back参数可展开配置参数。可通过拖动其拖拉条或设置其左右参数对该骨骼的运动范围进行调整，Scene窗口中骨骼对应扇形的大小也会随之改变。下图所示的就是Upper Leg FrontBack范围为0 ~ 10的预览窗口。

{% asset_img 4.png %}

{% asset_img 5.png %}

(4)设置完毕之后单击配置窗口右下角的“Done”按钮结束Muscle的配置。重新播放该动画，如果骨骼的最大运动范围与动画中的运动范围有相交，则在更改后的动画中，其骨骼只会在设置的范围内运动。

除了防止过于夸张或错误的动作，设置Muscle参数还可以实现对原动画的修改。比如原动画是一个边奔跑边招手的动作，而开发所需的仅仅是一个单纯奔跑的动画，那通过限制手部的运动，便可以快速地完成动画的修改。

# 动画控制器的创建

Mecanim动画系统引入了动画控制器的概念，通过动画控制器可以把大部分动画相关的工作从代码中分离出来，游戏动画师可以独立地完成动画控制器的创建，且不涉及任何代码。下面将介绍动画控制器的创建。

(1)首先创建一个名为“Mecanimstudy”的工程项目，然后将资源目录下的Animations、Models、Textures等文件夹依次复制进本项目中的Assets资源文件夹下，然后创建一个名为“AniControllers”的空文件夹，用于存放项目所需的动画控制器文件。

(2)读者可在AniControllers文件夹中单击鼠标右键，在弹出菜单中依次单击“Create->Animator Controller ”创建一个动画控制器，并命名为“StaticAnimatorController”。双击该动画控制器，进入动画控制器编辑窗口。

{% asset_img 6.png %}

# 动画控制器的配置

配置动画控制器是学习Mecanim动画系统的重点。

1.动画状态机和过渡条件

理解动画控制器中的方块的含义之前，需先理解Mecanim动画系统中动画状态机的概念。该动画系统基于状态机思想对游戏动画进行控制。通过使用动画状态机，游戏动画师可以进行无代码的可视化开发。下面通过下表向读者展示了动画控制器中必要的状态机意义。

| 名称  | 说明  |
| :------------ | :------------ |
| State Machine  |  动画状态机，可包含若干个动画状态单元 |
|  State | 动画状态单元，动面状态机机制中的最小单元  |
| Sub-State Machine  | 子动画状态机，可包含若干个动画状态单元成子动画状态机 |
|  Blend Tree | 动画混合树，一种特殊的动画状态单元  |
| Any State  | 特殊的状态单元，表示任意动画状态  |
|  Entry | 本动画状态机的入口  |
|  Exit |  本动画状态机的出口 |

{% asset_img 7.png %}

每一个动画控制器都可以有若干个动画层，每个动画层都是一个动画状态机，动画状态机中可以同时包含若干个动画状态单元或子动画状态机。每一个动画状态机都必然会含有“Any State”、“Entry”、“Exit”动画状态单元，用于实现该状态机不同的必须功能。

下面简单介绍一下动画状态单元和动画过渡条件的搭建，其详细步骤如下所示。

(1)可以通过单击鼠标右键，在弹出菜单中选择“Create State->Empty”创建空动画状态单元，也可以将动画片段直接拖曳进动画状态机编辑窗口中进行创建。在此笔者通过向编辑窗口拖曳进“Boy@ForwardKick”和“Boy@KickBack”两个动画文件，创建两个动画状态单元。

(2)然后将鼠标箭头放在动画状态单元上，单击鼠标右键选择“Make Transition”创建动画过渡条件，并再次点击在另一个动画状态单元上，完成动画过渡条件的连接。Mecanim动画系统通过动画过渡条件实现各个动画片段之间的逻辑，开发人员只需控制这些过渡条件即可实现对动画的控制。

(3)为了实现所需效果，笔者已将该动画状态机搭建成下图所示的状态，读者可按着笔者所搭建的状态进行连接，在这个动画状态机中，Idle被设为默认动画且显示为黄色，其他动画状态单元则显示为灰色，读者可以在任意非默认动画单元上单击鼠标右键选择“Set As Default”将其设置为默认动画。

{% asset_img 8.png %}

2.过渡条件的参数设置

动画状态机和过渡条件搭建完成之后，就需要对状态机间的过渡条件进行设置。为了实现对各个过渡条件的操控，需要创建一个或多个参数与之搭配。Mecanim支持的过渡参数类型有Float、Int、Bool及Trigger，其在动画控制器代表的意义需要游戏动画师提前设计好。

(1)下面向游戏控制器添加一个Float类型的参数实现对游戏过渡条件的控制。点击Parameters视口中的“+”添加一个Float类型的参数，并命名为“AniFlag”，设置其初始值为-1.0。

{% asset_img 9.png %}

(2)然后选中任意一个过渡条件，在Inspector视口中的Conditions列表中点击“+”按钮添加参数控制，读者进行参数的设置，为参数添加对比条件。Mecanim动画系统为Float类型的参数提供了“Greater”和“Less”对比条件。

{% asset_img 10.png %}

{% asset_img 15.png %}

3.代码对游戏控制器的控制

(1)动画控制器创建和配置完成之后，接下来创建一个名为“MecanimBehaviour”的场景来测试该游戏控制器是否可用，在该场景中创建一个地形，给地形添加绿色草地纹理，再将Models文件夹下的Boy模型文件拖曳到场景中，并调整光照方向至合适角度。

{% asset_img 11.png %}

(2)接下来进行UI界面的开发。读者可依次单击“GameObject->UI->Button”创建一个按钮，并命名为“Button0”，按照此步骤再创建一个“Button1”按钮，这两个按钮分别用于对两个动画的控制，当按下任意一个按钮时，系统将启动对应的动画过渡。

{% asset_img 12.png %}

(3)选中Boy游戏对象，为其添加一个Animator组件，并将先前创建的StaticAnimatorController动画控制器掩曳到Animator组件下的“Controller”框中。然后再新建一个C#脚本，将其命名为“StaticAniCtrl.cs”，并把脚本拖曳给Boy对象。

{% asset_img 13.png %}

StaticAniCtrl脚本用于实现对动画控制器的控制、游戏按钮的响应以及摄像机的跟随。
```cs
using UnityEngine;
using System.Collections;

public class StaticAniCtrl : MonoBehaviour 
{
    Animator myAnimator; // 声明Animator组件
    Transform myCamera; // 声明摄像机对象

    void Start () 
    {
        myAnimator = GetComponent<Animator>(); // 初始化Animator组件
        UIInit(); // 初始化UI界面
        myCamera = GameObject.Find("Main Camera").transform; // 初始化摄像机对象
    }  

    void Update() 
    {
        myCamera.position = transform.position + new Vector3 (0, 1.5f, 5); // 摄像机对象跟随
        myCamera.LookAt(transform); // 摄像机对象朝向
    } 

    void UIInit()
    {
        // 按钮位置
        GameObject.Find("Canvas/Button0").transform.GetComponent<RectTransform>().localPosition = new Vector3(Screen.height / 6- Screen.width / 2, Screen.height * 2/ 5 - Screen.height /2);
        // 按钮大小
        GameObject.Find("Canvas/Button0").transform.GetComponent<RectTransform>().localScale = Screen.width / 600.0f * new Vector3(1, 1, 1);

        // 按钮位置
        GameObject.Find("Canvas/Button1").transform.GetComponent<RectTransform>().localPosition =  new Vector3(Screen.height / 6 - Screen.width / 2, Screen.height / 6 - Screen.height /2);
        // 按钮大小
        GameObject.Find("Canvas/Button1").transform.GetComponent<RectTransform>().localScale =  Screen.width / 600.0f * new Vector3(1, 1, 1);
    }

    public void ButtonOnClick(int index) 
    {
        Debug.Log(index);
        myAnimator.SetFloat("AniFlag", index); // 向动画控制器传递参数
    }
}
```
* 第1-20行用于Start方法和Update方法的开发。在Start方法中，初始化了Animator组件和摄像机对象， Aniamtor用于动画的播放控制，而摄像机对象则在Update方法中进行调用。Update方法中进行了摄像机对象的跟随操作，使摄像机对象与游戏角色对象相互关联。
        
* 第22-33行用于UIInit方法的开发，用于初始化游戏按钮，使本案例在任意分辨率的屏幕中都能正常运行，不至于被拉伸。
        
* 第35-38行用于按钮回调方法的开发，当被指定的按钮被按下时，系统将会调用此方法。本函数将会根据按下按钮的不同，向Animator组件传递相对应的参数值，动画控制器获得该参数之后，将对指定的过渡条件进行调控，从而实现对动画播放的操控。

(4)给Button0和Button1的On Click()添加回调，参数分别设置为1和2，如下图所示。

{% asset_img 14.png %}

(5)单击运行按钮之后，案例的运行效果会显示在Game窗口中。点击屏幕上的两个按钮，可以使场景中的小男孩做出不同的动作。

{% asset_img 20.png %}

# 角色动画的重定向

角色动画的重定向是Mecanim动画系统的一大特色功能。Unity 3D提供了一套用于人形角色动画的重定向机制，游戏美工只需独立地制作好所有角色模型，而游戏动画师也可独立地进行动画的制作，且两者不互干涉，只需在Mecanim动画系统中稍做处理即可使用。

1.角色动画重定向原理

* 人形角色模型绑定的骨骼架构所包含的骨骼数量和名称不尽相同，难以实现动画的通用。为了解决这一个问题，Mecanim动画系统提供了一套简化过的人形角色骨骼架构，而Avatar文件就是模型骨骼架构与系统自带骨骼架构间的桥梁，重定向的模型骨骼架构都要通过Avatar与自带骨骼架构搭建映射。

* 映射后的模型骨骼可能通过Avatar驱动系统自带骨骼运动，这样就会产生一套通用的骨骼动画，其他角色模型只需借助这套通用的骨骼动画，就可以做出与原模型相同的动作，即实现角色动画的重定向。通过这项技术的运用，可以极大地减小开发者的工作量，以及项目文件和安装包的大小。

2.角色动画重定向的应用

(1)新建一个场景，在场景中创建两个游戏对象用于演示，将其分别命名为“Boy”和“Girl”，如下图所示，再创建一个动画控制器并命名为“SetParController”，然后再分别拖曳到两个游戏对象的Animator组件中的Controller选择框内。

{% asset_img 19.png %}

{% asset_img 16.png %}

(2)然后再创建一个C#脚本并将其命名为“AniController”，然后把脚本拖曳到Boy对象上。该脚本用于操控角色动画的播放、实现动画按钮的回调、实现摄像机对象的跟随。
```cs
using UnityEngine;
using System. Collections;

public class AniController : MonoBehaviour 
{
    #region Variables
    Animator animator; // 声明Boy对象动画控制器
    Animator girlAnimator; // 声明Girl对象动画控制器
    Transform myCamera; // 声明摄像机对象
    #endregion
    
    #region Function which be called by system

    void Start ()
    {
        animator = GetComponent<Animator>(); // 初始化Boy对象动画控制器
        girlAnimator = GameObject.Find("Girl").GetComponent<Animator>(); // 初始化Girl对象动画控制器
        UIInit(); // 初始化界面
        myCamera = GameObject.Find("Main Camera").transform; // 初始化摄像机对象
    } 

    void Update () 
    {
        myCamera.position = transform.position + new Vector3(0, 1.5f, 5);//摄像机跟随        
        myCamera.LookAt(transform);//摄像机朝向
    }

    #endregion
    #region UI recall function and setting

    public void ButtonOnclick(int Index) // 按钮回调事件  
    {
        bool[] pars = new bool[] {true, false}; // 声明启动数组
        animator.SetBool("JtoR", pars[Index]); // 传递控制参数
        animator.SetBool("RtoJ", pars[(Index+1) % 2]); // 传递控制参数
        girlAnimator.SetBool("JtoR", pars[Index]);  // 传递控制参数
        girlAnimator.SetBool("RtoJ", pars[(Index + 1) % 2]);//传递控制参数   
    }

    void UIInit() 
    {
        //按钮位置
        GameObject.Find("Canvas/Button0").transform.GetComponent<RectTransform>().localPosition = new Vector3(Screen.height / 6 - Screen.width / 2, Screen.height * 2 / 5 - Screen.height /2);

        //按钮大小
        GameObject.Find("Canvas/Button0").transform.GetComponent<RectTransform>().localScale = Screen.width / 600.0f * Vector3.one;

        //按钮位置  
        GameObject.Find("Canvas/Button1").transform.GetComponent<RectTransform>().localPosition = new Vector3(Screen.height / 6 - Screen.width / 2, Screen.height / 6 - Screen.height / 2);

        //按钮大小
        GameObject.Find("Canvas/Button1").transform.GetComponent<RectTransform>().localScale = Screen.width / 600.0f * Vector3.one;  
    }
    #endregion
}
```

* 第6-9行是进行参数的声明。

* 第14-26行的主要功能是Start方法和Update方法的开发。在Start方法中，进行了两个Animator组件的初始化，以便后续代码中进行参数传递，同时进行了UI界面的初始化，使其在不同分辨率的屏幕中都可以正常运行。

* 第31-38行的主要功能是进行按钮回调事件的开发和UI界面的初始化，当有任意一个按钮被按下时，系统将会调用此方法，并根据按下按钮的不同，进行不同的操作，向动画控制器传递一个特定的参数，从而实现对动画的操控。

{% asset_img 17.png %}

(3)接下来单击运行按钮，其运行效果就会呈现在Game窗口中。当读者点击任意一个按钮时，两个游戏角色对象就会做出相同的动作。两个角色对象通过Mecanim中的动画重定向功能，同时播放同一个动画。

{% asset_img 18.png %}

# 角色动画的混合--创建动画混合树

实际的游戏开发过程中，有时候会有将两个动画混合成一个动画的需求，比如要做一个边跑边招手的动作等。在Unity 3D 4.0版本以前，想要做这样的动作只能重新制作一个动画，而如今Mecanim动画系统为开发人员提供了另一种途径，那就是角色动画的混合。

(1)首先新建一个动画控制器，并将其命名为“BlendController”。打开动画控制器编辑窗口，单击鼠标右键后依次单击“Create State->From New Blend Tree”创建一个角色动画混合树，并命名为“Blend Tree”。

(2)不难发现，动画混合树的创建按钮是“Create State”的子按钮，从中可以发现动画混合树实际上也是一个动画状态单元，在动画状态机看来，其体现出来的作用与普通动画状态单元并无区别，只是动画混合树能够将若干个动画混合成一个动画进行处理而已。

(3)双击前面创建的动画混合树，进入混合树编辑窗口。接下来新建一个Float类型的参数，并将其命名为“BlendPar”，该参数用于对动画混合的控制，Mecanim动画系统会根据这个参数值的大小，对该动画混合树进行配置。


{% asset_img 22.png %}

(4)回到Inspector窗口，将Parameter参数设置为“BlendPar”，然后在Motion列表的右下角点击“+”符号添加两个动画条目，然后将Assets\Animations\FighAnis目录下的“Boy@JumpTurnKick”和“Boy@StepSideKick”动画分别拖曳到对应框内。

{% asset_img 21.png %}

(5)接下来搭建一个场景，并命名为“MecanimBlend”，接着将Assets\Models目录下的Boy角色模型拖曳到场景中去。将BlendController动画控制器拖曳给Boy对象的Animator组件，然后单击运行按钮，在Game窗口中就显示了其运行效果。

{% asset_img 23.png %}

开发这个动画混合树时，笔者使用了简单的1D混合方式进行混合，BlendPar参数在其中充当了混合因子的作用。除了1D混合方式，Mecanim动画系统还同时支持其他动画混合方式。
    
# 角色动画的混合--混合类型介绍

角色动画混合的强大之处在于动画混合树的混合方式，不同的混合方式和巧妙的参数设置，可以混合出丰富的动画效果。动画混合树编辑窗口中的“Blend Type”下拉列表中有多个选项，下面将详细讲解这几个参数的意义和用法。

1.1D混合方式
1D混合方式是最简单的动画混合方式，也是最常用的一种。每个被混合的子动画都会被分配到一个可修改的Float类型的值，开发人员通过改变挂载的混合参数实现不同的混合效果，混合参数越接近某个动画值，则该动画在混合结果中占的比例就越大。

这种混合方式的缺点是每个混合动画只能由最多两个原动画混合而成，这在一些特殊情况下就很难满足要求。而Mecanim动画系统提供的2D混合方式则刚好解决了这个问题。

2.2D Simple Directional混合方式
2D Simple Directional以两个混合参数作为被混合结果动画的横纵坐标值，混合动画和混合动画以正方形的形式分布在混合面板中，各自的混合比例用正方形外围的圆圈表现出来。每个动画的分布也以颜色深浅形象地表现出来。

3.2D Freeform Directional混合方式
使用了2D Freeform Directional混合方式的动画混合时，原动面的分布以另外一种方式存在。每个原动画都是一个放射性的显示面板，颜色越白动画权重越大；反之则越小同时读者可以通过移动原动画点，对显示面板进行调整。

4.2D Freeform Cartesian混合方式
而2D Freeform Cartesian则是另一种混合方式，原动画用与其他动画相连的渐变表示。与其他混合方式相同，这种混合方式也通过两个混合参数控制混合动画效果，并以混合面板中的颜色深浅代表各个子动画在混合动画中的权重。

动画控制树中的混合参数在使用的过程中不可以设置为刚好等于某个原动画的值，否则将出现不可知错误。要知道，动画混合树充当的仅仅是混合的作用，不带任何的逻辑成分，不要试图通过混合树实现某段动画的关闭或开启，那样的功能只能通过搭建状态单元和过渡条件来完成。

# Mecanim中的代码控制

1.StateMachineBehaviour脚本
开发人员可以为动画状态机或动画状态单元添加继承于StateMachineBehaviour类的脚本，用于在指定动画的播放过程中进行自定义操作，可在该脚本中进行下表所示方法的重写，这些方法在StateMachineBehaviour类中已经被定义。

| 方法签名  | 说明  |
| :------------ | :------------ |
| OnStateEnter(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)    |  当动画开始播放的时候被调用一次 |
| OnStateUpdate(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)  |   当动画已经在播放时，每一帧调用一次 |
| OnStateExit(Animator animator, AnimatorStateInfo stateInfo, int layerIndex) | 当动画结束播放时播放一次  |
| OnStateMove(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)  | 当动画被移动时播放  |
| OnStateIK(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)  |  当动面触发逆向运动学时调用此方法 |

简单的案例：
(1)首先创建一个C#脚本，将其命名为“FKBehaviour”，并使其维承于StateMachineBehaviour类。该脚本的主要功能是实现对角色对象挂载的脚本的开启和关闭。与其他脚本不同的是，该脚本的挂载对象是动画状态单元，而不是游戏对象。其详细代码如下所示。

```cs
using UnityEngine;
using System.Collections;

public class FKBehaviour : StateMachineBehaviour 
{
    // 动画开始播放时进行的操作
    override public void OnStateEnter(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
    {
        // 开启脚本
        GameObject.Find("Boy").GetComponentInChildren<MeleeWeaponTrail>().enabled = true;
    }

    // 动画结束时进行的操作
    override public void OnStateExit(Animator animator, AnimatorStateInfo stateInto, int layerIndex)
    {
        // 关闭脚本
        GameObject.Find("Boy").GetComponentInChildren<MeleeWeaponTrail>().enabled = false;
    }
}
```

该脚本主要用于OnStateEnter和OnStateExit方法的重写。这两个方法分别在被挂载动画开始播放和结束播放时运行，并开启和关闭挂在Boy对象上的Melee Weapon Trail脚本。

(2)接下来打开前面创建的MecanimBehaviour场景，把Assets/Scripts目录下的Melee Weapon Trail脚本拖曳给Hierarchy窗口中的Boy/Boy/Boy Pelvis/Boy Spine/Boy R Thigh/Boy R Calf下的Boy R Foot对象上。
Melee Weapon：近战武器
Pelvis：骨盆
Spine：脊柱
Thigh：大腿
calf：小腿肚

(3)然后给Boy R Foot创建名为Base和“Tip”子对象，再将这两个子对象分别拖曳到Melee Weapon Trail脚本中的Base和Tip条目中。该脚本主要用于使Boy对象的右脚产生一个划痕，前面开发的FKBehaviour脚本通过开启和关闭本脚本来说明其作用。

(4)双击前面创建的StaticAnimatorController动画控制器，选中ForwardKick状态单元，单
击Inspector窗口中的“Add Behaviour”，在弹出的下拉框中选择FKBehaviour脚本。

(5)然后单击运行按钮，观察Game窗口。当点击按钮1时，Boy对象播放KickBack动画，此时运行效果与其他动画相比并无异样；而当点击按钮0时，Boy对象播放ForwardKick动画，Boy对象的右脚就会出现一道划痕。

2.通过代码生成动画控制器
如果需要创建一个带有10个动画状态单元的动画控制器，任意一个动画都可以过渡到其他动画包括自身，那就需要为这个动画控制器搭建100个过渡条件，直接搭建不仅工作量大，也不便于以后的修改和维护，因此有必要掌握通过代码动态生成动画控制器的方法。
下面通过一个简单的案例讲解动态生成动画控制器的方法。
(1)打开MecanimStudy工程项目，在Assets目录下创建一个名为“Editor”的文件夹，该文件夹用于存放编辑器类脚本文件。在该文件夹中创建一个C#脚本,并命名为“CreateController”。
```cs
using UnityEngine;
using System.Collections;
using UnityEditor.Animations;
using UnityEditor;

public class CreateController : Editor // 该类继承于编辑器类
{ 
	[MenuItem("CreateAnimator/CreateDynamicController")] //指定按钮
	static void Run()
	{
		// 生成控制器
		AnimatorController dynamicController = UnityEditor.Animations.AnimatorController.CreateAnimatorControllerAtPath("Assets/AniControllers/DynamicController. controller"); 
		// 根动画
		AnimatorStateMachine rootStateMachine = dynamicController.layers[0].stateMachine;
		AnimatorState[] states = new AnimatorState[10];  // 声明动画状态单元集合
		for (int i = 0; i < states.Length; i++) // 遍历动画状态单元集合
		{
			states[i] = rootStateMachine.AddState("state" + i); // 向状态机添加动画
			states[i].speed = 1.5f;  // 初始化动画播放速度
		}

		rootStateMachine.defaultState = states [0];  // 初始化根动画
		AnimationClip[] anis = new AnimationClip[10];  // 声明动画片段集合

		for (int i = 0; i < anis.Length; i++) // 获取动画片段
		{
			anis[i] = AssetDatabase.LoadAssetAtPath("Assets/Animations/AnisWithNum/Ani" + i + ".FBX", typeof(AnimationClip)) as AnimationClip; // 获取动面片段
		
			states[i].motion = anis[i];  // 设置动画状态中的动画片段
			states[i].iKOnFeet = false;  // 关闭逆向运动学
		}

		for (int i = 0; i < states.Length; i++) // 构建动画过渡条件
		{ 
			for (int j = 0; j < states.Length; j++)
			{  
				dynamicController.AddParameter("state" + i +"TOState" + j, AnimatorControllerParameterType.Trigger);  // 在动画控制器中生成一个触发器参数
				
				AnimatorStateTransition trans = states[i].AddTransition(states[j], false); // 生成触发器
				
				trans.AddCondition(AnimatorConditionMode.If, 0, "state" +i + "TOstate" + j); // 指定触发器参数
			}
		}
		states[states.Length - 1].AddExitTransition(); // 指定输出动画
	}
}
```

口第1-25行的主要功能是进行动画控制器的创建，同时在动画控制器中创建10个动画状态单元，然后把Assets/Animations/AnisWithNum目录下的10个动画分别配置到这10个动画状态单元中去，最后再进行动画速度和逆向运动学的设置，统一其运行效果。

口第26-32行的主要功能是给前面创建的任意动画状态单元之间创建动画过渡条件，同时为每一个过渡条件创建并匹配一个过渡参数，这些参数根据前后动画名进行命名，以便在控制脚本中进行控制，最后给动画控制器指定结束动画，完成本脚本的开发。

(3)现在打开Unity 3D，可以在菜单栏见到"CreatAnimator/CreateDynamicController"按钮,如图10-109所示,该按钮在CreateController脚本中声明。单击该按钮,在Assets/AniControllers目录下将会生成一个名为"DynamicController"的动画控制器,如图10-110所示。

 (4)双击刚生成的DynamicController动画控制器，可在动画控制器编辑窗口查看其详情，拖动上面的动画状态单元，可发现其结构比想像中的复杂得多，同时，该动画控制器携带了大量的过渡参数。而这些复杂的结构均由CreateController脚本动态生成。

(5)接下来创建一个场景来检验一下动态动画控制器的可行性。新建一个名为“MecanimCreate”的场景，并将AssetsModels目录下的Boy模型文件拖曳到场景中，并创建10个按钮。然后把前面创建的DynamicController动画控制器拖曳到Boy对象的Animator组件中。
        
(6)接下来创建一个C#脚本，并将其命名为“DynamicAniCtr”，然后再拖曳到Boy对象上。该脚本用于操控角色动画的播放、实现动画按钮的回调、实现摄像机对象的跟随。
```cs
using UnityEngine;
using System.Collections;

public class DynamicAniCtrl : MonoBehaviour
{
	private Animator myAnimator; // 声明Animator组件
	private Transform cameraHandle; // 声明摄像机对象

	void Start() 
	{
		cameraHandle = GameObject.Find("Main Camera").transform; // 初始化摄像机对象
		myAnimator = GetComponent<Animator>(); // 初始化动画组件
		UIInit(); // 进行UI界面的初始化
	}
	
	void Update()
	{
		cameraHandle.position = transform.position + new Vector3(0, 1.2f, 4); // 摄像机跟随
		cameraHandle.LookAt(transform); // 摄像机朝向 
	}

	void UIInit()
	{
		//UI界面的初始化
		Transform uiCanvas = GameObject.Find("Canvas").transform; //获取UI引用
		for (int i = 0; i < uiCanvas.childCount; i++)  //遍历UI集合 
		{  
			// 设置按钮位置
			uiCanvas.GetChild(i).GetComponent<RectTransform>().localPosition = new Vector3(-Screen.width * 0.4f * i / 5 * Screen.height / 6, Screen.height / 3 - i % 5 * Screen.height / 6, 0);
			// 设置按钮大小
			uiCanvas.GetChild(i).GetComponent<RectTransform>().localScale = Screen.width / 600.0f * new Vector3(1, 1, 1);
		}
	}
		
	public void ButtonOnClick(int index) // 按钮回调事件
	{  
		for (int i = 0; i < 10; i++) // 遍历所有按钮
		{
			if (myAnimator.GetCurrentAnimatorStateInfo(0).IsName("state" + i)) // 当按下指定按钮
			{
				myAnimator.SetTrigger("state" + i + "TOstate" + index); // 激活指定触发器
				return; // 结束遍历
			}
		}  
	}	
}
```

口第1-14行的主要功能是进行Start和Update方法的开发。在Start中进行了摄像机对象和Animator组件的声明和初始化,同时进行了UI界面的初始化，使该案例在任意分辨率屏幕中都可以正常运行。Update方法实现了摄像机对象的实时跟随和朝向。

口第15-28行的主要功能是进行了UIInit和ButtonOnClick方法的开发。UIInit根据当前屏幕的尺寸和分辨率，进行了按钮的位置和大小的初始化。ButtonOnClick方法实现了按钮的回调，当按下任意一个按钮时，系统将向动画控制器发送指令，使其播放相对应对动画。

(7)最后单击运行按钮,其运行效果会出现在Game窗口中,如图10-115所示。场景中的Boy对象挂载了前面通过代码生成的动画控制器,读者点击其中的任意一个按钮之后,场景中的Boy对象将会调用Animator组件中的动画控制器播放指定的动画片段,如图10-116所示。


(1)首先创建一个场景，并命名为“MecanimScene”。然后把Models文件夹中的Boy模型文件拖曳到Scene窗口中，接着再创建一个地形，并命名为“Terrain”。调整灯光朝向，使场景足够明亮。
    
(2)接下来导入一个EasyTouch的插件。双击该插件，导入该插件，再打开Unity，可发现菜单栏多了图10-119所示的按钮。然后依次单击“Hedgehog Team-EasyTouch-Extensions”下的“Adding a new joystick”和“Adding a new button”添加一个虚拟据杆和四个按钮。

(3)在场景创建完成之后，开始进行动画控制器的创建和配置。首先新建一个动画控制器，并命名为“AniController”。该动画控制器将用于对本案例中所有动画的播放控制。

(4)双击该动画控制器，进入动画控制器的编辑窗口，然后将Assets/Animations/FightAnis目录下的“ldle" "Walk" "JumpDodge" "TurnKick " "StepSideKick"以及"CartWheel"等动画拖曳进动画控制器编辑窗口。

(5)然后为动画控制器依次添加"Tigger2SSK " "Trigger2JD" "Trigger2cW" "Trigger2TK""TriggerZExit"Triger walk"和"Trgger2ldle"等触发器类型的动画过渡参数,如图10-123所示。这些参数将分别用于操控动画控制器中各个动画的播放。

(6)然后为动画控制器搭建过渡条件至图10-124所示效果,并为所有过渡条件添加过渡参数。由于篇幅所限,各个过渡条件与参数间的详细搭配关系在此就不再赘述,读者可参考随书光盘/第10章/Mecanimstudy/Assets/AniControllers目录下的"AniController"文件进行配置。

(7)把AniController动画控制器拖曳到Boy对象的Animator组件下面。接下来在Scripts 文件夹中新建一个C#脚本,并命名为"HeroController",该脚本用于实现对动画播放的控制,然后将这个脚本拖曳到Boy对象上。该脚本的详细代码如下所示。
```cs
using UnityEngine;
using System.Collections;

public class HeroController : MonoBehaviour
{
	#region Variables

	private Animator myAnimator;  // 声明Animator组件
	private Transform myCamera;  // 声明摄像机对象
	private EasyJoystick myJoystick;  // 声明摇杆
	private EasyButton[] myButtons = new EasyButton[4]; // 声明游戏按钮
	private string[] triggerStrings  = new string[] { "Trigger2SSK", "Trigger2JD", "Trigger2CW", "Trigger2TK"};  // 声明游戏控制器参数名
	public static bool isWalk;  // 是否正在播放行走动画

	# endregion


	#region StartFunction
	void Start()
	{
		myAnimator = GetComponent<Animator>();  // 初始化Animator组件
		myCamera = GameObject.Find("Main Camera").transform; // 初始化摄像机对象
		myJoystick = GameObject.Find("MyJoystick").GetComponent<EasyJoystick>(); // 初始化摇杆
		for (int i = 0; i < myButtons.Length; i++) // 遍历按钮集合
		{ 
			myButtons[i] = GameObject.Find("Button" + i).GetComponent<EasyButton>(); // 初始化按钮
			
		}
 
	}
	#endregion 

	#region UpdateFunction
	void Update()
	{
		CameraBehaviour(); // 摄像机控制操作
		DirectBehaviour(); // 摇杆响应操作
	}
		
	
	void CameraBehaviour() 
	{
		myCamera.position = transform.localPosition + new Vector3(0, 2, -5); // 摄像机对象跟随
		myCamera.LookAt(transform); // 摄像机对象朝向
	}

	void DirectBehaviour() 
	{
		if (myJoystick.JoystickTouch != Vector2.zero)  // 当摇杆有所触碰时
		{
			if(!isWalk) 
			{
				myAnimator.SetTrigger("Trigger2Walk"); // 传递行走参数
			}
			isWalk = true; // 修改标志位
			transform.LookAt(new Vector3(myJoystick.JoystickTouch.x * 10000, transform.position.y, myJoystick.JoystickTouch.y * 10000)); // 对象朝向设置
		} else {
			if(isWalk)
			{
				myAnimator.SetTrigger("Trigger2Idle");
			}
			isWalk = false;
		}
	}

	void ButtonOnClick(string button)
	{
		myAnimator.SetTrigger(triggerStrings[button.ToCharArray()[button.Length -1] - 48]); // 传递参数
	}
	#endregion
}
```
口第1-20行的主要功能是进行Start方法的开发。在该方法中进行了动画组件、摄像机对象及U1界面的声明和初始化,同时声明一个参数集合,便于在开发过程中对动画播放进行控制。口第21-44行的主要功能是行Update方法的开发。该方法主要调用了CameraBehaviour和DirectBehaviour函数,这两个函数分别进行了摄像机对象跟随的开发和虚拟摇杆的监控,使场景中的Boy对象实时地朝向摇杆所指向的地方。

口第45-49行的主要功能是实现四个UI按钮的监听。当读者按下任意一个按钮时,系统将调用该方法,向动画控制器传递相对应的参数,实现对动画播放的控制。

(8)最后读者可通过单击运行按钮来运行本案例, Game窗口将显示本案例的运行效果,如图10-125所示。当读者操控虚拟摇杆时,场景中的Boy对象将按摇杆指向的方向行走;当读者点击四个按钮中的任意一个时,场景中的Boy对象将执行相对应的动作,如图10-126所示。