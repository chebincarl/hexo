---
layout: title
title: Unity脚本程序开发
date: 2019-03-12 07:43:41
tags: Unity
---

Unity中的脚本程序如果要起作用，主要途径为将脚本附加到特定的游戏对象上。这样，脚本中不同的方法在特定的情况下会被回调，实现特定的功能。下面给出几个最常用的回调方法。

<!--more-->
* Start：在游戏场景加载时被调用，在该方法内可以写一些游戏场景初始化之类的代码。
* Update：会在每一帧渲染之前被调用，大部分游戏代码在这里执行，除了物理部分的代码。
* FixedUpdate：会在固定的物理时间步调调用一次。这里也是基本物理行为代码执行的地方。

同时，还有一种可以称为方法外部代码的源代码，其在物理加载时运行，还可以用于初始化脚本状态，有点类似于C#里面的成员变量声明。

开发人员还可以重写一些处理特定事件的回调方法，这类方法一般以On前缀开头，如OnCollisionEnter方法（此方法在系统检测到碰撞开始时被回调）等。

其实上述的方法与代码在开发中一般都是位于MonoBehaviour类的子类中的，也就是说开发脚本代码时，主要是继承MonoBehaviour类并重写其中特定的方法。

## 继承自MonoBehaviour类

Unity中所有挂载到游戏对象上的脚本中的类必须继承MonoBehaviour类（直接的或间接的）。MonoBehaviour类中定义了各种回调方法，如Start，Update和FixedUpdate等回调方法。

## 类名字必须匹配文件名
C#脚本中类名需要手动编写，而且类名还必须和文件名相同，否则当脚本挂载到游戏对象时，在控制台会报错。

## 使用Awake或Start方法初始化
用于初始化脚本的代码必须置于Awake或Start方法中。两者的不同之处在于，Awake方法是在加载场景时运行；Start方法是在第一次调用Update或FixedUpdate方法之前被调用；Awake方法运行在所有Start方法之前。

## Unity脚本中协同程序有不同的语法规则
Unity脚本中协同程序（Coroutines）必须是IEnumerator返回类型，并且yield被yield return替代，具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine; //导入系统包
using System.Collections;

public class NewBehaviourScript : MonoBehaviour  // 声明类
{
    IEnumerator SomeCoroutine{ // C#协同程序
        yield return 0; // 等待1帧
        yield return new WaitForSeconds(2); // 等待2s
    }
}
```

## 只有满足特定情况变量才能显示在属性查看器中
只有序列化的成员变量才能显示在属性查看器中，而private和protected类型的成员变量只能在专家模式中显示，而且，其属性不被序列化或显示在属性查看器中。如果属性想在属性查看器中显示，其必须是public类型的。

## 尽量避免使用构造函数

不要在构造函数中初始化任何变量，要用Awake或Start方法来实现。即便是在编辑模式，Unity仍会自动调用构造函数，这通常是在一个脚本编译之后，因为需要调用脚本的构造函数来取回脚本的默认值。无法预计何时调用构造函数，它或许会被预制件或未激活的游戏对象所调用。而在单一模式下使用构造函数可能会导致严重后果，会带来类似随机的空引用异常。因此，如果想实现单一模式就不要用构造函数，要用Awake或Start方法。事实上，没必要在继承自MonoBehaviour类的构造函数中写任何代码。

## 断点

Unity中通过Debug.Break()设置断点，如果想查看特定情况发生时对象属性的变化时，用断点可以快速地完成。

# 常用操作
Unity中很多对游戏对象的操作都是通过在脚本中修改对象的Transform（变换属性）与Rigidbody（刚体属性）参数来实现的，上述属性的参数可以非常方便地通过脚本编程实现修改。例如，让物体绕x轴顺时针旋转20°，则可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    void Update()
    {
        this.transform.Rotate(20, 0, 0);
    }
}
```
脚本开发完成后，将这个脚本挂载到需要旋转的游戏对象上，在项目运行时即可实现所需功能。如果希望游戏对象沿z轴正方向移动，则可以使用如下的C#代码片段来实现。该代码运行时可以实现游戏对象GameObject每帧向前移动1个单位。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    void Update()
    {
        this.transform.Translate(0, 0, 1);
    }
}
```
一般情况下，在Unity中，x轴为红色的轴表示左右，y轴为绿色的轴表示上下，z轴为蓝色的轴表示前后。

用于旋转的Rotate方法和用于移动的Translate方法都有4个参数的重载形式。第四个参数为Space枚举类型，如果设置为“Space.Self”，变换被应用相对于自身轴；如果设置为“Space. World”，
变换被应用相对于世界坐标系统。如果不设置第四个参数，则默认设置为“Space.Self”。

## 记录时间
在Unity中记录时间需要用到Time类。Time类中比较重要的变量为deltaTime（只读），它指的是从最近一次调用Update或者FixedUpdate方法到现在的时间，如果想均匀地旋转一个物体，不考虑帧速率的情况下，可以乘以Time.deltaTime，具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    void Update()
    {
        this.transform.Rotate(10 * Time.deltaTime, 0, 0);
    }
}
```

