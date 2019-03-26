---
layout: title
title: GameObject、场景世界以及单例模式
date: 2019-03-08 21:46:47
tags: Unity 
---
Unity中的游戏和世界可通过场景表达，其中场景表示为位于笛卡儿3D坐标系（定义了X、Y、Z轴）内的游戏对象集合。这里，场景单位采用Unity单位进行测算，在实际操作过程中，对应单位表示为米。通过Unity进行脚本设计时，应理解场景和对象的含义，以及对象间的交互方式。也就是说，需要了解场景中的单一、独立对象间的彼此通信方式，以使进程按照希望的方向发展。本章主要讨论本地Unity方法，进而通过优化方式搜索、引用并访问场景对象。除此之外，本章还将对相关方法进行逐一考察，并在具体应用环境中以及性能和效率方面对其予以评估。

<!--more-->

# GameObject对象

通常情况下，GameObject表示场景中的基本单元或实体，一般对应于常见场景中的某项“事物”。这里，具体的环境行为或者游戏中所需的实际对象并不重要，无论何种情形，用户仅需通过GameObject加以实现。GameObject对于游戏体验者而言不一定可见，例如声音、碰撞器以及管理类。另外，某些GameObject处于可见状态，其中包括网格、动画网格、精灵对象等。无论是否可见，GameObject通常在场景中作为相关组件集合予以实例化。实际上，<label style="color:red">组件表示为继承自MonoBehaviour的一个类</label>，并可绑定在场景中的GameObject上，进而调整其行为。各个GameObject至少包含一个组件且不可移除，即Transform组件（对于GUI对象则是RectTransform组件）。该组件记录对象的位置、旋转以及缩放状态。例如，如果在场景中创建了一个空游戏对象（通过应用菜单中的GameObject->Create Empty命令），场景中的新对象仅包含一个Transform组件。相应地，即使GameObject非空，但依然包含了空GameObject对象的组件，Transform组件用于维护其在场景中的物理位置。

当然，GameObject可包含多个组件，对象的行为源自组件间的组合和交互结果。另外，用户还可通过Component菜单向对象添加更多的预置组件。通过将脚本拖曳至对象上，还可添加自定义组件。

因此，GameObject由组件构成。在较高层次上，场景则由单一场景空间内的GameObject集合构成。进一步讲，对象自身彼此间存在着较为重要的关系，并通过层次结构加以定义。同时，对象还可能为其他对象的子对象，反之则称作父对象（即transform.parent）。对于对象的运动和转换方式，这一关系十分重要。简而言之，对象Transform组件的对应值向下级联，并添加至其子对象的转换中。通过该方式，子GameObject相对于其父对象偏移和转换；而父对象位置则表示为子对象位置的原点。然而，如果某一对象未包含父对象，则会相对于世界原点(0, 0, 0)进行转换。

# 组件间的交互方式

如前所述，GameObject表示为组件的集合，关于组件间的交互和通信方式，需要考察其中的组织方式。其中，各个组件可高效地定义为自包含的脚本文件，且独立于其他组件，并与其他组件进行交互。特别地，用户常会访问变量，并在同一GameObject上调用其他组件中的函数，甚至会在每一帧中执行此类操作。本节将讨论这一类交互组件间的通信方式。

一个对象绑定了10个CS文件。这10个文件间如何相互调用彼此的变量和方法？

## SendMessage和BroadcastMessage

一种调用其他组件函数的方法是使用SendMessage和BroadcastMessage，此类函数通常与类型无关。特别地，这一类方法可在脚本中的任意处调用，并通过与同一对象绑定的、其他组件中的名称调用相关方法，且无须考察具体的类型。也就是说，这一类函数并不关注组件的类型，因而便于使用。然而，该方案的问题也较为明显。

缺点：

