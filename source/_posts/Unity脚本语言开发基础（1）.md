---
layout: title
title: Unity脚本语言开发基础（1）
date: 2019-04-16 10:33:35
categories: Unity
tags: Unity5.X 完全自学手册
---
思考并回答以下问题：
1.值类型和引用类型的区别是什么？各自有哪些类型？
2.C#变量的赋值方式有两种，分别是什么样子的？
3.几种修饰符的作用范围？
4.怎么创建数组？给数组赋值？读取数组的值？

<!--more-->

# C#脚本语法

## 变量

C#是一种强类型语言，在使用任意一个对象前，必须声明这个对象的类型，如整数型、浮点型、字符串类型等。

C#的类型分为两大类：值类型和引用类型。两者的主要区别是值在内存中的存储方式不同。值类型的实例通常是在线程的堆栈上分配的，引用类型的对象则是在托管堆上分配的。

值类型包括内置类型（用关键字int、float、bool、string等声明）。

引用类型包括类（用关键字class声明）和委托（用关键字delegate声明）。

在Unity中，定义C#变量的格式如下：
**数据类型 变量名称**
例如，下面定义一个整数变量intNum。
```cs
int intNum;
```
我们可以通过对变量赋值来对其初始化，赋值的方式是使用“=”赋值运算符给变量赋值，赋值的格式有两种，一种为：
```cs
int intNum;
intNum = 5;
```
另一种是以字面形式初始化，形式如下:
```cs
int intNum = 5;
```
在C#声明变量之前还可以添加访问修饰符：
private，（默认修饰符）只能在本类中访问；
protected，只能由类或派生类中访问；
internal，只能在本项目中访问。

但是如果想让脚本中定义的变量在Unity的Inspector视图中显示，则必须用public修饰。

## 数组

数组是具有相同类型的一组数据，如一个班级、一副麻将、一副扑克牌等都可以看成一个数组。数组就是同一数据类型的组值。在C#中只能使用内建数组，如下所示。
```cs 
using UnityEngine;
using System.Collections;

public class ArraySample : MonoBehaviour
{
    private int[] array = new int[5];

    void Start()
    {
        for (int i=0; i<array.Length;i++)
        {
            array[i] = i;
            print_r(i);
        }
    }
}
```
把脚本添加到Unity中，得到运行结果，如下图所示。

{% asset_img 1.png %}

## 语句

for比while可以更好的控制循环次数。

## 函数

Unity脚本一些常用的内置运行函数。

Awake:在游戏运行时调用,用于初始化;
Start:只在游戏开始时执行一次,在Awake0函数后执行:
Updates在游戏每一帧都执行一次,在Start函数后执行;
LateUpdate:同Update,只是它会在Update()函数执行后再执行:
FixedUpdate: 当游戏中引入刚体系统,并使用适配的方式同步物理时钟时,可以让动力学更精确地计算:
OnGUI:绘制游戏界面的函数,因为每一帧执行多次,所以一些时间相关的函数要尽量避免直接在其内部使用;
OnMouseOver:鼠标光标停留在物体上时执行该函数的内容;
DnMouseEnter:鼠标光标进入物体范围时执行该函数的内容.和OnMouseOver不同,该函数只执行一次:
OnMouseExit:鼠标光标离开物体范围时执行该函数的内容;
nMouseDown:鼠标键按下时执行该函数的内容;
OnMouselp:鼠标键释放时执行该函数的内容;OnMouseDrag:按住鼠标键拖动时执行该函数的内容。
关函数。  
OnMouse系列函数是针对指定物体的,如果要使用全局鼠标控制操作,则需要使用射线相

## C#脚本
在Unity中使用C#脚本时需要注意以下几点。
1.所有的行为脚本必须从monobehaviour继承（直接的或间接的）。在Javascript中，这个是自动完成的，但是在C#中，必须显式注明。如果你通过Asset->Create->C Sharp Script创建脚本，系统模板已经包含了必要的定义。
```cs
public class newbehaviourscript : monobehaviour 
{
}
```

2.使用Awake或Start函数初始化. Awake和Start函数的不同之处在于, Awake函数是在加戟场景时运行, Start函数是在第一次调用Update或Fixedupdate函数之前被调用, Awake函数运行在所有的Start函数之前,

3.类名字必须匹配文件名.

4.协同函数(Coroutines)返回类型必须是1Enumerator,并且用yield returm替代yield.

5.不要使用命名空间,目前Unity暂不支持命名空间.

6.只有序列化的成员变量才能显示在检视面板中,私有和保护变量只能在专家模式中显示,属性不被序列化或显示在检视面板中。