如果想每秒增加或者减少一个值，需要乘以Time.delaTime，同时也要明确在游戏中是需要每秒1个单位还是每帧1个单位的效果。如果是乘以Time.deltaTime，那么，游戏对象就会按固定的节奏运动而不是依赖游戏的帧速率，因此，游戏对象的运动变得更容易控制。

例如，想让游戏对象沿y轴正方向每秒上升5个单位，具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    public GameObject gameObject; // 声明游戏对象

    void Update()
    {
        Vector3 te = gameObject.transform.position; // 获取游戏对象的位置坐标
        te.y += 5 * Time.deltaTime; // 沿y轴每秒上升5个单位
        gameObject.transform.position = te; // 设置游戏对象的位置坐标
    } 
}
```
如果涉及刚体时，可以写在FixedUpdate方法里面。在FixedUpdate方法里面，如果想每秒增加或者减少一个值，需要乘以TimefixedDeltaTime。例如，想让刚体沿y轴正方向每秒上升5个单位，具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    public GameObject gameObject; // 声明游戏对象

    void FixedUpdate()
    {
        Vector3 te = gameObject.GetComponent<Rigidbody>().transform.position; // 获取刚体的位置坐标
        te.y += 5 * Time.fixedDeltaTime; // 刚体沿y轴每秒上升5个单位
        gameObject.GetComponent<Rigidbody>().transform.position = te; // 设置刚体的位置坐标
    } 
}
```

FixedUpdate方法是按固定的物理时间被系统回调执行的，其中代码的执行和游戏的帧速率无关。

## 访问游戏对象组件
组件属于游戏对象，比如把一个Renderer（渲染器）组件附加到游戏对象上，可以使游戏对象显示到游戏场景中；把Camera（摄像机）组件附加到游戏对象上可以使该对象具有摄像机的所有属性。由于所有的脚本都是组件，因此一般的脚本都可以附加到游戏对象上。

常用的组件可以通过简单的成员变量取得。下面介绍了一些常见的成员变量，如下表所示。
|  组件名称 | 变量名称  |
| :------------ | :------------ |
| Transform  | transform  |
| Rigidbody  | rigidbody  |
| Renderer  |  renderer |
| Camera  | camera（只在摄像机对象有效） |
| Light  | light（只在光源对象有效） |
| Animation  | animation  |
| Collider  | collider  |

这里的组件体现在属性查看器上，而变量是在脚本中体现的。一个游戏对象的所有组件及其所带的属性参数都能够在属性查看器中查看。如果想通过挂载在游戏对象上的脚本代码来实现获得该游戏对象上的对应组件及其属性，可以通过变量名来获得。

如果想查看所有的预定义成员变量，可以查看关于Component、Behaviour和MonoBehaviour类的文档。如果游戏对象中没有想要取得的值，那么上面的变量将为null。在Unity中，附加到游戏对象上的组件可以通过GetComponent方法获得，具体操作时可以使用如下的C#代码片段来实现。代码中第4行和第5行代码功能是一样的，都是使游戏对象沿x轴正方向移动，而第5行代码通过获取Transform组件来使游戏对象移动。

