---
layout: post
title: CATransition初探
date: 2015-04-25 12:46:45 +0800
comments: true
categories: 
---
本周的工作中有一个替换UINavigationBar标题字符串的小需求。为了使字符串的替换更自然，我找了利用CATransition实现动画自然过渡的方法，新字符串淡出旧字符串淡出。

```objectivec
CATransition *animation = [CATransition animation];
animation.duration = 0.5;
animation.type = kCATransitionFade;
animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];
[self.navigationController.navigationBar.layer addAnimation: animation forKey: @“changeTextTransition”];
self.navigationController.navigationBar.topItem.title = title;
```
在我对iOS动画的局限的认知中，动画是通过时间函数或帧变化逐渐改变View的属性值，如frame，color以实现动画效果。然后CATransition的变化过程中，有新旧两个值的残影，用之前的思路是显然是无法走通的。出于好奇，对CATransition的实现原理进行了了解，下图为其在CAAnimation中的继承位置，和我们常用的CABasicAnimation和CAKeyframeAnimation不是一个分支。因此可以料想到实现上的差异。
![screenshot]({{ site.url }}/assets/img/4-27/65cc0af7gw1dxlusbklpmj.jpg)
进一步了解其实现过程，根据国外友人在SO的[回答](http://stackoverflow.com/questions/2233692/how-does-catransition-work):

>When you add the transition as an animation, an implicit CATransaction is begun. From that point on, all modifications to layer properties are going to be animated rather than immediately applied. The way the CATransition performs this animation to to take a snapshot of the view before the layer properties are changed, and a snapshot of what the view will look like after the layer properties are changed. It then uses a filter (on Mac this is Core Image, but on iPhone I’m guessing it’s just hard-coded math) to iterate between those two images over time.

一个CATransition动画启动后的持续过程中，所有对这个layer所包含的属性值的操作都将以动画的方式进行。为了达到渐变效果，会先截取初始视图的快照，然后截取变化后最终视图的快照。通过算法在两个快照之间做变化的效果。两张快照的变化过程依旧遵循时间函数。

所以就最终解释了为什么在启动Animation后，通过设置NavigationBar的Title值，依旧能够实现淡入淡出效果了。


