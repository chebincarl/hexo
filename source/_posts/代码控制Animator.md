---
layout: title
title: 代码控制Animator
date: 2019-05-27 18:17:32
categories: Unity
tags: 大话Unity2018
---
今日思考题
1.实现人物的站立、走、跑的状态切换

<!--more-->

# 动画资源神器

https://www.mixamo.com/#/
注册了一个Adobe账户登陆进去，就看到了很多人物的模型。

人物模型选择

人物模型选择

选好一个人物模型，再切换到Animation，哇，这么多的动作！


“哈哈，这是Adobe收购mixamo以后免费开放的，不仅仅有这些哦，这个网站还有一个强大的功能是自动绑定骨骼，如果你有一个角色想在游戏中使用，但是如果连骨骼都没有绑定的话，是没办法用骨骼动画的，也没办法将其他动画重定向到这个模型上。点右上角的UploadCharacter就可以上传自己的模型，自动绑定骨骼了！”


代码控制Animator

我们在学Transition的时候，只用到了按播放时间切换动画，但是更强大的功能是按参数切换动画

“那当然了，我们当时把Animator类比为一个视频播放器，视频播放器在我点击下一个的时候当然要能切换到下一个视频了”

“打开Animator看左上角”


“原来都没发现还有一个Parameters”
“你没用到当然没发现了，现在你可以添加一个参数，比如人物的速度”
“好，那我添加一个float类型的speed参数”
“这时候比如你想让人物从静止状态切换到走路状态，就可以加一个transition，条件用speed >0了，你试试”

“添加上了！现在我怎么才能看到效果呢？”
“现在把场景运行起来，手动改一下那个speed的值就能看到效果了！”

调整Animator参数.gif

“但是现在感觉不太流畅，问题在哪呢?”
“第一个，你这个Transition除了speed大于0这个条件，还有一个条件是Has Exit Time，就是播放完idle的动画并且speed>0才会切换到下一个动画，这个Has Exit Time得去掉。第二个，你这个两个动画都应该是循环播放吧”
“知道了，我改掉再试试”
“改成这样了，嘿嘿，貌似可以了。那我怎么用代码控制呢，比如说我按键盘上WASD的时候控制人物移动。”
“使用动画系统时，有两种控制人物移动的方式：”

使用动画中的位移
这种好处是：人物的脚步会跟地面贴合，不会出现滑步的问题（人物的移动距离比步子大或者小），控制简单
这种缺点是：比较依赖动画的制作，程序控制性不高

使用代码控制人物的位移
好处：可控性高
缺点：容易出现滑步，控制复杂

小新：“那我先来试试这个简单的方式吧，使用动画中的位移。”
大智：“这种方式需要先设置两个地方：1、物体上Animator组件的Apply Root Motion需要勾选，相当于就是把动画中的位置修改应用到物体上；2、人物的动画类型需要设置为Humanoid。这两个地方设置好以后，再用代码去修改animator组件中的参数就行了，这个你去看文档！”
“好嘞，我去设置一下”

小新设置完这两个地方后，就开始去查看Animator组件的文档，怎么修改Animator Controller中的参数呢？小新在文档中寻找了半天，并没有找到相关的说明。但是找到了Unity的一篇教程，里面说到设置参数的办法是使用SetInteger、SetFloat、SetBool、SetTrigger四个方法。小新在Unity的API文档中找到了这几个方法，并记下了下面的笔记：

SetInteger、SetFloat、SetBool、SetTrigger分别对应Paramters中的Int、Float、Bool、Trigger类型。

SetInteger有两个重载：

public void SetInteger(string name, int value);
public void SetInteger(int id, int value);
对于第一个重载，第一个参数类型是string，对应的是parameter中的参数名称。第二个参数是要设置的值。

对于第二个重载，第一个参数是animator中parameter中参数的ID什么是参数的ID？

其他SetFloat、SetBool、SetTrigger都类似，但是唯一不同的是SetFoat还有额外的两个重载方法：

public void SetFloat(string name, float value, float dampTime, float deltaTime);
public void SetFloat(int id, float value, float dampTime, float deltaTime);
其中前两个参数和上面是类似的，不同的是后面两个参数。

dampTime 阻尼时间。

deltaTime 时间增量。
隐隐约约感到一股神秘的力量，但是不是特别明白

难点详解
“大智，我看完了，但是我想考考你，嘿嘿，则呢么样？”
“还考我？放马过来吧！”大智内心道：“你这点小技俩我还看不出来，不就是自己没看懂的地方么！”
“听好了，第一题：bool参数和Trigger参数的区别是什么？”
“bool参数和trigger参数很像，都是代表布尔值，但是trigger参数只能被设为true，一旦被transition使用，就会自动被设为false。”
“回答的不错，嗯。。。”小新沉思了10秒钟
“小样儿，就你。bool类型一般用于持续的状态，比如角色是否趴下。而trigger一般用于使用一次就会恢复的状态，比如开枪，开枪动画播放完以后，会自动恢复到之前的动作。”

“我明白了。第二题：parameter的id是什么？”
“我们在设置parameter的时候设置的是一个字符串的名称，但是在Unity内部是有一个数字id跟它对应的，使用Animator.StringToHash这个API可以将字符串的参数名转为数字id。使用数字id的代码运行效率会稍微高一些。”

“第三题：SetFloat的那个damp是怎么用的？”
“damp翻译过来一般是阻尼的意思，你可以理解为缓行。这样Fload值会渐变过去，而不是一下子变成设置的Float值，这个在有些情况下很有用，比如人物的速度。玩家按下W的时候，应该是一个逐渐从0到最大速度的过程，而不应该一下从0到最大速度，这时候就可以用到damp。如果你对那两个参数还不知道怎么设置，可以看一下这个公式：”

# 总结

Animator中可以设置参数，用来控制Transition的变化

Has Exit Time也是transition切换的一个条件，只有transition的所有条件都满足时才会进行切换

在代码中可以使用Animator类中的SetXXX方法控制参数，进而控制状态的转换。