```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    void Update()
    {
        transform.Translate(1, 0, 0); // 沿x轴移动一个单位
        GetComponent<Transform>().Translate(1, 0, 0); // 沿x轴移动一个单位
    } 
}
```
同样地，也可以通过GetComponent方法获取其他的脚本。比如有一个HelloWorld的脚本，里面有一个sayHello方法。HelloWorld脚本要与调用它的脚本附加在同一游戏对象上，具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    void Update()
    {
        HelloWorld helloWorld = GetComponent<HelloWorld>(); // 获取"HelloWorld"脚本组件
        helloWorld.sayHello();
    } 
}
```

在C#代码中只有public类型的变量和方法才能在所有其他类中使用，private类型的变量和方法只能在自身类中使用，protected类型的变量和方法只能在子类和同命名空间下的类中使用，而不写类型的变量和方法只能在同命名空间下的类中使用。

## 访问其他游戏对象

大部分脚本不单单控制其附加到的游戏对象。Unity脚本中有很多方法访问其他的游戏对象和游戏组件，可以通过属性查看器指定参数的方法来获取游戏对象，也可以通过Find()方法来获取游戏对象。下面将对这几种方法进行详细介绍。

### 通过属性查看器指定参数

代码中声明public类型的游戏对象引用，在属性查看器中就会显示这个游戏对象参数，然后就可以将需要获取的游戏对象拖曳到属性查看器的相关参数位置，具体操作时可以使用如下的C#代码片段来实现。代码获取游戏对象上的{“Test”脚本组件，然后执行doSomething方法。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{   
    public GameObject otherObject;

    void Update()
    {
        Test test = otherObject.GetComponent<Test>(); // 获取“Test”脚本组件
        test.doSomething(); // 执行doSomething方法
    } 
}
```

### 确定对象的层次关系
游戏对象在游戏组成对象列表中存在父子关系，在代码中可以通过获取Transform组件来找到子对象或者父对象。具体操作时可以使用如下的C#代码片段来获取游戏对象的子对象和父对象。

```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{   
    void Update()
    {
        transform.Find("hand").Translate(0, 0, 1); // 其沿z轴每帧移动1个单位
        transform.parent.Translate(0, 0, 1); // 每帧移动1个单位
    } 
}
```
一旦读者成功获取到“hand”子对象，就可以通过GetComponent方法获取“hand”对象的其他组件。例如，有一个Test脚本挂载在子对象“hand”上，具体操作时可以使用如下的C#代码来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{   
    void Update()
    {
        transform.Find("hand").GetComponent<Test>().a = 2; // 找到子对象“hand”，同时设置“Test”脚本中的变量a为2
        transform.Find("hand").GetComponent<Test>().doSomething(); // 执行doSomething方法
        transform.Find("hand").GetComponent<Rigidbody>().AddForce(0, 0, 2); // 为“hand”子对象的刚体属性上加一个沿z轴的大小为2的力
    } 
}
```

也可以使用脚本来循环获取到所有的子对象，然后对子对象做某种操作，如平移、旋转等。具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{   
    void Update()
    {
        foreach(Transform child in transform) // 循环获取所有的子对象
        {
            child.Translate(0, 5, 0); // 沿y轴每帧移动5个单位
        }
    } 
}
```
### 通过名字或标签获取游戏对象
Unity脚本中可以使用FindWithTag方法和Find方法来获取游戏对象，FindWithTag方法获取指定标签的游戏对象，Find方法获取指定名字的游戏对象。具体操作时可以使用如下的C#代码片段。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{   
    void Start()
    {
    	GameObject name = GameObject.Find("somename"); // 获取名称为“somename”的游戏对象
    	name.GetComponent<Test>().doSomething(); 
    	name.transform.Translate(0, 0, 1); // 沿z轴平移

    	GameObject tag = GameObject.FindWithTag("sometag"); // 获取标签为“sometag”的游戏对象
    	tag.transform.Translate(0, 0, 1); // 沿z轴平移
    	tag.GetComponent<Test>().doSomething();
    }
}
```
这样，通过GetComponent方法就能得到指定游戏对象上的任意脚本或组件。

### 通过传递参数来获取游戏对象
一些事件回调方法的参数中包含了特殊的游戏对象或组件信息，例如触发碰撞事件的Collider组件。在OnTiggerStay方法的参数中有一个碰撞体参数，通过这个参数能得到碰撞的刚体。具体代码如下。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
	void OnTriggerStay(Collider other) // 重写OnTriggerStay方法
	{
		if (other.GetComponent<Rigidbody>()) // 如果该游戏对象上有刚体组件
		{
			other.GetComponent<Rigidbody>().AddForce(0, 0, 2); // 给刚体施加一个力
		}

		if (other.GetComponent<Test>()) // 如果该游戏对象上有“Test”脚本组件
		{
			other.GetComponent<Test>().doSomething();
		}

	}
}
```
### 通过组件名称获取游戏对象
Unity脚本中可以通过FindObjectsOfType方法和FindObjectOfType方法来找到挂载特定类型组件的游戏对象。FindObjectsOfType方法可以获取所有挂载指定类型组件的游戏对象，而FindObjectOfType方法获取挂载指定类型组件的第一个游戏对象。具体操作时可以使用如下的C#代码来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    void Start() // 重写Start方法
    {
        Test test = FindObjectOfType<Test>(); // 获取第一个找到的“Test”组件
        Debug.Log(test.gameObject.name); // 打印挂载“Test”组件的第一个游戏对象的名称

        Test[] tests = FindObjectsOfType<Test>(); // 获取所有的“Test”组件
        foreach( Test te in tests)
        {
            Debug.Log(te.gameObject.name); // 打印挂载“Test”组件所有的游戏对象的名称
        }
    }
}
```

### 向量
3D游戏开发中经常需要用到向量和向量的运算，Unity中提供了完整的向量以及向量操纵方法,分别为表示二维向量的Vector2类和表示三维向量的Vector3类,因为二维向量和三维向量的使用方法相同,下面将以三维向量为例详细介绍Unity中向量的使用方法。

Vector3类可以在实例化时进行赋值，也可以实例化后给x、y、z分别进行赋值。具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    public Vector3 position1 = new Vector3(); // 实例化Vector3
    public Vector3 position2 = new Vector3(1, 2, 2); // 实例化Vector3并赋值

    void Start() // 重写Start方法
    {
        position1.x = 1; 
        position1.y = 2;
        position1.z = 2;
    }
}
```

