---
layout: title
title: Android那些事
date: 2019-03-14 21:32:31
tags: Unity
---

Unity的Android开发配置

<!--more-->

# 所需组件配置

## Unity中的下载
Unity需要认识Android，Android需要在我们的电脑中。
在Unity中操作：
（1）Android Build Support下载
（2）Switch Platform

{% asset_img 1.png %}

## JDK的安装与配置
Android是利用Java来开发的，所以需要先安装Java。谷歌JDK，安装JDK8（此文版本：jdk-8u152-windows-x64）。
JDK版本：1.8 必须大于等于7，不能是9。
Unity requires the 64-bit version JDK 8 (1.8).

查看数字签名。exe上右键“属性->数字签名”，如下图所示。
{% asset_img 2.png %}

点击jdk.exe安装JDK，目录D:\Install\java\jdk。
安装完JDK后，会附赠安装JRE，不能和JDK一个目录，安装在D:\Install\java\jre。

安装完成后，此电脑，右键“属性->高级系统设置->高级->环境变量”。

系统变量，新建，JAVA_HOME变量，变量值选择或填写JDK的安装目录。

{% asset_img 3.png %}

系统变量，找到Path变量，编辑，在变量值最后输入JDK安装目录下的bin目录（可以直接输入也可以写成%JAVA_HOME%\bin）与JRE安装目录下的bin目录。

{% asset_img 4.png %}

系统变量，新建，CLASSPATH变量，变量值填写：
.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar

{% asset_img 5.png %}

运行cmd，输入java -version与javac -version，
若显示版本信息，则说明安装和配置成功
{% asset_img 6.png %}

## Android Studio的安装
Android Studio下载（能打开哪个用哪个）
https://developer.android.com/
https://developer.android.google.cn/index.html （一般是这个）
http://www.androiddevtools.cn/
此文安装版本为3.3.2。
安装Android Studio时会安装SDK。

{% asset_img 7.png %}
{% asset_img 8.png %}
{% asset_img 9.png %}
{% asset_img 10.png %}
{% asset_img 11.png %}
我的电脑是AMD的CPU，所以装HAXM失败了。
{% asset_img 12.png %}
{% asset_img 13.png %}

只有Android Studio编译成功，Unity才可以成功。所以下面直接用Android Studio编译一个空工程。

{% asset_img 14.png %}
{% asset_img 15.png %}
没有遇到什么问题。然后Build一个APK。之前的版本可能会在Build时出现各种问题。但这个3.3.2版本没有。
{% asset_img 16.png %}
当出现以下提示时，则Android Studio自己可编译成功了。
{% asset_img 17.png %}

## 在Unity中设置AndroidSDK与JDK的目录
## Android Studio的配置工作
## Android SDK的规整工作 

# 发布相关知识
## 三种BuildSystem的简介
## 创建自己的密钥库用于签名
## 使用Unity发布Android空工程之顺利的情况
## 使用Unity发布Android空工程之不顺利的情况

# 调试
## 安装Google USB驱动及获取Unity Remote
## UnityRemote原理介绍及移动端相关API概览
## UnityRemote的使用
可以看见程序Debug打印的东西，还可以断点。
## AVD的使用
Android Virtual Device
能看到Android里面的logcat，能看到Android系统里面的一些信息。
AMD的CPU在创建模拟器时abi不要选x86，选armeabi就行了。
{% asset_img 100.png %}
## 真机结合logcat调试与第三方模拟器调试

# 移动端适配技术

## 使用UGUI中的锚点功能进行自适应
## 设置Canvas的自适应缩放
## 通过代码设置视口的自适应
## Touch类的简单介绍
## Touch类的剩余成员讲解
## 制作手指追踪的小Demo
## 制作单指手势识别的小Demo与返回键的操控权
## 其他常用手势识别的思路与双端通用的API介绍
## Unity标准资源中的虚拟摇杆简介

# 与Android互相调用
## Unity与Android的联合开发模式简介
## 互相调用模式之导出Jar包+扩展MainActivity+Java主导（上）
## 互相调用模式之导出Jar包+扩展MainActivity+Java主导（中）
## 互相调用模式之导出Jar包+扩展MainActivity+Java主导（下之Unity调用Android）
## 互相调用模式之导出Jar包+扩展MainActivity+Java主导（下之Android调用Unity）
## 互相调用模式之导出Aar包
## 互相调用模式之提供扩展类
## 互相调用模式之C#主导式调用入门
## 各种模式的适用情况讲解
## Unity显示Toast之通过拓展MainActivity实现
## Unity显示Toast之通过C#调用实现
## AndroidJavaProxy的使用方式讲解
## 获取讯飞语音听写的SDK
## 创建供Unity使用的SDK库（上）
## 创建供Unity使用的SDK库（下）
## 完成语音识别SDK的接入

# Player Settings
## PlayerSettings中一些参数的设置 