---
layout: title
title: Avatar换装系统
date: 2019-04-27 16:12:07
categories: Unity
tags: Unity 3D实战核心技术详解
---
换装系统又称纸娃娃系统，是运用在游戏开发上的关键技术。市面上关于换装的游戏非常多，3D MMORPG游戏中也使用了角色换装技术，所以掌握换装技术是非常重要的。换装系统原理也同样适用于换武器、换表情等。本章先讲换装实现原理，再讲换装的编程实现，最后提醒读者在运用该技术解决问题时应注意的事项。

<!--more-->

# 换装原理

游戏内的角色，能够像纸娃娃换装那样让玩家为自己的角色改变外观，一直深受广大玩家欢迎。一般而言，建好的3D模型，如果要将其中一个部位换成另外一个形状，最直接的做法是将该物件的模型Mesh替换掉，那么外观就改变了，但这种方法如果运用在需要做动作的模型上，被置换的部位可能就不会正常做动作了，更糟的状况是可能连模型显示的位置及方向都是错误的，所以，直接更换Mesh的方法只适用于静态模型物件。为此，必须找出更深入的方法来实现换装的功能。

首先看看模型的结构。在Hierarchy视窗中将物件展开，我们会发现几个名称相同并使用数字区别的物件，它们分別代表人物各部位的模型。由此可知，整个人物包含多个相同部位的模型，其在Unity中的表现如下图所示。对象FemaleAvatar的子节点Famale_Bip01是整个人物的骨架结构，人物的动作组件则设置在顶层对象（FemaleAvatar）的Animation组件中，这个模型是资源模型，而不是实际上要放在场景中的目标物件。模型的制作是需要美术将其呈现出来的，最后交付给程序使用。在这里给读者介绍模型资源结构的主要目的是希望读者了解模型制作要求。

{% asset_img 1.png %}

上图的FemaleAvatar对象中每个部位都有多个模型部件，这些部件就是用来换装的，其在Unity场景中的效果表现如下图所示。

{% asset_img 2.png %}

身体的每个部位是由两部分组成的，这也是换装模型的源文件，将模型作为来源模型资源，再依照需求将资源模型各部位重新组合成一个新的目标模型。接下来开始制作源模型Source和目标模型Target。现将人物模型FemaleAvatar放到场景中，同时再复制一个作为目标模型使用，把它们分别命名为Source和Target，在Unity编辑器的操作如下图所示。

{% asset_img 3.png %}

其中targetmodel模型是将源FemaleAvatar对象中除了Female_Bip01外其他部分都删除掉，将父节点名字改成targetmodel，这样就完成了targetmodel对象的制作，是不是很简单？然后将它们拖入Resources文件夹中作为实例化模型。接下来需要为其设置一个动作，可以循环播放的动作，设置组件参数如下图所示。

{% asset_img 4.png %}

Source物件是作为来源资源使用的，实际在场景中不需要显示。Source中的各部位名称必须要有编号，例如，face-001。为了便于区分换装的各个Mesh部分，如果没有编号，请开发者自行加上编号，这些工作需要程序员和美工事先定义好，完成以上的准备工作，就要开始写程序了。程序员的主要工作是先将Source中每个物件的SkinnedMeshRenderer取出并储存在data表中，data的内容则是根据部位分类索引。接下来在Target中加入SkinnedMeshRenderer，然后在每个部位取出一个指定的Mesh，利用CombineInstance类及Mesh.CombineMeshes()将各部位模型合并，同时也要重新排列材质，依照取出的SkinnedMeshRenderer的bone的名称，找到与Target的Female_Bip01子物件内名称相对应的物件重建骨架列表。最后将这些重新组合建立的资源赋给Target的各个SkinnedMeshRenderer，如此就可完成换装的工作了。原理搞清楚了，下面开始实现具体代码。

# 换装代码实现

