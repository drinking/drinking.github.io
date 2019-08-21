---
title: iOS省电指南—Reduce and Prioritize Work
date: 2019-08-21 16:39:48 +0800
layout: post
current: post
cover:  assets/images/covers/41566386531.jpg
navigation: True
tags: [battery energy timer ]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---

文章是苹果的官方文档《Energy Efficiency and the User Experience》的第一部分"Reduce and Prioritize Work"的简要梳理，刚好最近的功能需求会涉及到多种消耗CPU和IO的操作，预期会对用电量产生一定的影响。所以通过阅读该官方文档，好有一个全面的认知。

### 减少后台运算量
减少在后台工作包括及时终止必要的后台服务和防止不必要的服务在后台执行，降低必要服务的频率。这些都可以通过系统API完成。

苹果给出的几个例子：
- Not notifying the system when background activity is complete
- Playing silent audio
- Performing location updates
- Interacting with Bluetooth accessories
- Downloads that could be deferred

### 设置不同优先级
苹果对任务优先级有四个维度的划分，即对应Quality of service (QoS) 的四个枚举值。更用户的使用认知也是息息相关，比如交互的行为需要最快的反馈，一次数据的处理操作能够忍受一定的时间，一个下载操作需要更久。所以结合业务场景，选择合适的优先级相比一窝蜂都用高优先级能够带给用户更好的体验。
````
NSQualityOfServiceUserInteractive
NSQualityOfServiceUserInitiated
Default（不属于枚举中的）没有指定枚举值时的隐藏默认值，优先级为当前所在位置
NSQualityOfServiceUtility
NSQualityOfServiceBackground
Unspecified（不属于枚举中的）不涉及QoS的历史代码场景
```
### 减少计时器的使用
定时器会导致CPU或其他系统模块被从空闲态唤起，导致了一定的电能损耗。减少不必要的timer误用，比如靠timer轮训查看状态，就可以用通知的方式解决。对于倒计时功能的timer，可能暂时无法避免或者没有更好的方案。所以苹果也给了必要的三个使用选择。
- Use timers economically by specifying suitable timeouts.
- Invalidate repeating timers when they’re no longer needed.
- Set tolerances for when timers should fire.

设置timer的tolerance可以有效降低电能的损耗，因为如果多个timer的触发时间接近，系统可以高效的执行一次触发操作，大大提高调用效率。如果系统没有这种实现，在用户层面也可以实现一个类似的功能。另外如果同时有几个timer，低频的timer就可以由高频的timer统一管理，触发。比如每秒一次的timer就可以在第十次触发时顺带触发十秒一次的timer。NSTimer在系统层面应该也是有更高频的时钟在实行类似的管理。

### 引用文档
- [Energy Efficiency Guide for iOS Apps: Work Less in the Background](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/WorkLessInTheBackground.html#//apple_ref/doc/uid/TP40015243-CH22-SW1)