---
layout: title
title: 与Mono协同工作
date: 2019-03-09 14:25:00
categories: Unity
tags: Unity脚本设计
---
C#通常不会提供创建游戏所需的全部内容。就C#语言而言，其自身并不会加载、解析XML语言以支持游戏数据的存储；另外，该语言也无法创建窗口对象和GUI微件，以对复杂数据集执行高级的搜索和查找行为。当实现某些附加操作时，用户通常需要向外部库寻求帮助。其中，某些库可直接从Unity的Asset Store下载，这一类库往往用于实现特定的功能。相应地，Unity发布了Mono Framework，该框架具有免费、跨平台特征，并可见为Microsoft NET框架（编程库）的开源实现，其中包含了大多数类。对应类可用于处理字符串、文件输入和输出、搜索和排序数据、记录动态表、解析XML等。这也说明，作为可扩展的工具箱，Mono可高效地管理应用程序中的数据。本章将考察Unity应用程序中Mono的多种部署方式，即考察表、栈、Linq、正则表达式以及枚举结构等内容。
<!--more-->

# 表和集合

存储数据表是游戏编程中较为常见的任务，而此类数据的本质特征也存在多种变化方式，其中包括积分榜、玩家状态，库存物品，武器装备量，关卡表等。出于速度和效率考虑，应尽量采用静态数组存储数据。简而言之，静态数组于先期创建，其最大尺寸从一开始便固定，在运行期内，数据项可被添加和删除，但全部尺寸不会发生变化。当然，如果数据量未达到数组的最大尺寸，则对应空间将被浪费。顾名思义，静态数组适用于存储数据表（其内容通常较少变化），例如游戏中的关卡，采集的全部武器装备等。

然而，用户常会使用到动态数组，其尺寸可增长或收缩，并与处于变化状态下的真实数据相适应。例如生成或销毁敌方角色、库存物品的变化、武器装备的获取或丢弃等。

Mono Framework提供了多个类可维护数据表，其中包括3个主要类，即List、Stack和Dictionary，各个类用于实现不同的功能。

## List类

如果需要使用<span color="red">单一数据类型的无序数据项序列表，并可增长或收缩以匹配于存储数据的实际尺寸</span>，则List是一个较为理想的选择。List可用于添加和删除数据，并按照顺序遍历全部存储项。除此之外，<span color="red">List对象还可在Unity的Object Inspector中进行编辑。</span>

```cs 
using UnityEngine;
using System.Collections;
using Systen.Collections.Generic;

// Sample enemy class for holding enemy data
[System.Serializable]
public class Enemy
{
	public int Health = 100;
	public int Damage = 10;
	public int Defense = 5;
	public int Mana = 20; // 法力
	public int ID = 0;
}

public class Using_List: MonoBehaviour
{
	// List of active enemies in the scene
	public List<Enemy> Enemies = new List<Enemy>();

	void Start()
	{
		// Add 5 enemies to the list
		for (int i=0; i<5; i++)
		{
			Enemies.Add(new Enemy()); // Add method inserts item to end of the list
			// Remove 1 enemy from start of list (index 0)
			Enemies.RemoveRange(0, 1);	
		}

		// Iterate through list
		foreach (Enemy E in Enemies)
		{
			// Print enemy ID
			Debug.Log(E.ID);
		}	
	}
}
```

当使用List类时，需要包含System.Collections.Generic命名空间。
如果表数据类型声明为System.Serializable类，则该表可在Object Inspector中予以显示。
在类的成员声明中，用户可通过一条语句声明并初始化新的表实例。
通过Add方法，新对象可添加至表的尾部。
数据项可通过多种方法被移除。其中，RemoveRange方法可从表中删除多个连续数据项，其他的移除方法还包括Remove、RemoveAll以及RemoveAt。
用户可采用foreach循环遍历表中的全部数据项。
总体而言，在循环遍历过程中，不可添加或移除数据项。

未运行时

{% asset_img 1.png %}

点击play后，下图显示了 Object Inspector中的List类。
自动生成了5项，有一项被删除了，所以剩4项。