以下是完整的换装源代码。
```cs
using UnityEngine;
using System.Collections.Generic;

public class AvatarSys : MonoBehaviour 
{
    // 来源模型资源的物件
    private Transform source;
    // 目标物件
    private Transform target;
    // 实例化的源文件和目标文件
    private GameObject sourceobj;
    private GameObject targetobj;
    // 模型资源
    private Dictionary<string, Dictionary<string, Transform>> data = new Dictionary<string, Dictionary<string, Transform>>();
    // 播放的动作
    private Animation mAnim;
    // 目标物件的骨架
    private Transform[] hips;
    // 目标物件各部位的 SkinnedMeshRenderer（参照）
    private Dictionary<string, SkinnedMeshRenderer> targetSmr = new Dictionary<string, SkinnedMeshRenderer>();
    public static AvatarSys instance;
    // 各部分换装的名字
    string[,] avatarstr = new string[,] { { "coat", "003" }, { "hair", "003" }, { "pant", "003" }, { "hand", "003" }, { "foot", "003" }, { "head", "003" } };
 
    string[,] avatarstr0 = new string[,] { { "coat", "001" }, { "hair", "001" }, { "pant", "001" }, { "hand", "003" }, { "foot", "003" }, { "head", "003" } };
    string[,] avatarstr1 = new string[,] { { "coat", "003" }, { "hair", "001" }, { "pant", "001" }, { "hand", "003" }, { "foot", "001" }, { "head", "001" } };
 
 
    private float pos;

    // 用于初始化
    void Start () 
    {
        instance = this;
        AvatarManager(0.0f);
        AvatarManager0(1.0f);
       //AvatarManager1(2.0f);
    }

    // 创建多个换装模型
    void AvatarManager(float pos)
    {
        InstantiateAvatar();
        InstantiateSkeleton(pos);
 
        LoadAvatarData(source);
        hips = target.GetComponentsInChildren<Transform>();
        Inivatar();
    }
 
    void AvatarManager0(float pos)
    {
        InstantiateAvatar();
        InstantiateSkeleton(pos);
 
        LoadAvatarData(source);
        hips = target.GetComponentsInChildren<Transform>();
        Inivatar0();
    }
 
    void AvatarManager1(float pos)
    {
        InstantiateAvatar();
        InstantiateSkeleton(pos);
 
        LoadAvatarData(source);
        hips = target.GetComponentsInChildren<Transform>();
        Inivatar1();
    }
 
    // 实例化Avatar模型
    void InstantiateAvatar()
    {
        sourceobj = Instantiate(Resources.Load("FemaleAvatar")) as GameObject;
        source = sourceobj.transform;
        sourceobj.SetActive(false);
    }

    // 实例化骨骼动画
    void InstantiateSkeleton(float pos)
    {
        targetobj = Instantiate(Resources.Load("targetmodel")) as GameObject;
        target = targetobj.transform;
        target.transform.position = new Vector3(pos, 0.0f, 0.0f);
    }

    // 加载Avatar数据
    void LoadAvatarData(Transform source)
    {
        data.Clear();
        targetSmr.Clear();
 
        if (source == null)
            return;
        SkinnedMeshRenderer[] parts = source.GetComponentsInChildren<SkinnedMeshRenderer>(true);
        foreach (SkinnedMeshRenderer part in parts)
        {
            string[] partName = part.name.Split('-');
            if(!data.ContainsKey(partName[0]))
            {
                data.Add(partName[0], new Dictionary<string, Transform>());
                GameObject partobj = new GameObject();
                partobj.name = partName[0];
                partobj.transform.parent = target;
 
                targetSmr.Add(partName[0], partobj.AddComponent<SkinnedMeshRenderer>());
            }
            data[partName[0]].Add(partName[1], part.transform);
        }
    }

    // 改变Avatar模型
    public void ChangeMesh(string part, string item)
    {
        SkinnedMeshRenderer smr = data[part][item].GetComponent<SkinnedMeshRenderer>();
 
        List<Transform> bones = new List<Transform>();
        foreach (Transform bone in smr.bones)
        {
            foreach (Transform hip in hips)
            {
                if (hip.name != bone.name)
                {
                    continue;
                }
                bones.Add(hip);
                break;
 
            }
        }
        targetSmr[part].sharedMesh = smr.sharedMesh;
        targetSmr[part].bones = bones.ToArray();
        targetSmr[part].materials = smr.materials;
    }
 
    void Inivatar()
    {
        int nLength = avatarstr.GetLength(0);
        for (int i = 0; i < nLength; i++ )
        {
            ChangeMesh(avatarstr[i, 0], avatarstr[i, 1]);
        }
    }
 
    void Inivatar0()
    {
        int nLength = avatarstr0.GetLength(0);
        for (int i = 0; i < nLength; i++)
        {
            ChangeMesh(avatarstr0[i, 0], avatarstr0[i, 1]);
        }
    }
 
    void Inivatar1()
    {
        int nLength = avatarstr1.GetLength(0);
        for (int i = 0; i < nLength; i++)
        {
            ChangeMesh(avatarstr1[i, 0], avatarstr1[i, 1]);
        }
    }
} 
```
程序将模型资源的存储放到了一个Dictionary字典列表里面，代码如下所示。
```cs
// 模型资源
private Dictionary<string, Dictionary<string, Transform>> data = new Dictionary<string, Dictionary<string, Transform>>();
```
模型Mesh的挂接部位SkinnedMeshRenderer放在如下的列表里面，代码如下所示。
```cs
// 目标物件各部位的 SkinnedMeshRenderer （参照）
private Dictionary<string, SkinnedMeshRenderer> targetSmr = new Dictionary<string, SkinnedMeshRenderer>();
```
下面开始介绍代码的编写思路。我们可以创建多个Avatar模型，创建Avatar模型的函数如下所示。
```cs
// 创建多个换装模型
void AvatarManager(float pos)
{
    InstantiateAvatar();
    InstantiateSkeleton(pos);

    LoadAvatarData(source);
    hips = target.GetComponentsInChildren<Transform>();
    Inivatar();
}
```
该函数调用了多个接口函数，第一个函数InstantiateAvatar()用于创建资源实例，其实就是加载要更换的模型资源，加载完成后将其设置成不可见，代码如下所示。
```cs
// 实例化Avatar模型
void InstantiateAvatar()
{
    sourceobj = Instantiate(Resources.Load("FemaleAvatar")) as GameObject;
    source = sourceobj.transform;
    sourceobj.SetActive(false);
}
```

