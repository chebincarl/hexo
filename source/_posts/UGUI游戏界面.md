---
layout: title
title: UGUI游戏界面
date: 2019-01-31 15:28:36
categories: Unity
tags: Unity3D游戏开发第2版-宣雨松
---
UGUI提供的基础元素包括文本、图片、按钮、滑动条和滚动组件等，配合UI还提供了强大的EventSystem事件系统来管理UI元素。从Unity2017开始还引入了图集的概念，让UGUI的功能更加全面，用起来更灵活。UGUI在自适应处理上显然下了功夫，提供了锚点、布局、对齐方式和Canvas，专门用来解决分辨率不同所带来的自适应屏幕问题。
<!--more-->

# 基础元素
UI是游戏开发中非常重要的一部分，所有交互都需要在其中完成。界面中的元素是多元化的，例如文本、图片和按钮互相组合，搭配排列出完美的界面。UI提供了Rect Transform组件，用来设置锚点以及对齐方式。布局方式对于UI元素都是通用的，它的参数也比之前的Transform多了很多。

## Text 
Hierarchy视图Create->UI->Text，即可创建一个文本组件。如下所示，文本需要使用TTF字体。电脑上可以找到很多TTF字体，大多都会有无用的字符，最好使用FontCreator软件来精简一下字体库。
Text组件提供了横向、纵向自动换行的功能，分为横向与纵向两种。请注意Raycast Target复选框，如果UI元素不需要点击事件，一定不要勾选它。因为UGUI的事件系统会遍历出所有带Raycast Target的组件，这无疑会带来一些不必要的开销。还有UGUI默认的材质我们是无法修改的，但是可以重写它，只需要将新的材质拖入Material处。

warp：包裹 overflow：溢出 raycast：光线投射 emoticons：表情 geometry：几何
{% asset_img 1.png %}

## 描边和阴影

在Text游戏对象上添加Outline和Shadow，即表示支持文本的描边和阴影。可以设置它们的颜色以及描边的距离。
{% asset_img 2.png %}
描边的原理就是在原有Text组件的基础上在上、下、左、右个多画了一遍，所以它的效率是很低的。阴影会比描边好很多，因为它只需多画一遍，所以能用阴影就不要用描边。

## 动态字体

UGUI动态字体的原理是根据传入的字体以及字体的大小生成到一张纹理上，最终将纹理上的字体显示出来。这就带来一个问题：Text中设置了3个完全相同的字体，只是字体的大小不一样，然而在纹理中却生成了3份。所以，游戏中是不太建议使用动态字体的。
在Unity3D中添加字体
1、在Assets中新建文件夹Fonts；
2、在系统盘c盘中找到C:\Windows\Fonts
3、把相应的字体拖入到Fonts即可。
除了聊天和起名等必须由用户自己主动输入的文字外，游戏中大量的文字实际上并不需要使用动态字体，此时可以使用SDF字体。它的原理就是用位图来保存矢量信息，记录到边的最短距离，最后再用Shader还原回来。TextMesh Pro这个插件就使用了这个原理，建议在游戏中使用此插件。

## 字体花屏
UGUI的动态字体会动态生成材质，它开始是256x256（像素），然后根据使用字体的情况慢慢扩大，直到4096x4096（像素）。当文字太多不够放的时候，会触发UGUI内部重建字体贴图命令，接着就可能造成文字花屏了。若要解决这个问题，就要监听它内部重建的事件，然后整理刷新一下当前场景中的所有字体即可。
如下代码所示，监听Font.textureRebuilt字体重建事件，记录当前待重建的字体，在下一帧更新的时候，调用FontTextureChanged()方法刷新所有Text文本。
```cs
using UnityEngine;
using System.Text;
using UnityEngine.UI;

public class Script_05_01 : MonoBehaviour
{
    // 标记某个字体发生了重建
    private Font m_NeedRebuildFont = null;

    void Start()
    {
        // 监听字体贴图重建事件
        Font.textureRebuilt += delegate (Font font)
        {
            m_NeedRebuildFont = font;
        };
    }

    void Update()
    {
        if (m_NeedRebuildFont)
        {
            // 找到当前场景中的所有Text，重新刷新一下
            Text[] texts = GameObject.FindObjectsOfType<Text>();
            if (texts != null)
            {
                foreach (Text text in texts)
                {
                    if (text.font == m_NeedRebuildFont)
                    {
                        text.FontTextureChanged();
                    }
                }
            }
            m_NeedRebuildFont = null;
        }
    }
}
```
这里使用GameObject.FindObjectsOfType<Text>()方法取到当前Hierarchy视图中的所有字体，依次遍历后，重新刷新它即可。