Vector3类中也定义了一些常量，例如Vector.up等同于Vector(0, 1, 0)，这样可以简化代码。这些常量对应的值如下表所示。

| 常量  | 值  |
| :------------ | :------------ |
| Vector3.zero  | Vector(0,0,0)  |
| Vector3.one  | Vector(1,1,1)  |
| Vector3.forward  | Vector(0,0,1)  |
| Vector3.up  | Vector(0,1,0)  |
| Vector3.right  | Vector(1,0,0)  |

Vector3类中有很多对向量进行操纵的方法，例如想要获得两点之间的距离时，可以使用Distance方法来完成。这些方法的具体作用如下表所示。
| 方法  | 作用 |
| :------------ | :------------ |
| Lerp  | 两个向量之间的线性插值  |
| Slerp  | 在两个向量之间进行球形插值  |
| OrthoNormalize  | 使向量规范化并且彼此相互垂直  |
| MoveTowards  | 从当前的位置移向目标  |
| RotateTowards  | 当前的向量转向目标  |
| SmoothDamp  | 随着时间的推移，逐渐改变一个向量朝向预期的目标  |
| Scale  | 两个矢量组件对应相乘  |
| Cross  | 两个向量的交叉乘积  |
| Reflect | 沿着法线反射向量  |
| Dot  | 两个向量的点乘积  |
| Project  | 投影一个向量到另一个向量  |
| Angle  | 返回两个向量的夹角  |
| Distance  |  返回两点之间的距离 |
| ClampMagnitude  | 返回向量的长度，最大不超过maxLength所指示的长度 |
| Min  | 返回两个向量中长度较小的向量  |
| Max  | 返回两个向量中长度较大的向量  |
| operator +  | 两个向量相加  |
| operaton -  | 两个向量相减  |
| operatos *  | 两个向量相乘  |
| operator /  | 两个向量相除  |
| operator ==  | 两个向量是否相等  |
| operastor !=  | 两个向量是否不相等  |

### 成员变量和静态成员变量
一般情况下，定义在方法体外的变量是成员变量，如果这个变量为public类型的，就可以在属性查看器看到，若在属性查看器对它的值进行修改，它的值就会随着项目一起自动保存。C#脚本如下。
```cs
public int a = 1;
```
可以在属性查看器中看到这个变量，名字为“a”，它默认显示的值为“1”，读者可以随时在属性查看器中修改它的值。