首先，用户可能会通过其他组件上的名称调用某一函数，抑或根本不会采用这一方法。由于消息发送至全部组件中，因而用户无法对目标组件进行选取（SendMessage:Calls the method named methodName on every MonoBehaviour in this game object）。其次，SendMessage和BroadeastMessage方法在内部依赖于反射机制，因而频繁使用常会导致性能问题，例如在Update事件中调用此类函数，或者OnGUI事件（这将使情况变得更为糟糕）。因此，在实际操作过程中，应寻找相应的替代方案。

## GetComponent函数

如果用户需要直接访问某一对象上的特定单一组件，且已知其数据类型，则可尝试使用GetComponent函数，如示例代码所示。该函数可访问与GameObject绑定的、首个匹配类型的组件。一旦获取其引用，则可像常规对象那样访问该组件，设置、获取其public变量，并调用其中的相关方法。
```cs
using UnityEngine;
using System.Collections;
 
public class MyCustomComponent : MonoBehaviour 
{
    // Reference to transform of object
    private Transform ThisTransform = null;
 
    void Start() 
    {
    	// Get cached reference to transform
        ThisTransform = GetComponent<Transform>(); 
    }
 
    void Update()
     {
        if (ThisTransform != null ) 
        {
            ThisTransform.localPosition += Time.deltaTime * 10.0f * ThisTransform.forward; 
        }
    }
}
```
* 第7行和第12行代码：变量ThisTransform声明为private类型，该变量将赋予与GameObject绑定的、指向Transform组件的引用。该操作将通过函数GetComponent并在Start事件中完成。当访问Transform组件时，还需进一步使用到派生的转换属性，例如“ThisTransform=transform;”。

* 第20行代码：ThisTransform变量直接用于设置GameObject的localPosition。再次强调，对于Transform组件，还可采用transform.localPosition。然而，该方法（transform）将从内部调用额外的函数（属性其实就是一个方法）：成员transform定义为C#属性而非标准变量（如果是变量就直接返回了）。因此，在Start或Awake事件内采用GetComponet函数（获取指向private类变量（即第7行）的组件引用）通常被认为是访问外部组件的一种高效方法，当组件频繁被访问时尤其如此，例如Update函数。

Transform组件定义了两个主要的位置成员，即position和localPosition，并以特定方式调整对象的位置。相对于世界原点，位置成员一般定义了对象在世界空间内的位置。因此，在脚本中设置该变量并不会与实际数字对应
（位于Object Inspector中的Transform组件）。若对象表示为另一个对象的子对象，且后者并未位于世界原点处，则Unity将相对于父对象偏移对象的局部位置,进而将其设置于世界空间中。相比较而言，localPosition成员则直接对应于Object Inspector中Transform组件的位置值。特别地，对象位置表示为相对于父对象位置的偏移值；如果该对象不包含父对象，则表示相对于世界原点的偏移量。此时，成员position和localPosition彼此相等。

## 获取多个组件

有些情况下，用户需要获取列表中的多个组件，即全部组件列表或者仅与特定类型匹配的列表，用户可通过GetComponents函数实现这一操作，如示例代码所示。用户可在单次事件中调用该函数，例如Start和Awake事件，而非Update事件。
```cs
using UnityEngine;
using System.Collections;

public class MyCustomComponent : MonoBehaviour 
{
    // Reference to all components as array
    private Component[] AllComponents = null;
 
    // Use this for initialization
    void Start() {
        // Gets a list of all components attacted to this object
        AllComponents = GetComponents<Component>();
 
        foreach(Component C in AllComponents)
        {
            Debug.Log(C.ToString());
        }
    }
}
```
关于GetComponent和GetComponents函数，Unity提供了相应的变化版本，并使用了对象间的通信机制，而非仅是同一对象中不同组件间的通信。这一类函数包括GetComponentsInChildren（获取子对象中全部组件的累计列表），以及GetComponentsInParent（获取父对象的全部组件）。

## 组件和消息

GetComponent函数族工作良好，可满足组件间通信的大多数要求，若使用得当，则优于SendMessage或BroadcastMessage方法。尽管如此，当给定GameObject后，基于SendMessage调用某一方法时依然十分有效，用户无须事先了解组件的类型。

