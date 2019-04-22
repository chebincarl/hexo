---
layout: title
title: Unity脚本语言开发基础（1）
date: 2019-04-16 10:33:35
categories: Unity
tags: Unity5.X 完全自学手册
---
本章涵盖：
* C#脚本语法
* 创建脚本
* 常用脚本API

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
intNum=5;
```
另一种是以字面形式初始化，形式如下:
```cs
int intNum=5;
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