## Image组件
{% asset_img 3.png %}
Image组件用来显示图片。图片一共分为以下4种格式。
* Simple：直接显示图片。
* Sliced：通过九宫格方式显示图片，可用SpriteEditor来编辑九宫格的区域。
* Tiled：平铺图片。
* Filled：像技能CD一样，可以旋转图片。

Preserve Aspect复选框表示是否强制等比例显示图片。单击下方的Set Native Size按钮后，可重新格式化图片大小。

## Raw Image组件
Image组件只能显示Sprite，Raw Image组件既可以显示任意Texture，也可以使用Sprite，不过它还是以Texture方式显示的。Image组件使用Sprite时，可以使用Atlas来合并批次，但是Raw Image组件却不能，每个Raw Image就占一次DrawCall。所以，一般不太建议使用Raw Image组件。
但是有时候又不能不使用Raw Image组件，比如Render Texture，需要将摄像机渲染到纹理中，就必须使用它。如下图所示，Raw Image组件的参数也比较简单，不想Image那样有很多类型，它挂上图片就可以使用了。Raw Image属于原始显示图片的组件，初始化速度也是最快的。但是由于它无法合并DrawCall，所以大量UI系统不建议使用它。
{% asset_img 4.png %}

## Button组件
Button组件必须依赖Image组件。按钮有普通、点击、抬起和悬浮这几种状态，切换各状态可以改变它的颜色，也可以更换Sprite图片样式，再或者使用Animation动画系统来控制各状态。Button组件直接提供了点击方法来监听点击事件。下图显示了Button组件的参数。
{% asset_img 5.png %}
如下代码所示，调用onClick.AddListener()方法就可以给按钮添加监听点击的事件了。
```cs
using UnityEngine;
using UnityEngine.UI;

public class Script_05_02 : MonoBehaviour
{
    public Button button;

	void Start ()
    {
        button.onClick.AddListener(delegate ()
        {
            Debug.Log("click");
        });	
	}
}
```
需要说明的是，按钮不仅可以通过代码来添加点击事件，也可以通过在按钮组件面板中来添加。选择OnClick事件，在下拉框中选择对应的方法名，前提是该方法需要为public属性的公有方法。开发中，在代码中对按钮进行监听更加灵活，不建议在面板中添加点击方法。

## Toggle组件
Toggle组件就像单项选择一样，选择其中一个选项，剩下的会自动取消选择。记得要把所有Toogle对象都关联进同一个ToogleGroup里。下面来监听一下它的选择/取消选择事件。Toogle组件可以使用onValueChanged()方法来监听切换事件。
{% asset_img 6.png %}
```cs
using UnityEngine;
using UnityEngine.UI;

public class Script_05_03 : MonoBehaviour
{
    public Toggle[] toogles;

    private void Start()
    {
        foreach (var toogle in toogles)
        {
            toogle.onValueChanged.AddListener(delegate (bool selected)
            {
                Debug.LogFormat("toogle = {0} selected = {1}", toogle.name, selected);
            });
        }
    }
}
```
如果运行时需要用脚本单独改变其中的某一个，可以使用toogle.isOn来设置它的选中状态。