如果声明的是一个组件类型的变量（类似GameObject、Transform、Rigidbody等），需要在属性查看器中拖曳游戏对象到变量处并确定它的值。具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    public Transform ren; // 声明一个Transform组件

    void Update() 
    {
        if (Vector3.Distance(ren.position, transform.position) < 10)  // 如果ren和transform的距离小于10
        {
            Debug.Log(ren.position);
        }
    }
}
```

可以通过private关键字创建私有变量，此时在属性查看器中就不会显示该变量，避免其被错误地修改，具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    private Collider collider; // 声明私有的collider组件
    void OnCollisionEnter(Collision, collisionInfo) // 重写OnCollisionEnter方法
    {
        collider = collisionInfo.collider; // 获取Collider组件
    }
}
```
在C#脚本中可以通过static关键字来创建全局变量，这样就可以在不同脚本间调用这个变量。具体操作时可以使用如下的C#代码片段来实现。
```cs
public static int test;
```
如果想从另外一个脚本中调用变量“Test”，读者可以通过“脚本名.变量名”的方法来调用。具体操作时可以使用如下的C#代码片段来实现。
```cs
using UnityEngine;
using System.Collections;

public class HelloWorld : MonoBehaviour
{
    void Start()
    {
        Test.test = 1; // 为“Test”脚本中的“test”变量赋值
    }
}
```
### 实例化游戏对象

Unity中如果想创建游戏对象，可以通过创建游戏对象菜单在场景中创建游戏对象（这些游戏对象在场景加载的时候被创建出来），也可以在脚本中动态地创建游戏对象。在游戏运行的过程中，根据需要在脚本中实例化游戏对象的这些方法更加灵活。

Unity中如果想创建很多相同的物体（如射击出去的子弹、保龄球瓶等）时，可以通过实例化（Instantiate）快速实现，而且实例化出来的游戏对象包含了这个对象所有的属性，就能保证原封不动地创建所需的对象。实例化在Unity中有很多用途，充分地使用它非常必要。

例如，创建一个脚本“Hit.cs”，该脚本的功能为当一个碰撞体撞击到一个物体时，销毁这个物体，并在原来的位置实例化一个损坏的物体。该脚本的代码如下所示。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    public GameObject explosion; // 声明游戏对象引用

    void OnCollisionEnter() // 重写OnCollisionEnter方法
    {
        Destroy(gameObject, 1); // 撞击发生1s后销毁对象
        GameObject theClonedExplosion = Instantiate(explosion, transform.position, transform.rotation) as GameObject; // 在物体原来的位置实例化一个损坏的物体
    }
}
```
Destroy(gameObject,n)方法是在ns后销毁物体。如果想立刻销毁物体可以使用DestroyImmediate(gameObject.boolean)，如果参数的布尔值为true，就会立刻销毁物体。

### 协同程序和中断

协同程序，即在主程序运行时同时开启另一段逻辑处理，来协同当前程序的执行，但它与多线程程序不同，所有的协同程序都是在主线程中运行的，它还是一个单线程程序。在Unity中可以通过StartCoroutine方法来启动一个协同程序。

StartCoroutine方法为MonoBehaviour类中的一个方法，也就是说该方法必须在MonoBehaviour或维承于MonoBehaviour的类中调用。StartCoroutine方法可以使用返回值作为IEnumerator类型方法的参数。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(doThing());  // 开启协同程序
    }

    IEnumerator doThing() // 声明doThing方法
    {
        Debug.Log("dothing"); 
        yield return null;
    }
}
```

协同程序中可以使用yield关键字来中断协同程序，也可以使用WaitForSeconds类的实例化对象让协同程序休眠。
```cs
yield return new WaitForSeconds(2);
```