```cs
using UnityEngine;
using System.Collections;
 
public class MyCustomComponent : MonoBehaviour
{
    // Reference to component on which function must be called
    public MonoBehaviour Handler = null;
 
    //Use this for initialization
    void Start()
    {
        //Call function immediately
        Handler.Invoke("OnSave", 0.0f);
    }
}
```
* 第7行代码：对应类中包含了public引用变量Handler。通过该字段，用户可通过Object Inspector将任意组件拖曳至Handler字段中，并以此表示消息发送的组件。<label style="color:red">需要注意的是，该类表示为MonoBehaviour类型或者继承自MonoBehaviour。这也意味着，用户无须事先了解具体的对象类型。</label>

* 第14行代码：MonoBehaviour的Invoke方法被调用，进而运行名称匹配的对应方法。其中，第二个浮点参数定义了时间值（以秒计），此后，相关函数将被调用。相应地，0表示为即时调用。

# GameObject和场景世界

Unity中的另一项重要任务则是根据脚本搜索场景中的对象，尤其是运行期内实例化的对象，例如玩家对象。场景中的全部敌方对象，这对于相关操作十分重要，其中包括生成敌方角色、拾取能量棒、重定位玩家，以及检测对象间的碰撞。为了获取特定GameObject的引用，Unity提供了一系列与GameObject类相关的函数集。尽管有效，但函数开销较大，因而建议在单次事件中进行调用，例如Start和Awake函数。下面讨论与获取对象协同工作的其他技术和方法。

## 获取GameObject

获取场景中的某一对象可通过GameObject.Find或GameObject.FindObjectWithTag函数实现。其中，出于性能考虑，这里建议使用GameObject.FindObjectWithTag函数。下面首先考察GameObject.Find函数。该函数利用匹配名称搜索首次出现的对象所处的场景。此处，搜索名称应与Hierarchy面板中所呈现的名称匹配。对此，函数执行字符串比较操作并以此确定匹配结果，因而其执行速度较慢且相对烦琐。另外，该函数仅对包含唯一名称的对象有效，而某些时候，情况并非如此。尽管如此，若函数包含了适当的名称，GameObject.Find函数依然十分有效，如下所示。
```cs
//Find Object with the name of player
ObjPlayer = Gameobject.Find("Player");
```
关于GameObject，读者可能会注意到Find函数定义为静态函数，因而无须执行GameObject的实例化操作以调用该函数。用户可通过GameObject.Find在源文件中实现随意调用。本章稍后将对静态和全局作用域予以介绍。

GameObject.Find函数的执行速度较慢。对此，应在单次事件中使用该函数，例如Awake和Start事件。

依据标签进行搜索可视为一种相对高效的方法。场景中的各个对象均包含一个标签成员，默认条件下设置为Untagged。该成员定义为唯一的标识符，用以标记独立对象或多个对象（整体形成一个集合）。总体而言，当采用标签搜索对象时，首先需要将标签显式地赋予某一对象中。用户可在脚本中利用GameObject.tag这一public成员实现这一行为。然而，通常情况下，用户多会使用Unity Editor，即单击Object Inspector中的Tag下拉菜单并选取一个标签。除此之外，用户还可选择Add Tag选项，进而创建新的自定义标签。其中，通用标签包括Player、Enemy、Weapon、Bonus、Prop、Environment、Light、Sound以及GameController等内容。

当对一个或多个对象赋予了相应的标签后，用户可在代码中通过标签高效地搜索对象。对于包含匹配标签的对象，GameObject.FindGameObjectWithTag函数将搜索相应的场景，并返回首个对象（返回一个对象数组），如示例代码所示。需要注意的是，虽然GameObject.FindGameObjectWithTag函数使用字符串参数，但Unity在内部将该字符串转换为数值形式，进而提升标签的比较速度。