更换的资源实例化后，需要将它们挂接到骨骼动画上，所以接下来需要实例化出Target目标模型——骨骼动画，也就是函数InstantiateSkeleton（float pos）要做的事情，代码如下所示。
```cs
// 实例化骨骼动画
void InstantiateSkeleton(float pos)
{
    targetobj = Instantiate(Resources.Load("targetmodel")) as GameObject;
    target = targetobj.transform;
    target.transform.position = new Vector3(pos, 0.0f, 0.0f);
}
```
资源和骨骼动画实例化后就要考虑更换Mesh，也就是换装的操作。函数InstantiateAvatar（）只是把资源实例化出来，还没有把各个Mesh取到，这样还不能进行换装，在换装之前还要做的一项工作就是拿到各个部分的Mesh。函数LoadAvatarData（Transform source）就是做这个工作的，代码如下所示。
```cs
// 加载Avatar数据
void LoadAvatarData(Transform source)
{
    data.Clear();
    targetSmr.Clear();

    if (source == null)
        return;
    SkinnedMeshRenderer[] parts = source.GetComponentsInChildren<SkinnedMeshRenderer>(true);
    foreach (SkinnedMeshRenderer part in parts)
    {
        string[] partName = part.name.Split('-');
        if(!data.ContainsKey(partName[0]))
        {
            data.Add(partName[0], new Dictionary<string, Transform>());
            GameObject partobj = new GameObject();
            partobj.name = partName[0];
            partobj.transform.parent = target;

            targetSmr.Add(partName[0], partobj.AddComponent<SkinnedMeshRenderer>());
        }
        data[partName[0]].Add(partName[1], part.transform);
    }
}
```
执行该函数后，就拿到了运用于换装的各个Mesh，下面开始换装了，换装的函数是public void ChangeMesh(string part，string item)，其参数分别表示换装的名字和换装项的名字，类似这个“coat”“03”，函数代码如下所示。
```cs
// 改变Avatar模型
public void ChangeMesh(string part, string item)
{
    SkinnedMeshRenderer smr = data[part][item].GetComponent<SkinnedMeshRenderer>();

    List<Transform> bones = new List<Transform>();
    foreach (Transform bone in smr.bones)
    {
        foreach (Transform hip in hips)
        {
            if (hip.name != bone.name)
            {
                continue;
            }
            bones.Add(hip);
            break;

        }
    }
    targetSmr[part].sharedMesh = smr.sharedMesh;
    targetSmr[part].bones = bones.ToArray();
    targetSmr[part].materials = smr.materials;
}
```
这样整个换装代码的流程就写完了，该脚本可以直接挂接到场景中的某个对象上，该程序是挂接到Camera上面的，如下图所示。

