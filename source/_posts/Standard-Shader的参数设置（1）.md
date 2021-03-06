---
layout: title
title: Standard Shader的参数设置（1）
date: 2019-05-22 11:38:19
categories: Unity
tags: 大话Unity2018
---
思考并回答以下问题：
动手调节一下参数的值，尝试给每个属性一个贴图试试看。

<!--more-->

{% asset_img 1.png %}

对于大部分的材质需求，使用Unity内置的Standard着色器就够用了。下面详细讲解一下Standard着色器具体参数的设置

{% asset_img 2.png %}

# Rendering Mode（渲染模式）

Standard Shader中的第一个材质参数是渲染模式。使用这个参数设置材质是否透明以及使用哪种类型的透明模式。

* <span style="color:blue;">Opaque （不透明）</span>- 默认，适用于不需要透明的物体。

{% asset_img 3.png %}
<center>Opaque模式</center>

* <span style="color:blue;">Cutout（裁剪）</span> - 适用于不透明区域和透明区域之间具有明显边缘的透明效果。在这种模式下，<span style="color:red;">没有半透明区域</span>，要么完全不透明，要么完全透明不可见。当需要使用透明通道来决定物体的形状（例如树叶、带孔的布块）时，此功能非常有用。贴图中Alpha值小于Alpha Cutoff属性的区域会被裁剪掉。（所以图片都有Alpha值）

{% asset_img 4.png %}
<center>贴图中alpha小于0.16的部分都会完全不显示，大于0.16的部分会完全显示（无透明度）</center>

* <span style="color:blue;">Transparent（透明）</span> - 适用于渲染逼真的透明材质，如透明塑料或玻璃。在这种模式下，材质会使用透明度值（基于贴图的alpha通道和颜色的alpha），但反射和高光还保持可见，就像真正的透明材质一样，所以即使alpha为0，物体看起来并不是完全透明的。

{% asset_img 5.png %}
<center>物体有半透明的状态，但是由于有反射和高光，所以看起来并不是全透明</center>

* <span style="color:blue;">Fade（淡出）</span> - 透明度可以让物体完全透明，包括任何高光或可能的反射。如果模型需要淡入或淡出，则此模式非常有用。它不适合渲染真实的透明材料，如透明塑料或玻璃，因为反射和高光也会消失。

{% asset_img 6.png %}
<center>全透明</center>

# Albedo（反照率）

{% asset_img 7.png %}
<center>Albedo属性</center>

Albedo参数用来控制物体表面的基本颜色。

有时仅为Albedo参数指定一个颜色就够了，但为Albedo参数指定一张贴图更为常见。这张贴图应该表示模型表面的颜色。需要注意的是，Albedo贴图非常重要的一点是贴图内不应包含任何灯光阴影信息，这样才能基于场景环境渲染出正确的光影效果。

## 透明度

Albedo颜色的Alpha值控制材质的透明程度。仅在渲染模式设置为非Opaque时有用。上文渲染模式中提到了选择正确的透明度模式非常重要，因为它决定了是否能够看到合适的反射和高光效果，以及能否完全透明。

{% asset_img 8.png %}
<center>Transparent模式下不同alpha值的效果</center>

当使用贴图作为Albedo参数的值时，可以通过贴图的Alpha通道来控制材质的透明度。Alpha通道的值对应透明程度，白色完全不透明，黑色完全透明。通过这种方式可以让材质的不同区域有不同的透明度。

{% asset_img 9.png %}
<center>带有RGB通道和Alpha通道的贴图，可以点击RGB / A按钮来切换预览的图像通道</center>

{% asset_img 10.png %}
<center>最终的结果，一个建筑物的破碎的窗户。玻璃中的缝隙完全透明，而玻璃碎片部分透明，窗框完全不透明</center>

# Metallic（金属）

{% asset_img 11.png %}
<center>Metallic属性</center>

<span style="color:red;">Metallic属性不仅适用于真实的金属的材料。</span>这个属性被称为金属，是因为可以控制一个物体的表面的金属程度或非金属程度。

Metallic参数决定了物体表面的像金属的程度。金属度高时，物体表面会更多地反映环境并且其Albedo颜色变得更不明显。在全金属表面上，表面颜色完全由环境反射决定。金属度较低时，Albedo颜色更清晰，并且表面上也可以看到反射。

{% asset_img 12.png %}
<center>Metallic参数从0到1的不同情况（smoothness保持0.8）</center>

默认情况下，如果没有指定贴图，Metallic和Smoothness参数都由滑块控制。这对一些材质已经足够了。但是，如果模型的表面混合了不同的部分，比如一部分是布料，但是一部分是金属纽扣，就可以使用贴图来控制材质表面上的金属度和平滑度。贴图中越亮的部分表示金属感越强，越暗的部分表示金属感越弱。

Metallic参数使用贴图时，Metallic和Smoothness的滑块会自动隐藏。材质的金属度由贴图的R(红色)通道中的值控制，材质的Smoothness属性由贴图的Alpha通道控制（贴图的绿色和蓝色通道会被忽略）。这意味着你可以用单个贴图定义粗糙或光滑的区域和金属或非金属的区域，这对于复杂的模型表面非常有用。例如一张贴图通常包含多个区域，比如皮料、布料、手和脸的皮肤、金属扣等。