{% asset_img 2.png %}
{% asset_img 3.png %}

List类支持多种方法，可逐项或整体移除数据项，并可于表循环外部进行操作。然而，在某些场合下，通过循环遍历操作则是一类相对简单、方便的处理方法。例如，用户需要在处理完毕后移除各个数据项。一类较为经典的操作是：删除场景中的全部引用类型对象，例如敌方角色，且同时移除对应的数组数据项，以避免产生null引用。然而，循环方式的数据项移除方式可能会产生问题。对于迭代器而言，这很容易在数组内丢失数据项的位置信息，其原因在于，全部数据项的数量在循环过程中发生变化。当在某一次处理过程中执行循环和移除操作时，应反向遍历数组（而非前向）。
```cs
//Remove all items from a loop
void RemoveAllItems ()
{
    // Traverse list backwards
    for (int i= Enemies.Count-1; i>=0; i--)
    {
        //Call function on enemy before removal
        Enemies[i].MyFunc();

        // Remove this enemy from list
        Enemies.RemoveAt(i);
    }
}
```
前向删时删除为0的，则第二个默认又变成0了。

## Dictionary类

当用户根据某一键值对搜索并直接访问特定元素时，将是Dictionary类的用武之地。对于表中的各个数据项，用户需要存储对应的键或ID，并以此进行独立识别。随后，Dictionary类可根据唯一的键直接访问相应的数据项。对于拼词类游戏，如果用户希望查找特定单词的含义或分值时（在字典或单词库中），Dictionary类的功能类似于真正的字典。其中，单词自身定义为键，而单词的具体信息则表示为值。

当然，用户也可通过多个List对象复制此类行为，而非使用Dictionary类。但在计算性能方面，Dictionary类则具有明显的速度优势。用户可以较小的性能开销在字典中存储海量的数据，因而可实现基于键值对的快速数据查找行为。
```cs
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class Using Dictionary : MonoBehaviour
{
	// Database of words. <Word, Score> key-value pair
	public Dictionary<string, int> WordDatabase = new Dictionary<string, int>();

	void Start()
	{
		// Create some words
		string[] words = new string[5];
		Words [0] ="hello";
		Words [1] ="today";
		Words [2] ="car";
		Words [3] = "vehicle";
		Words [4] ="computers";

		// add to dictionary with scores
		foreach (string Word in Words)
		{
			WordDatabase.Add(Word, Word. Length);
		}
		
		// Pick word from list using key value
		// Uses array syntax!
		Debug.Log("Score is:" + WordDatabase["computers"].ToString());
	}
}
```

类似于List类，此处应包含System.Collections.Generic命名空间。
声明并创建字典。与List类不同，Dictionary并不会出现于Unity的Object Inspector中。
Dictionary类通过Add方法添加数据。
除了利用键数据确定各项数据元素（而非数组索引）之外，Dictionary类中的元素其访问方式类似于数组。

# Stack类

在纸牌游戏中，玩家需要抽取最上方的纸牌；另外，对于取消历史记录、路径搜索编码、复杂的法术召唤系统，以及汉诺塔游戏中，均会看到栈结构。根据后入先出（LIFO）规则，栈可定义为一种特殊的表。用户可将数据项置入表中，并在垂直方向上相互堆叠，且最近置入的数据项位于栈的最上方。随后，可从栈顶逐一弹出数据项（从数组中移除数据项）。相应地，弹出的顺序通常与置入的顺序相反。

因此，栈对于撤销或回绕操作十分有效。
```cs 
using UnityEngine;
using System. Collections;
using System. Collections. Generic;

[System.Serializable]
public class PlayingCard
{
	public string Name;
	public int Attack;
	public int Defense;
}

public class Using_Stack: MonoBehaviour
{
	// Stack of cards
	public Stack<PlayingCard> CardStack = new Stack<PlayingCard>();
	// Use this for initialization
	void Start()
	{
		// Create card array
		PlayingCard[] Cards = new PlayingCard[5];
		// Create cards with sample data
		for (int i=0; i < 5: i++)
		{
			Cards[i] = new PlayingCard():
			Cards [i].Name = "Card-0" + i. ToString();
			Cards[i].Attack = Cards[i].Defense =i*3
			// Push card onto stack
			CardStack.Push(Cards[i);
		}
		
		//Remove cards from stack
		while (CardStack.Count > o)
		{
			PlayingCard PickedCard = CardStack.Pop();
			// Print name of selected card.
			Debug.Log(PickedCard. Name);
		}
		
	}
}
```

