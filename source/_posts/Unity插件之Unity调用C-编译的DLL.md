---
layout: title
title: 'Unity插件之Unity调用C#编译的DLL'
date: 2019-04-04 10:10:04
tags:
---
Unity插件分为两种：托管插件（Managed Plugins）和本地插件（Native Plugins）。本文先来说说Unity中的托管插件，本地插件的文章留到下一篇文章再说。有时候我们会有这样的需求：给第三方公司提供Unity插件，而又不希望第三方看到具体代码，这时就可以将核心代码编译成dll文件供第三方调用。或者说，同一个公司多个项目都用到某个模块，则可以把该模块封装成dll，方便在不同项目之间共用和维护。

<!--more-->

# 一、创建DLL

打开VS，选择文件 -> 新建 -> 项目后打开新建项目对话框。接着操作如下：

{% asset_img 7.webp %}

点击确定后，编写如下代码：

```cs 
using System;

namespace MyLib
{
    public class MyClass
    {
        public static string GetBlogName()
        {
            return "Sheh伟伟的简书";
        }

        public static TimeSpan GetBlogTime(DateTime time)
        {
            return (time - DateTime.Parse("2016/11/14"));
        }
    }
}
```
然后生成dll文件，操作如下图：

{% asset_img 8.webp %}

# 二、导入DLL

将MyLib项目目录下的bin/Debug目录下的MyLib.dll文件拷贝到Unity项目的Plugins目录下。这时，Unity控制台报Unhandled Exception: System.Reflection.ReflectionTypeLoadException: The classes in the module cannot be loaded的错误，报错详细信息如下：

{% asset_img 1.webp %}

这是因为我用的Unity版本是5.3.4f1，支持的.NET Framework的版本为3.5。而我创建的MyLib项目的默认.NET Framework 3.5版本为4.5.2。选择项目，然后右键选择属性 -> 应用程序，将目标框架改为 .NET Framework 3.5或以下 ，如下图：

{% asset_img 2.webp %}

接着重新生成一下dll文件，重新导入到Unity就行了。

三、调用DLL

在Unity新建一个TestDll的脚本，并挂到主摄像机上，脚本代码如下：
```cs
using UnityEngine;
using System;
using MyLib; // 导入dll

public class TestDll : MonoBehaviour {

    private string blogUrl = "http://www.jianshu.com/users/fd3eec0ab0f2/latest_articles";
    void Start ()
    {
        string myBlog = string.Format("{0}:{1}", MyClass.GetBlogName(), blogUrl);
        Debug.Log(myBlog);

        TimeSpan span = MyClass.GetBlogTime(DateTime.Now);
        Debug.Log("写这篇博客到现在的时间间隔：" + span.TotalDays);
    }
}
```
注意，调用Dll中的方法一定要使用using语句引入导入到unity中的dll类库。

四、导入Unity DLL

怎么导入Unity原生类库呢？别急，接下来就说说怎么在自定义的dll类库中调用Unity中的类。
首先选中项目，接着右键选择添加 -> 引用后，弹出引用管理器，如下图：

{% asset_img 3.webp %}

在应用管理器界面点击浏览按钮，找到UnityEngine.dll文件点击确认按钮导入，如下图所示：

{% asset_img 4.webp %}

> Unity类库在Windows上的路径为Program Files\Unity\Editor\Data\Managed\UnityEngine.dll。

接着，将项目中无用的类库都移除掉，只保留用到的System和UnityEngine两个类库，如下图：

{% asset_img 5.webp %}

然后，修改MyClass脚本，代码如下：
```cs
using System;
using UnityEngine;

namespace MyLib
{
    public class MyClass
    {
        public static string GetBlogName()
        {
            return "Sheh伟伟的简书";
        }

        public static TimeSpan GetBlogTime(DateTime time)
        {
            return (time - DateTime.Parse("2016/11/13"));
        }

        public static void CreateCube()
        {
            GameObject go = GameObject.CreatePrimitive(PrimitiveType.Cube);
            int random = UnityEngine.Random.Range(-5, 5);
            go.transform.position = new Vector3(random, random, 0);
        }
    }
}
```
代码编写完成后，重新生成一下dll，然后导入到Unity中，接着修改Unity脚本TesDll，具体代码如下：
```cs
using UnityEngine;
using System;
using MyLib;

public class TestDll : MonoBehaviour {

    private string blogUrl = "http://www.jianshu.com/users/fd3eec0ab0f2/latest_articles";
    string myBlog;
    double time;

    void Start ()
    {
        myBlog = string.Format("{0}:{1}", MyClass.GetBlogName(), blogUrl);
        Debug.Log(myBlog);

        TimeSpan span = MyClass.GetBlogTime(DateTime.Now);
        time = span.TotalDays;
        Debug.Log("写这篇博客到现在的时间间隔：" + time);
    }

    void OnGUI()
    {
        GUILayout.Label(myBlog);
        GUILayout.Label(time.ToString());

        if(GUILayout.Button("Create Cube"))
        {
            MyClass.CreateCube();
        }
    }
}
```
运行后的效果图如下：

{% asset_img 6.webp %}