{% asset_img 5.png %}

挂接好脚本后，需要把要实例化的资源模型FemaleAvatar和targetmodel放到Resources资源文件夹下面，主要是便于实例化加载，如下图所示。

{% asset_img 6.png %}

接下来为了便于换装操作，需要做几个UI按钮，搭建一个舞台场景，如下图所示。

{% asset_img 7.png %}

最后需要编写一个脚本用于UI的操作，完整的代码如下所示。
```cs
using UnityEngine;
using System.Collections;
 
public class Avatar_Btn : MonoBehaviour {
 
    int count = 0;
 
    public void OnClick()
    {
        string name = this.gameObject.name;
        switch (name)
        {
            case "coat":
                if (count == 0)
                {
                    AvatarSys.instance.ChangeMesh("coat", "001");
                    count = 1;
                }
                else
                {
                    AvatarSys.instance.ChangeMesh("coat", "003");
                    count = 0;
                }
                break;
            case "hair":
                if (count == 0)
                {
                    AvatarSys.instance.ChangeMesh("hair", "001");
                    count = 1;
                }
                else
                {
                    AvatarSys.instance.ChangeMesh("hair", "003");
                    count = 0;
                }
                break;
            case "hand":
                if (count == 0)
                {
                    AvatarSys.instance.ChangeMesh("hand", "001");
                    count = 1;
                }
                else
                {
                    AvatarSys.instance.ChangeMesh("hand", "003");
                    count = 0;
                }
                break;
            case "head":
                if (count == 0)
                {
                    AvatarSys.instance.ChangeMesh("head", "001");
                    count = 1;
                }
                else
                {
                    AvatarSys.instance.ChangeMesh("head", "003");
                    count = 1;
                }
                break;
            case "pant":
                if (count == 0)
                {
                    AvatarSys.instance.ChangeMesh("pant", "001");
                    count = 1;
                }
                else
                {
                    AvatarSys.instance.ChangeMesh("pant", "003");
                    count = 0;
                }
                break;
            case "foot":
                if (count == 0)
                {
                    AvatarSys.instance.ChangeMesh("foot", "001");
                    count = 1;
                }
                else
                {
                    AvatarSys.instance.ChangeMesh("foot", "003");
                    count = 0;
                }
                break;
        }
    }
}
```
该类实现了接口publicvoid OnClick（）。调用AvatarSys脚本中的ChangeMesh完成换装，将该脚本挂接到UI的每个Button上面，如下图所示。

{% asset_img 8.png %}

完成以上操作后就可以运行游戏了，效果如下图所示。

{% asset_img 9.png %}

角色在运动，单击头发按钮，角色的头发相比于图2-9已经被更换过了。
再单击裤子，裤子已经被更换过了。
再看看换装后运行的资源效果，如下图所示。

{% asset_img 12.png %}

换装后的材质发生了改变，通过上图可以看出，有不同部位的裤子材质。

以上介绍主要是为了帮助读者了解换装所需要做的工作，实际项目开发时，不太可能把游戏中的角色全身各部位的模型资料全部都载入作为来源资料。例如游戏中的武器有100种，角色背包中有3种武器，但为了换装却把100种武器都载入到游戏中，而实际上此角色最多也只能变换背包中的3种武器而已，这样无疑是浪费了97种武器所占用的资源。所以在了解如何换装后，实际操作时应该尽量把来源资源包装起来，只取出需要的资源来进行换装。

#　小结
本章的Avatar换装技术，是针对3D游戏设计的，2D游戏只需要换材质就可以了，比较简单，这里就不介绍了。3D的Avatar换装技术是作为游戏开发者和VR/AR开发者必须要掌握的技能之一。要想掌握Avatar技术主要从两方面入手：一是技术实现，Mesh是绑定到骨骼上的；二是3D MAX建模的要求，或者说是规范，要告诉美工如何建模。希望通过本章的学习，读者可以举一反三做一套更换面部表情的换装系统。