```cs
using UnityEngine;
using System.Collections;
 
public class ObjectFinder : MonoBehaviour
{
    // Tag name of objects to find
    public string TagName = "Enemy";
    // Array of found objects matching tag
    public Gameobject[] FoundObjects;
 
    //Use this for initialization
    void Start()
    {
        // Find objects of matching tag
        FoundObjects = Gameobject.FindGameObjectsWithTag(TagName);
    }
}
```
某些时候，用户可能将多个标签赋予单一对象。然而， Unity目前尚不支持此项功能。当然，用户可绕开这一限定条件，将空游戏对象作为主对象的父对象，并将所需的标签赋予各个子对象中。当按照标签搜索对象时，应注意获取指向父对象的索引，即所需的实际对象。

## 对象比较
当针对特定对象搜索场景时，GameObject搜索函数较为有效，但在一些场合下，用户需要对已获取的两个对象进行比较。一般情况下，该操作涉及两个对象间的名称和标签的比较。对此，可通过CompareTag函数实现这一功能，如下所示。   
```cs
// Compares tag of this object with another Obj_Y
boo bMatch = gameObject.CompareTag(Obj_Y.tag);
```
除此之外，用户还需要比较两个对象的相等性，进而确定是否为同一对象，而非简单地判断二者是否具有同一标签。当对决策行为进行编码时，该操作十分重要。例如，在战斗过程中，当判断敌方角色是否投入战斗抑或逃离时，可进一步确定其附近是否存在其他作战单位，并对该角色予以支援。如前所述，用户可通过标签搜索获取场景中的全部敌方对象。对应结果中包含了执行呼救且制订决策的当前角色，因而应将其排除在外。示例代码展示了GetInstanceID的应用方法，如下所示。
```cs
// Find objects of matching tag
FoundObjects = GameObject.FindGameObjectsWithTag(TagName);
// Search through all objects and exclude ourselves
foreach(GameObject O in FoundObjects)
{
	// If two objects are the same
	if (O.GetInstanceID() == gameObiect.GetInstanceID()) continue; // Skip this iteration
	// [...] Do stuff here
}
```

## 获取最近对象
当给定GameObject数组后（可能返回自搜索结果），如何获取场景中线性距离最近的对象？对此，示例代码显示了此类对象的获取方式，其中，使用了Vector3.Distance函数计算场景中两点间的最短距离（以米计）。
```cs
// Returns the nearest game object
GameObject GetNearestGameObject(GameObject Source, GameObject[] DestObjects)
{
	//Assign first object
	GameObject Nearest = DestObjects[0];

	//Shortest distance
	float ShortestDistance = Vector3.Distance(Source.transform.position, DestObjects[0].transform.position);

	// Loop through all objects
	foreach(GameObject Obj in DestObjects)
	{
		// Calculate distance
		float Distance = Vector3.Distance(Source.transform.position, Obj.transform.position);

		// If this is shortest, then update
		if (Distance < ShortestDistance)
		{
			//Is shortest, now update
			Nearest = Obj;
			ShortestDistance = Distance;
		}
	}
	// Return nearest
	return Nearest;
}
```

## 获取特定类型的对象

有时，用户需要使用到场景中特定类型组件的列表，而并不关心所绑定的游戏对象。这一类组件包括敌方角色、所有可收集的对象、转换组件、碰撞器等。基于脚本的实现过程虽然简单但开销较大，如示例代码所示。特别地，通过调用Object.FindObjectsOfType函数，用户可获得特定场景对象实例的完整列表，除非对象处于禁用状态。考虑到该方法的计算开销，应避免在帧事件中对其加以调用；相反，该方法仅可在Start和Awake这一类非频发函数中加以使用，如下所示。
```cs
void Start()
{
	//Get a list of all colliders in the scene
	Collider[] Cols = Object.FindObjectsOfType<Collider>();
}
```

## GameObject之间的路径

当给定两个场景中的GameObject后，例如玩家和敌方角色，通常会测试二者间的路径。也就是说，测试是否存在一个碰撞器，并与两个对象间的虚拟直线相交，如下图所示。这在视见系统中十分有用，同时也用于对象剔除机制中，并以此实现相应的AI功能。