可以将多个协同程序进行连接，创建一个脚本，该脚本功能为在Start方法开启doThing1协同程序，在doThing1协同程序中开启并等待执行doThing2协同程序，doThing2协同程序休眠2s然后打印“doThing2”提示信息，doThing2协同程序执行完后返回doThing1协同程序然后打印“doThing1”提示信息，具体代码如下。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(doThing1()); // 开启doThing1协同程序
    }

    IEnumerator doThing1() // 声明doThing1方法
    {
        yield return StartCoroutine(doThing2); // 开启doThing2协同程序
        Debug.Log("dothing1"); 
        
    }

    IEnumerator doThing2() // 声明doThing2方法
    {
        yield return new WaitForSeconds(2); // 协同程序休眠2s
        Debug.Log("dothing2"); 
    }
}
```
### 一些重要的类

#### MonoBehaviour类
MonoBehaviour类是每个脚本的基类，其继承自Behaviour类。在C#脚本中，必须直接或间接地继承MonoBehaviour类，MonoBehaviour类中的一些方法可以重写，这些方法会被系统在固定的时间回调，常用的可以重写的方法如下表所示。
| 方法  | 说明  |
| :------------ | :------------ |
| Update  | 当脚本启用后，该方法在每一帧被调用  |
| FixedUpdate  | 当脚本启用后，这个方法会在固定的物理时间步调调用一次  |
| Awake  | 当一个脚本实例被载入时该方法被调用  |
| Start  | 该方法仅在Update方法第一次被调用前调用 |
| OnCollisionEnter  | 当刚体撞击碰撞体或碰撞体撞击刚体时该方法被调用  |
| OnEnable  | 当对象变为可用或激活状态时该方法被调用  |
| OnDisable  | 当对象变为不可用或非激活状态时该方法被调用  |
| OnDestroy  | 当对象被销毁时该方法被调用  |
| OnGUI  | 渲染和处理GUI事件时调用  |

MonoBehaviour类中有许多可以被子类继承的成员变量，这些成员变量可以在脚本中直接使用，常用的可继承的成员变量如下表所示。
| 成员变量  | 说明  |
| :------------ | :------------ |
| enabled  | 启用行为被更新，禁用行为不更新 |
| transform  | 附加到游戏物体的Transform组件（如无附加则为空）  |
| rigidbody  | 附加到游戏物体的Rigidbody组件（如无謝加则为空）  |
| camera  | 附加到游戏物体的Camera组件（如无附加则为空）  |
| light  | 附加到游戏物体的Light组件（如无附加则为空)  |
| animation  | 附加到游戏物体的Animation组件（如无附加则为空）  |
| constantForce  | 附加到游戏物体的ConstantForce组件（如无附加则为空）  |
| renderer  | 附加到游戏物体的Renderer组件（如无附加则为空）  |
| audio Source  | 附加到游戏物体的AndioSource组件（如无附加则为空）  |
| guiText  | 附加到游戏物体的GUIText组件（如无附加则为空）  |
| collider  | 附加到游戏物体的Collider组件（如无附加则为空） |
| particleEmitter  | 附加到游戏物体的ParticleEmitter组件（如无附加则为空） |
| gameObject  | 组件附加的游戏物体，一个组件总是被附加到一个游戏物体  |
| tag  | 游戏物体的标签  |

MonoBehaviour类中有许多可以被子类继承的成员方法，这些成员方法可以直接在子类中使用。常用的可继承的成员方法如下表所示。

| 成员方法  | 说明  |
| :------------ | :------------ |
| GetComnonent  | 返阿游戏物体上指定名称的组作  |
| GetComponentInChildren  | 返回游戏对象及其子对象上指定类型的第一个找到的组件  |
| GetComponents  | 返回游戏物体上指定名称的全部组件  |
| SendMessage  | 在游戏物体每一个脚本上调用指定名称的方法  |
| Instantiate  | 实例化游戏对象  |
| Destroy  | 删除一个游戏物体、组件或资源  |
| DestroyImmediate  | 立即销效物体 |
| FindObjectsOfType  | 返回指定类型的所有激活的加载的物体列表  |
| FindObjectOType  | 返同指定类型第一个激活的加载的物体  |

#### Transform类
场景中的每一个物体都有一个“Transform”组件，它就是Transform类实例化的对象，用于储存并操控物体的位置、旋转和缩放，每一个Transform可以有一个父级，允许分层次应用位置、旋转和缩放。可以在“Hierarchy”面板查看层次关系。Transform类中包含了很多的成员变量，常用的成员变量如下表所示。
| 成员方法  | 说明  |
| :------------ | :------------ |
| position  | 在世界空间坐标中游戏对象的位置  |
| localPosition  | 相对于父级的变换的位置  |
| eulerAngles  | 物体旋转的欧拉角  |
| localEulerAngles  | 相对于父级旋转的欧拉角  |
| right  | 在世界空间坐标变换的红色轴。也就是x轴  |
| up  | 在世界空间坐标变换的绿色轴。也就是y轴  |
| forward  | 在世界空间坐标变换的蓝色轴。也就是z轴  |
| rotation  | 在世界空间坐标物体变换的旋转角度  |
| localRotation  | 物体变换的旋转角度相对于父级的物体变换的旋转角度  |
| localScale  | 相对于父级物体变换的缩放  |
| parent  | 物体变换的父级 |
| worldToLocalMatrix  | 从世界坐标转为自身坐标的矩阵变换（只读）  |
| localToWorldMatrix  | 从自身坐标转为世界坐标的矩阵变换（只读）  |
| childCount  | 变换的子物体数量  |
| lossyScale  | 物体的全局缩放（只读）  |

Transform类中也包含了很多的成员方法。常用的成员方法如下表所示。
| 成员方法  | 说明  |
| :------------ | :------------ |
| Translate  | 移动游戏对象的方向和距离  |
| Rotate  | 应用一个欧拉角的旋转角度  |
| RotateAround  | 按照指定角度通过在世界坐标轴旋转物体  |
| LookAt  | 旋转物体，这样指向目标的当前位置  |
| TransformDirection  | 从自身坐标到世界坐标变换方向  |
| InverseTransformDirection  | 变换方向从世界坐标到自身坐标  |
| TransformPoint  | 变换位置从自身坐标到世界坐标  |
| InverseTransformPoint  | 变换位置从世界坐标到自身坐标  |
| DetachChildren  | 所有子物体解除父子关系  |
| IsChildOf  | 这个变换是否是父级的子物体  |
#### Rigidbody类
“Rigidbody”组件可以模拟物体在物理效果下的状态，它就是Rigidbody类实例化的对象，它可以让物体接受力和扭矩，让物体相对真实地移动，如果一个物体想被重力所约束，其必须含有Rigidbody组件。
Rigidbody类中包含了很多的成员变量，常用的成员变量如下表所示。

| 成员变量  | 说明  |
| :------------ | :------------ |
| velocity  | 刚体的速度向量  |
| angularVelocity  | 刚体的角速度向量  |
| drag  | 物体的阻力  |
| angularDrag  | 物体的角阻力  |
| mass  | 刚体的质量  |
| useGravity  | 控制重力是否影响整个刚体  |
| isKinematic  | 控制物理是否影响这个刚体 |
| freezeRotation  | 控制物理是否改变物体的旋转  |
| collisionDetectionMode  | 刚体的碰撞检测模式  |
| centerOfMass  | 相对于变换原点的重心  |
| worldCenterOfMass  | 在世界坐标空间的刚体的重心（只读）  |
| inertiaTensorRotation  | 惯性张量的旋转  |
| inertiaTensor  | 相对于重心的质量的惯性张量对角线  |
| detectCollisions  |  碰撞检测应否启用（默认总是启用的） |
| useConeFriction  |  用于该刚体的锥形摩擦力 |
| position  | 该刚体的位置  |
| rotation  | 该刚体的旋转  |
| interpolation  | 插值允许你以固定的帧率平滑物理运行效果  |
| solverIterationCount  | 允许覆盖每个刚体的求解迭代次数  |
| sleepVelocity  | 线性速度，低于该值的物体将开始休眠  |
| sleepAngularVelocity  | 角速度，低于该值的物体将开始休眠  |
| MaxAngularVelocity  | 刚体的最大角速度  |


Rigidbody类中也包含了很多的成员方法，常用的成员方法如下表所示。

| 成员方法  | 说明  |
| :------------ | :------------ |
| SetDensity  | 基于附加的碰撞器假设一个固定的密度设置质量  |
| AddForce | 施加一个力到刚体  |
| AddRelativeForce  | 施加一个力到刚体，相对于自身的系统坐标  |
| AddTorque  | 施加一个力矩到刚体  |
| AddRelativeTorque  | 施加一个力矩到刚体，相对于自身的系统坐标  |
| AddForceAtPosition  | 在指定位置施加一个力  |
| AddExplosionForce  | 施加一个力到刚体来模拟爆炸效果，爆炸力将随着到刚体的距离线性衰减  |
| ClosestPointOnBounds  | 到附加的碰撞器包围盒上的最近点 |
| GetRelativePointVelocity  | 相对于刚体在指定点的速度  |
| GetPointVelocity  | 刚体在世界坐标空间中指定点的速度  |
| MovePosition  | 移动刚体到指定位置  |
| MoveRotation  | 旋转刚体到指定角度  |
| Sleep  | 强制一个刚体休眠至少一帧  |
| IsSleeping  | 判断刚体是否在休眠  |
| WakeUp  | 强制唤解在休眠状态中的刚体  |

#### CharacterController类

角色控制器是CharacterController类的实例化对象，用于第三人称或第一人称游戏角色控制。它可以根据碰撞检测判断是否能够移动，而不必添加刚体和碰撞器。而且角色控制器不会受到力的影响。 CharacterController 类包含了很多的成员变量，常用的成员变量如下表所示。

| 成员变量  | 说明  |
| :------------ | :------------ |
| isGrounded  | 角色控制器是否触碰地面  |
| velocity  | 角色控制器当前的相对速度  |
| collisionFlags  | 在最近一次角色控制器移动方法调用时，角色控制器的哪个部分与周围环境相碰撞  |
| radius  | 角色控制器的半径  |
| height  | 角色控制器的高度  |
| center  | 角色控制器的中心位置  |
| slopeLimit  | 角色控制器的坡度度数限制  |
| stepOffset  | 角色控制器的台阶偏移量（台阶高度） |
| detectCollisions  | 其他的刚体和角色控制器是否能够与本角色控制器相碰撞  |

CharacterController类中也包含了很多的成员方法，常用的成员方法如下表所示。
| 成员方法  | 说明  |
| :------------ | :------------ |
| SimpleMove  | 以一定的速度移动角色  |
| Move  | 一个更加复杂的移动函数，每次都绝对移动  |

## 性能优化
Unity本身已经针对各个平台在功能上进行了大量的优化，保证了程序的顺利运行，但在使用Unity开发软件的过程中，培养良好的开发习惯，积累编程技巧，对开发人员是至关重要的。良好的开发习惯不仅能帮助开发人员编写健康的程序，还能达到事半功倍的效果。下面将介绍一些针对Unity开发的优化措施。
1.缓存组件查询
当通过GetComponent获取一个组件时，Unity必须从游戏物体里查找目标组件，如果是在Update方法中进行查找，就会影响运行速度。可以设置一个私有变量去储存这个组件，这样，Unity无需在每一帧中去查询组件。实现方法可以参考如下代码片段。
```cs
using UnityEngine;
using System.Collections;