{% asset_img 4.png %}

Object Inspector什么都没有

{% asset_img 5.png %}

# IEnumerable和IEnumerator接口

当与数据集协同工作时，例如List，Dictionary和Stack等，用户通常会根据特定的方案遍历表中的全部或部分数据项。如前所述，某些时候，用户需要前向遍历序列中的数据项，而在其他场合下，后向遍历则更加方便。对此，用户可采用标准的for循环。然而，该过程中可能会产生某些问题。对此，可通过IEnumerable和IEnumerator接口处理这一类问题。
```cs
// Create a total variable
int Total = 0;

// Loop through List object,from left to right
for (int i=0; i < MyList.Count; i++)
{
    // Pick number from list
    int MyNumber = MyList[i];

    // Increment total
    Total+= MyNumber;
}
```
当使用for循环时，问题主要体现在3个方面，此处暂且讨论前两个问题。首先，循环的语法内容并无特别之处，其中使用了整型迭代变量（i）访问数组数据元素。其次，迭代器自身并不具备“边界安全”这一特征。实际上，这可产生上溢或下溢问题，并导致越界错误。

在某种程度上，上述问题可通过相对整洁的foreach循环予以解决，进而保证边界安全，并采用更为简单的语法结构，代码如下所示。
```cs
// Create a total variable
int Total= 0;

// Loop through List object, from left to right
foreach (int Number in MyList)
{
    // Increment total
    Total += Number;
}
```
不难发现，foreach循环更为简洁，且兼具良好的可读性，但实际问题远不止于此。foreach循坏仅适用于实现了IEnumerable接口的类。实现了IEnumerable的对象须返回基于IEnumerator接口的有效实例。因此，对于工作于foreach循环内的某一对象，该对象依赖于两个接口。对于简单的循环和遍历行为，当前操作显然过于复杂。对应的处理方法可描述为：IEnumerable和IEnumerator不仅可处理前两个问题（即基于foreach循环的简单语法和边界安全问题），还应可解决第三个问题。特别地，该方法应可遍历对象组（甚至是非数组类型）。也就是说，可遍历不同的对象类型，该功能十分强大, 下面将通过具体实例对此加以考祭。

例如，在中世纪风格的RPG游戏场景中，居住着不同的邪恶法师角色采用Wizard类进行编码，这一类角色以随机地点和随机时间间隔出现于关卡中,并通过召唤法术、执行某些破坏任务对玩冢进行干扰。相应的随机生成结果可描述为:默认状态下,玩豕无法知晓某一时刻场景中的法师数量。尽管如此,这里依然需要获取法师的全部数量;或许,法师角色可能处于禁用、隐身、暂停或被销毁状态;抑或需要知晓其全部数量以防止其数量超出一定范围。因此,若不考虑法师的生成过程及其随机性,最终依然需要根据要求访问关卡中的全部法师角色。

如前所述,第2章曾定义了一个可遍历的法师角色列表,如示例代码6-7所示。
// Get all wizards
Wizard[] WizardsInScene = Object.FindObjectsOfType<Wizard>();

// Cycle through wizards
foreach (Wizard W in WizardsInScene )
{
// Access each wizard through W
}


当频繁使用时, FindObjectsOfType函数的计算速度较慢,且性能较差。

对此,可通过IEnumerable和IEnumerator实现类似的行为,以消除性能问题。当采用上述两个接口时,可使用foreach循环高效地遍历场景中的全部法师角色,即使此类角色位于数组中,如示例代码6-8所示。

示例代码6-8