{% asset_img 1.png %}

对此，存在多种方法可实现上述行为，方法之一是使用Physics.LineCast函数，如示例代码所示。
```cs
using UnityEngine;
using System.Collections;
//sample class to determine if a clear line or path exists between two objects
public class ObjectPath : MonoBehaviour 
{
	//Reference to sample enemy object
	public GameObject Enemy = null;

	//Layer mask to limit line detection
	public LayerMask LM;
	//----------------------------------------------------
	// Update is called once per frame
	void Update()
	{
		//Check if clear path between objects
		if(!Physics.Linecast(transform.position, Enemy.transform.position, LM))
		{
			//There is clear path
			Debug.Log("Path clear");
		}
	}
	//----------------------------------------------------
	//Show helper debug line in viewport
	void OnDrawGizmos()
	{
		Gizmos.DrawLine(transform.position, Enemy.transform.position);
	}
	//----------------------------------------------------
}
```
* 第07行代码：示例类与Player绑定；否则，另一个源对象将接收一个public成员变量Enemy，并对其进行路径测试。

* 第10行代码：LayerMask变量定义了一个位掩码，以此表示碰撞测试所用的场景层。

* 第16行代码：Physics.Linecast函数用于确定两个场景对象间是否存在一条连续路径。需要注意的是，如果两个对象均包含碰撞器，例如BoxColliders，则会在碰撞检测过程中对其加以使用。换而言之，对象自身的碰撞器可对LineCast调用产生影响。因此，可使用LayerMask变量包含或排除特定层。

## 访问对象的层次结构
Unity中的Hierarchy面板提供了父子关系的可视化视图，并应用于场景中的全部GameObject中。由于子对象继承了其父对象的转换，因而该关系十分重要。尽管如此，在编辑器中定义并编辑层次结构关系还远远不够，通常还需要在代码中构建对象间的父子关系，遍历特定对象的子节点，进而处理数据或调用其中的功能项。下面首先考察父对象，示例代码显示了如何通过Transform组件将对象X绑定至对象Y上（作为子对象）。
```cs
using UnityEngine;
using System.Collections;

public class Parenter : MonoBehaviour 
{
	// Reference to child object in scene
	private GameObject Child;
	// Reference to parent object in scene
	private GameObject Parent;
	
	// Use this for initialization
	void Start() 
	{
		// Get parent and child objects
		Child = GameObject.Find("Child");
		Parent = GameObject.Find("Parent");

		// Now parent them
		Child.transform.parent = Parent.transform;
	}
}
```
下面讨论如何遍历与父对象绑定的全部子对象。再次强调，该过程通过Transform组件实现，如示例代码所示。
```cs
using UnityEngine;
using System.Collections;

public class CycleChildren : MonoBehaviour 
{
	// Use this for initialization
	void Start() 
	{
		// Cycle though children of this object
		for(int i=0; i < transform.childCount; i++)
		{
			// Print name of child to console
			Debug.Log(transform.GetChild(i).name);
		}
	}
}
```

# 场景、时间和更新操作

Unity中的场景体现了同一3D空间内的有限GameObjet集，并共享同一时间帧。由于动画通常意味着一段时间内的变化效果，因而游戏中需要建立统一的时间概念，进而实现同步动画和变化效果。在Unity中，Time类用于读取时间数据并负责在脚本中的传递操作。因此，对于游戏中可预测的、一致的运动行为，与该类协同工作则是一项十分重要的技能。

游戏中均体现了一定的帧速率，且定义为每秒的帧数（FPS），并可在Game选项卡的Stats面板中进行查看。FPS表示1秒内游戏代码的循环次数，并依据相机将相关内容绘制至屏幕上。其中，每次循环过程称作一帧，帧速率随时间以及不同的计算机设备动态变化，并会受到计算机功耗、运行中的其他处理操作，以及当前帧中的渲染内容等因素的影响。也就是说，随着时间的推移以及设备间的变化，用户难以获取一致的FPS。