public class NewBehaviourScript : MonoBehaviour
{
    private Transform m_transform;
    
    void Start()
    {
        m_transform = this.transform;
    }

    void Update()
    {
        m_transform.Translate(new Vector3(0, 0, 1));
    }
}
```

2.使用内建数组
虽然ArrayList和Array用起来容易并且方便，但是相比较内建数组而言，前者和后者的速度还是有很大的差异。内建数组直接嵌入struct数据类型存入第一缓冲区里，不需要其他类型信息或者其他资源，因此用作缓存遍历更加快捷。实现代码如下。


脚本编译

作为一名Unity开发者，熟悉Unity脚本的编译步骤是很有必要的。这样可以让我们更加高效地编写自己的代码，如果代码出现了问题，还能有效地改正错误。由于脚本的编译顺序会涉及特殊文件夹，所以脚本的放置位置就非常重要了。
根据官方的解释，脚本的具体编译需要以下4步。

(1)所有在Standard Assets. Pro Standard Assets和Plugins中的脚本将被首先编译。在这些文件夹之内的脚本不能直接访问这些文件夹以外的脚本,不能直接引用类或它的变量,但是可以使用GameObject.SendMessage与它们通信。

(2)所有在Standard Assets/Editor, Pro Standard Assets/Editor和Plugins/Editor中的脚本接着被编译。如果想要使用UnityEditor命名空间,必须将你的脚本放置到这些文件夹。

(3)然后所有在Assets/Editor外面的,并且不在(1)、 (2)中的脚本文件被编译。

(4)所有在Assets/Editor中的脚本,最后被编译。