## Slider组件
Slider组件就是一个滑块在进度条上左右拖动，游戏中经常会使用它来做人物的血条。如下图，滑动Slider组件，可以更改血条的显示方式。
{% asset_img 6.png %}
如下代码所示，使用OnValueChanged()方法监听Slider组件的滚动事件，并且可以取到滚动的进度。
```cs
using UnityEngine;
using UnityEngine.UI;

public class Script_05_04 : MonoBehaviour
{
    public Slider slider;

    private void Start()
    {
    	// 设置取值范围内的最小值/最大值
        slider.minValue = 0;
        slider.maxValue = 100;

        slider.onValueChanged.AddListener(delegate (float value)
        {
            Debug.LogFormat("value = {0}", value);
        });
    }
}
```
如果运行时需要用脚本更改它，则可以使用slider.value来设置新的值。另外，Slider和Toggle都是使用onValueChanged()来监听事件的，但是它们传递的UnityEvent是不同的，并且回调的参数也是不同的。

## Scrollbar & ScrollView组件
Scroll Rect组件由Scrollbar组件（拖动条）和ScrollView组件（滑动区域）组成。它的原理就是使用Scroll Rect组件设置一个滑动区域，然后挂上Scrollbar组件监听滑动的事件，这和Slider组件类似。

## 使用ScrollRect组件制作游戏摇杆
ScrollRect是UGUI提供的基础拖动组件，给它的Content绑定一个滑块就能工作了。利用这个特性来做游戏摇杆再合适不过了。
如下代码所示，继承ScrollRect后，重写OnDrag()方法来监听滑动摇杆事件。其中，contentPosition.magnitude计算滑动摇杆的长度，我们需要让摇杆保持在圆形区域内。
```cs
using UnityEngine;
using UnityEngine.EventSystems;
using UnityEngine.UI;

public class Script : ScrollRect
{
    protected float mRadius = 0f;

    protected override void Start()
    {
        base.Start();
        // 计算摇杆半径
        mRadius = (transform as RectTransform).sizeDelta.x * 0.5f;
    }

    public override void OnDrag(PointerEventData eventData)
    {
        base.OnDrag(eventData);
        var contentPosition = this.content.anchoredPosition;
        if (contentPosition.magnitude > mRadius)
        {
            contentPosition = contentPosition.normalized * mRadius;
            SetContentAnchoredPosition(contentPosition);
        }
    }
}
```
在上述代码中，mRadius表示摇杆滑动圆形区域的半径，contentPosition.normalized表示摇杆的单位向量，两者相乘，即可得出摇杆最终所在的位置。

# 事件系统
UGUI所有的事件系统都是依赖EventSystem组件来完成的，Unity新版的事件系统已经全面代替了之前的SendMessage系统了。操作的事件是非常庞大的。鼠标、键盘和手势能产生太多事件了，而且不同的UI操作事件还不太一样。就拿按钮来说，点击事件、按下事件和抬起事件都各不相同。可想而知，所有的UI系统加在一起能产生的事件将有多少。需要在全局添加一个EventSystem对象，运行起来后，在面板中还可以查看操作的一些详细信息，便于日后调试。
事件系统不仅会抛出点击一类的事件，还可以取到最基本的操作信息，例如鼠标在屏幕中的坐标、滑动开始的坐标以及滑动结束的坐标等。新版的EventSystem不仅供UI使用，3D游戏对象也可以使用它，并且使用方法都比较接近。

## UI事件
UI事件依赖于Graphic（图解的） Raycaster组件，如下图所示，它必须绑定在Canvas组件上，表示这个Canvas下所有UI元素支持的事件。比如游戏中同时有很多Canvas（幕布），如果想让游戏中某些UI不可接收点击事件，那么可以考虑把部分Canvas上的Graphic Raycaster组件设置成enable=false，或者直接删掉Graphic Raycaster组件即可。
{% asset_img 8.png %}

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



## UI事件管理
## UnityAction和UnityEvent
## RaycastTarget优化
## 渗透UI事件

# Canvas组件
## 自适应屏幕
## 锚点对齐方式
# Atlas