{% asset_img 2.png %}

关于帧这一概念的近似描述，Unity提供了MonoBehaviour类可实现的3个类事件，并执行随时间连续更新、变化的各项功能。

* Update：针对场景中处于活动状态的各个GameObjet，Update事件将对其上的活动组件逐帧调用一次。如果对象通过MonoBehaviour.SetActive方法被禁用，则Update事件将不再被该对象调用，直至其再次处于激活状态。简而言之，Update最大程度上准确地描述了Unity中的帧这一概念，因此，对于一段时间内需更新或监视的重复性行为，这一概念十分有用，例如玩家输入事件、键盘事件以及鼠标单击事件。需要注意的是，对于各帧，此处无法保证Update事件在全部组件间的调用顺序。例如，在某一帧内，用户无法保证对象X上的Update函数是否先于对象Y被调用。

* FixedUpdate：类似于Update事件，该事件通常在每帧内被调用多次，但调用模式具有一定的规律，即各次调用间采用了固定的时间间隔。FixedUpdate事件最为常见的应用则是与Unity物理内容协同工作。随着时间变化，如果用户希望更新速度或Rigidbody组件上的属性，则可使用FixedUpdate事件而非Update。

* LateUpdate：类似于Update，该事件在每帧内被调用。然而，LateUpdate通常在Update和FixedUpdate之后被调用，这也表明，当调用LateUpdate时，可确保针对当前帧中的各个对象，Update和FixedUpdate已被调用完毕。因此，LateUpdate常用于更新相机运动，特别是第三人称相机，以使相机可于当前帧内的最新位置尾随对象。

当创建随时间变化的运动行为时，Update，FixedUpdate和LateUpdate的细节内容，以及时间和FPS等概念，对于游戏的编码方式有着重要的指导意义。

## 规则1--帧的重要性

帧通常在每秒内出现多次，否则，游戏将处于不连续状态。在各帧内，Update事件将对场景中各个活动的MonoBehaviour调用一次。在很大程度上，各帧中场景的计算复杂度（以及性能）主要取决于Update事件中的内容。随着功能的增加，其处理时间和负载也将随之增长，对于CPU或GPU皆是如此。对于包含大量对象和组件的场景，如果在Update函数中对代码缺乏有效的规划，并有效地降低载荷量，将极易产生失控状态。因此，应对Update事件以及规则调用的帧事件加以精心设计。简而言之，仅在必要时，例如读取玩家输入或查看鼠标的运动行为，方可将对应代码置于其中。对此，事件驱动编程可有效地降低Update函数中的负载量。

## 规则2--相对于时间的运动

鉴于帧频率无法得到有效保证（帧速率随着时间和不同的计算机设备发生变化），因而需要对运动和变化行为的代码予以精心设计，进而提供一致的游戏体验。下面考察场景中的立方体对象在一段时间内的平滑运动行为。示例代码显示了一种运动行为的构建方法（并非是最佳方案）。
```cs
using UnityEngine;
using System.Collections;

public class Mover : MonoBehaviour 
{
	// Amount to move cube per frame
	public float Speed = 1.0f;

	// Update is called once per frame
	void Update() 
	{
		// Move cube along x axis
		transform.localPosition += new Vector3(AmountToMove, 0, 0);
	}
}
```
上述代码在一定范围内有效，并在各帧或者通过AmountToMove变量移动绑定对象，其问题在于，对应行为与帧速率相关。当前，鉴于帧随时间和计算机设备的不同而发生变化，因而各用户将得到不同的体验结果。特别地，立方体对象将以不同的速度运动。由于无法预测游戏针对特定用户的运行方式，因而情况变得不容乐观。对此，可将运动行为映射至时间上，而非帧。帧处于变化状态，但时间始终保持一致。因此，这里可使用Time类中的deltaTime变量，参见示例代码，即上面代码的修正版本。
```cs
using UnityEngine;
using System.Collections;

public class Mover : MonoBehaviour 
{
	//Speed of cube
	public float Speed = 1.0f;

	// Update is called once per frame
	void Update() 
	{
		//Move cube along forward direction by speed
		transform.localPosition += transform.forward * Speed * Time.deltaTime;
	}
}
```
deltaTime定义为浮点值，表示为自上一次Update函数调用后的时间量（以秒计）。例如，0.5表示自上一帧起时间经历了0.5秒。考虑到deltaTime可用于乘数，因而该值十分有用。在各帧中，当deltaTime与速度相乘时，即可知晓对象的运动距离，即距离=速度x时间。因此，针对对象的运动行为，deltaTime与帧速率无关。