{% asset_img 13.png %}
<center>没有Metallic贴图时的皮箱</center>

上图中，皮箱的材质不包含Metallic贴图，整个箱子共用一个metallic值和一个smoothness值，这样效果就不好。理想情况是，箱子的金属扣应该金属感更强。

{% asset_img 14.png %}
<center>使用了Metallic贴图的皮箱</center>

上图中，材质被赋予了一张Metallic贴图。现在金属扣的金属感很强并且光照也有相应的变化。皮革提手比皮箱的皮革表面更光滑，但是它们的“金属感”较低，所以它看起来是有光泽的非金属表面。最右边的贴图可以看出：金属感强的区域是贴图中颜色较浅的区域，而金属感低的区域是贴图中灰黑色区域。

# Smoothness（平滑度）

默认情况下，没有使用Metallic贴图时，Smoothness是由一个滑条控制物体表面有更多的“微观表面细节”还是更光滑。

{% asset_img 15.png %}
<center>smoothness的值从0-1的不同情况</center>

上面提到的“微观表面细节”不是直接能看到的细节，而是在计算光照时参与计算的一个概念。微观表面细节越多，则受到光照时，发生的散射越强。表面越光滑，所有光线都倾向于以可预测和一致的角度反射。极端情况下，非常光滑的表面反射光的方式和镜子一样。较光滑的表面在较宽的角度范围内反射光线（因为光线照到微观表面的凸起），因此反射具有较少的细节，并以更加分散的方式散布在表面上。

{% asset_img 16.png %}
<center>平滑度的低，中和高值（从左到右），作为材质的理论微观表面细节图。黄线代表照射到表面的光线并在不同平滑度下的反射角度。</center>

光滑的表面具有非常低的微观表面细节，或者根本没有，因此光线以均匀的方式反射，产生清晰的反射。粗糙表面在其微观表面细节处凹凸不平，因此光线会以广泛的角度反射出来，这些反射的光线平均后就是漫反射颜色，而且没有明显的反射。

{% asset_img 17.png %}
<center>平滑度的低，中，高值（从上到下）</center>

## 使用Smoothness贴图

与许多其他参数类似，可以指定贴图而不是使用单个滑块值。这样可以更好地控制物体表面不同区域的光滑度。

{% asset_img 18.png %}
<center>Source 选择光滑度的来源</center>

* Metallic Alpha：从Metallic贴图的Alpha通道中获取。贴图中alpha的0-1对应Smoothness的0-1。

* Albedo Alpha：从Albedo贴图的Alpha通道中获取。贴图中alpha的0-1对应Smoothness的0-1。

# Forward Rendering Options

在材质的最下方，有两个特殊的选项，主要是针对Metallic和Smoothness属性。

{% asset_img 19.png %}

* Specular Highlights：是否显示高光效果

* Reflections：是否显示反射效果

# Normal map（法线贴图/凹凸贴图）

{% asset_img 20.png %}

法线贴图是<span style="color:red;">凹凸贴图</span>的一种。它是一种特殊的贴图，可以将表面的细节（如凹凸、凹槽、划痕）添加到受光照影响的模型中，就好像它们是由真实网格渲染的一样。

例如，你可能想飞机外壳上有沟槽、螺钉、铆钉。一种方法是将这些细节建模为几何图形，如下所示。

{% asset_img 21.png %}
<center>通过建模实现的飞机外壳上的细节</center>

将这种微小的细节建模成“真实”的网格通常不是一个好主意。在上图右侧，你可以看到制作单个螺母需要很多个面。在有许多表面细节的大型模型上，这会需要非常多的面。为了避免这种情况，我们应该使用法线贴图来增加复杂表面的细节，并使用较低分辨率的多边形表面来模拟较大的形状。

如果用凹凸映射来实现这些细节，那么表面网格可以变得更简单，并且仅用一张贴图就可以表现出这些细节，现今的图形硬件不用耗费太多资源就可以渲染出来。添加法线贴图后，飞机外壳可以是一个低面模型，但是螺丝、铆钉、凹槽、划痕部分受光照影响后会显得有深度，就像网格实现的一样。

{% asset_img 22.png %}
法线贴图中绘制了螺丝、凹槽、划痕，它会影响光线如何从这个表面反射，给人以3D细节的视觉效果。除铆钉和螺钉外，贴图还能包含更多细节，如细微的凹凸和划痕

在现代游戏开发美术工作流中，美术使用他们的3D建模程序基于高模（高精度、高面数的模型）生成法线贴图，然后将法线贴图映射到对应的低模（低精度，低面数的模型）。低模使用法线贴图可以渲染出高模细节，能大幅提高运行时的性能。

# 总结
今天学习了Standard Shader的几个属性：

* Rendering Mode 渲染模式
* Albedo 反照率
* Metallic 金属度
* Smoothness 平滑度
* Normal Map 法线映射