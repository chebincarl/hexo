---
layout: title
title: Target Matching/动画重定向
date: 2019-05-31 17:11:32
categories: Unity
tags: 大话Unity2018
---
今日思考题

用Target Matching把跳跃的动画好好调一调

<!--more-->

绝地求生里面人物可以双手撑墙跳过一个堵墙或者窗户，想让角色的双手能恰好放到墙上，这个应该怎么做呢？

# Target Matching（目标点匹配）

可以使用Target Matching技术。在游戏中，经常有这种情况：角色的手或者脚需要在特定时间放在特定的位置。比如角色需要用手撑着跳过一个石头或一堵墙，或者跳起抓住房梁。Target Match就是让动画的特定片段去匹配特定的位置。

API：Animator.MatchTarget

Animator.MatchTarget的方法原型为：
```cs
public void MatchTarget(Vector3 matchPosition, Quaternion matchRotation, AvatarTarget targetBodyPart, MatchTargetWeightMask weightMask, float startNormalizedTime, float targetNormalizedTime = 1);
```
这个方法用来自动调节GameObject的位置和旋转。

参数的作用：
matchPosition 匹配的位置
matchRotation 匹配的旋转
targetBodyPart 身体的部位，从AvatarTarget枚举中选择。
weightMask 设置位置和旋转匹配的权重
startNormalizedTime 开始匹配的时间，注意是单位化时间（动画开始位置是0，结束位置是1）。如果开始时间已经超过了当前动画的播放时间，会匹配下一次符合的时间。比如：startNormalizedTime传入0.2，但是当前已经播放到0.3，会匹配下一次循环的0.2的位置。
targetNormalizedTime 最终匹配到的单位化时间。如果值大于1，可以设置特定循环数后的位置。比如2.3代表第2次循环的30%位置。

Unity会自动调整GameObject的位置和旋转来保证到特定时间时角色的特定部位能达到指定的位置和旋转。Target matching只能对Base Layer（index 0）生效。同一时间只能有一个match target生效，后续一个的match target需要等待前面的执行完成，再后续的match target会被忽略。

## 参数详解

首先前两个参数是匹配的位置和旋转，比如要匹配墙上，那就是墙上的位置，旋转就是让手的角度贴合墙，应该可以用调IK的方式来调。

targetBodyPart的类型是一个枚举，可以用来匹配对应的位置，比如说匹配左手，那就应该传入左手。weightMask是一个结构体，可以同时设置位置和旋转的权重，这个权重的概念我们已经接触很多次了，如果为1就是完全匹配到对应的位置和旋转。


最后两个参数是开始匹配和完全匹配上的时间。比如想在特定的时间匹配到对应的位置，那么肯定不能到那个时间点一下子把动画拉过去，那样太突兀了。动画中很多的混合都是为了让动画看起来更平滑，更自然。所以除了这个匹配到的特定时间，也就是targetNormalizedTime，还需要一个开始匹配的时间startNormalizedTime。开始时间到了就开始去进行匹配混合，到targetNormalizedTime时正好精确的到达匹配的位置。


注意

Target Matching只能对Base Layer生效

# 动画重定向

动画重定向，也就是Retargeting，比如将角色A的动画应用到没有动画的角色B上面，实现动画的重用。

要想重用动画不一定要用到Retargeting系统。如果两个角色的骨骼结构一样，那么动画其实是可以直接重用的。

那为啥还要这个Retargeting系统呢？

虽说两个角色的骨骼结构一样，动画是可以直接重用。但是要求很严格，包括骨骼的命名、骨骼的数量、骨骼的父子结构等等要求一模一样。这种情形一般只会出现在同一个动画师手里。如果骨骼稍微有些差别，那就没办法直接重用了。


这时候就需要用到Retargeting系统了，但是需要注意的是Retargeting系统只能用于人形动画。重定向相当于将不同的骨骼结构都映射到了Unity的Avatar上面。之前我们配置过AvatarMask，和那个有点像。配置的位置就是在模型导入设置的Rig标签里

{% asset_img 1.png %}
配置Avatar的入口


点击Configure可以进入配置界面，Unity会自动创建一个新场景用于Avatar的配置。

{% asset_img 2.png %}
Avatar配置界面


在Mapping页面下，可以将模型的骨骼节点映射到Unity的Avatar上面对应的骨骼节点。在Musule & Settings页面下可以设置每个骨骼节点可以旋转的范围，避免出现不合常理的骨骼旋转。

Retargeting系统就相当于将模型A和模型B的骨骼都映射到了一个Unity Avatar上面，这样最终都是用这个同一个Avatar来控制动画，达到重用的目的。

相当于用了一个中间件，将两个不同的东西匹配了起来。

在设置模型的Rig的时候，需要注意的是，Avatar Definition除了Create From This Model这个选项之外，还有一个选项是Copy From Other Avatar，如果两个模型的骨骼结构是一致的，可以直接使用这个Copy From Other Avatar将Avatar复制过来，就不需要再重新设置一遍了，也能节省内存。

# 总结

这两个东西里面虽然都有Target，但是其实是两个不同的东西。Target Matching用来匹配动画到特定的位置，Retargeting系统是用来重用动画，让骨骼结构不同的动画可以复用。