# 永久对象

默认状态下，Unity假设各个对象处于自闭合的时间和场景空间内。场景间的差别类似于各个独立的天体间的差别。最终结果可描述为，对象无法在所属的场景之外存在；同时，当活动场景变化时，对象也将不复存在。由于场景间彼此不同且相互独立，因而这体现了对象的一般行为方式。即使如此，某些对象依然需要被保留，并会在不同场景间被载入，例如玩家角色、积分榜系统或者GameManager类。这一类高优先级对象，其存在不应受到特定场景的限制，并可跨越多个场景。通过DontDestroyOnLoad函数，用户可生成持久性对象，其中包含了某些较为重要的结果。
```cs
using UnityEngine;
using System.Collections;

// This object will survive scene changes
public class PersistentObj : MonoBehaviour 
{
	
	// Use this for initialization
	void Start()
	{
		// Make this object survive
		DontDestroyOnLoad(gameObject);
	}
}

```
场景间的对象持久性十分重要，对象将持有这一特性并在不同场景间运动。这也意味着，子对象中也继承了这一类持久性对象，及其所采用的数据和资源，例如网格、纹理以及声音等。据此，持久性对象可采用轻量级方式构建，例如不包含子对象的空游戏对象，其中仅涉及了可保证正常工作的基本组件，以使场景变化过程中仅保留某些必要、重要的数据。

当改变Unity中的活动场景时，可使用SceneManager.LoadScene函数或SceneManager.LoadSceneAsync。

如前所述，DontDestroyOnLoad函数在活动场景中现有对象上被调用，并防止场景变化时对象被销毁。此时，将会产生与对象复制相关的问题。特别地，如果随后重新载入或者返回至持久对象所处的原场景时，将产生对象的持久特性复制。也就是说，源自上一个场景的原持久性特征，以及针对新创建场景实例的最新对象实例。当然，当每次重新进入场景时，问题还会被再次放大--每次均会产生新的复制，这并非是期望结果。通常情况下，用户仅需要一个对象实例：一名玩家、一个游戏管理器以及一个积分榜。对此，用户需要创建单例对象。

# 理解单例模式

某些类的实例化方式与其他类有所不同，大多数类针对属性集和行为定义了模板，并可在场景中多次实例化为GameObject。例如，敌方角色类可用于实例化多个对象；能量棒类可创建多个对象，等等。然而，某些类则需要作为单一实体存在，例如GameManager、HighScoreManager、AudioManager以及SaveGameManager，并包含统一的行为集合。简单地讲，任何时候只可包含一个类实例。否则，这将混淆对象的含义，并使其有效性大打折扣。此类对象称作单例对象，通常作为持久对象存在于各场景中。单例的核心内容可描述为，任意时刻内存中仅包含一个类实例。下面将在GameManager类中创建单例对象。

