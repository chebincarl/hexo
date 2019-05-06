---
layout: title
title: Namespaces
date: 2019-05-05 13:44:43
tags:
---
思考并回答以下问题：
命名空间是什么？怎么用？

<!--more-->

As projects become larger and the number of scripts increases, the likelihood（可能性） of having clashes between script class names grows ever greater. This is especially true when several programmers are working on different aspects of the game separately and will eventually combine their efforts in one project. For example, one programmer may be writing the code to control the main player character while another writes the equivalent（相等） code for the enemy. Both programmers may choose to call their main script class Controller, but this will cause a clash when their projects are combined.

To some extent（在某种程度上）, this problem can be avoided by adopting（采用） a naming convention or by renaming classes whenever a clash is discovered (eg, the classes above could be given names like PlayerController and EnemyController). However, this is troublesome when there are several classes with clashing names or when variables are declared using those names - each mention of the old class name must be replaced（替换） for the code to compile.

The C# language offers a feature called namespaces that solves this problem in a robust（强大的） way. A namespace is simply a collection of classes that are referred to using a chosen prefix on the class name. In the example below, the classes Controller1 and Controller2 are members of a namespace called Enemy:
```cs
namespace Enemy 
{
    public class Controller1 : MonoBehaviour {
        ...
    }
    
    public class Controller2 : MonoBehaviour {
        ...
    }
}
```

In code, these classes are referred to as Enemy.Controller1 and Enemy.Controller2, respectively（分别地）. This is better than renaming the classes insofar as（只要） the namespace declaration can be bracketed（括号） around existing class declarations (ie, it is not necessary to change the names of all the classes individually). Furthermore, you can use multiple bracketed namespace sections around classes wherever they occur, even if those classes are in different source files.

You can avoid having to type the namespace prefix repeatedly by adding a using directive at the top of the file.
```cs
using Enemy;
```
This line indicates that where the class names Controller1 and Controller2 are found, they should be taken to mean Enemy.Controller1 and Enemy.Controller2, respectively. If the script also needs to refer to classes with the same name from a different namespace (one called Player, say), then the prefix can still be used. If two namespaces that contain clashing class names are imported with using directives at the same time, the compiler will report an error.