在实际操作过程中，各游戏均包含GameManager或GameController类，且均为持久性单例对象。实际上，GameManager负责处理游戏中的全部高层功能项。例如，游戏是否处于暂停状态，是否满足胜利条件，以及任意时刻的决策方式。示例代码显示了GameManager类的部分内容。
```cs
using UnityEngine;
using System.Collections;

// Sample Game Manager class
public class GameManager : MonoBehaviour 
{
	// High score
	public int HighScore = 0;

	// Is game paused
	public bool IsPaused = false;
	
	// Is player input allowed
	public bool InputAllowed = true;
	
	void Start()
	{
		// Make game manager persistent
		DontDestroyOnLoad(gameObject);
	}
}
```
此类对象在场景中持久存在，但如何依此创建单例对象？示例代码实现了这一功能，如下所示。
```cs
using UnityEngine;
using System.Collections;

// Sample Game Manager class - Singleton Object
public class GameManager : MonoBehaviour 
{
	// C# Property to get access to singleton instance
	// Read only - only has get accessor
	public static GameManager Instance
	{
		// return reference to private instance
		get
		{
			return instance;
		}
	}

	private static GameManager instance = null;
	
	// High score
	public int HighScore = 0;

	// Is game paused
	public bool IsPaused = false;
	
	// Is player input allowed
	public bool InputAllowed = true;
	
	// Use this for initialization
	void Awake()
	{
		// Check if any existing instance of the class exists in the scene
		// If so, then destroy this instance
		if(instance)
		{
			DestroyImmediate(gameObject);
			return;
		}

		// Make this active and only instance
		instance = this;

		// Make game manager persistent
		DontDestroyOnLoad(gameObject);
	}
}
```
* 第9-18行代码：private成员被添加至Manager类中，并被声明为static。如果存在多个实例，该变量将在全部实例间被共享。当创建新实例后，可确定内存中是否存在现有的实例。另外，该变量还可通过Instance属性予以访问，且仅包含一个get成员并以此体现其只读特征。

* 第34-41行代码：在Awake事件中（在对象创建时被调用），将对实例变量进行检测，以查看内存中是否已存在现有的类实例。若存在，由于只可存在唯一的类实例，因而需要删除当前对象。这也表明，GameManager在场景变化过程中依然保持了持久性，且场景中仅包含唯一的原对象实例。

在示例代码中，GameManager类使用了Awake函数，而非Start函数。

Start和Awake函数之间的差别在于：Awake函数通常在Start函数之前被调用。Awake函数一般在对象创建时被调用。Start函数则在首帧被调用，其中，GameObject处于激活状态。如果GameObject某个场景下处于禁用状态，则Start函数不会被调用，直至该对象处于活动状态。对于默认条件下处于活动状态的对象，Start函数在场景开始处被调用，且位于Awake事件之后。

如果用户需要将组件引用缓存至类的局部变量中，例如ThisTransform中的Transform组件，则需要使用Awake事件而非Start事件。在Start事件中，可假设对象的全部引用均已处于有效状态。

GameManager的全局静态Instance属性的最大优势在于，可直接访问任何脚本文件，且无须使用到任何局部变量或对象引用。这意味着，各个类可直接访问全部GameManager属性，并调用较高层次的游戏功能。例如，可通过不同的类设置GameManager上的游戏积分变量，如示例代码所示。
```cs
using UnityEngine;
using System.Collections;

public class ScoreSetter : MonoBehaviour 
{
	// Use this for initialization
	void Start() 
	{
		//Set score on GameManager
		GameManager.Instance.HighScore = 100;
	}
}
```

# 本章小结

本章讨论了GameObject、场景、组件及其在场景中的应用方式。初看之下，此类问题较为简单，但理解其应用方法并对对象进行管理则颇具技巧，这一类技术常出现于Unity的游戏开发项目中。特别地，本章着重阐述了GameObject，即以交互方式生成同一行为的组件集。同时，Transform组件同样十分重要。除此之外，本章还介绍了场景内容，即GameObject所处的独立时间和空间。一般情况下，场景表示为一个自封闭的实体，并防止对象位于其外部空间。进一步讲，各场景均涉及时间概念，并以此生成变化和动画效果。其中，时间可采用deltaTime变量进行测算，并可表示为一个乘数因子，以实现与帧速率无关的运动行为。最后，本章还探讨了单例设计模式，通过静态成员定义类--在实际操作过程中，任意时刻内存中仅可包含一个活动实例。