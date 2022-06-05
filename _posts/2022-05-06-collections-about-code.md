---
title: Some quotes about coding
date: 2022-05-14 12:00:00 +0800
layout: post
current: post
cover:  assets/images/covers/2201654407127_.pic.jpg
navigation: True
tags: [Code quote]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true

---

> If you can't code, write books and blogs. record videos and podcasts.  
>
> — @NavalismHS

> Too many people hear this and start, jamming lines of code into their head thinking they need to memorize it all. Before you even write a line of code, learn to think like a program. Then memorize what's important. 
>
> — @SmellgoodToo

>创作者进入这个行业因为他们有好的品味，但创意工作需要长时间的积累。人们在初期看到自己的作品烂就放弃，恰恰是因为他们有好的品味，品味是王道——不要放弃。
>
>Nobody tells this to people who are beginners, I wish someone told me. All of us who do creative work, we get into it because we have good taste. But there is this gap. For the first couple years you make stuff, it’s just not that good. It’s trying to be good, it has potential, but it’s not. But your taste, the thing that got you into the game, is still killer. And your taste is why your work disappoints you. A lot of people never get past this phase, they quit. Most people I know who do interesting, creative work went through years of this. We know our work doesn’t have this special thing that we want it to have. We all go through this. And if you are just starting out or you are still in this phase, you gotta know its normal and the most important thing you can do is do a lot of work. Put yourself on a deadline so that every week you will finish one story. It is only by going through a volume of work that you will close that gap, and your work will be as good as your ambitions. And I took longer to figure out how to do this than anyone I’ve ever met. It’s gonna take awhile. It’s normal to take awhile. You’ve just gotta fight your way through.
>
>— Ira Glass, Hacker News 《Name one idea that changed your life》

> 道格原本想让程序与程序能够随意连接，但如何自然地描述一个无约束图并不那么容易，而且还存在语义上的问题：在程序之间流动的数据必须正确排队，而程序的无管理连接有可能容纳不了那么长的队列。而且肯无论如何也想不出实际应用场景。
> 但道格继续唠叨，肯继续思考。正如肯所说：“有一天，我想到了：管道。本质上就是管道。”他只花了一小时就在操作系统中添加了管道系统调用。他形容管道是“超级小菜”，因为I/O重定向的机制早已存在了。
>
> — 《Unix传奇：历史与回忆》

>具有好的编程能力的工程师能把真正复杂的项目落地，让设计和实现都可以做到足够的优雅，作者也一直在向这个方向努力 —— 不断提升编程能力。我们应该去实现身边经常使用的工具，多多造轮子来提升我们的编程能力：不要重复造轮子是一个错误的说法，正确的说法应该是不要发明轮子。
>
>抽象能力是决定编程能力的关键，人脑的处理能力绝对是有限的，没有人能够对 Linux、Kubernetes 中的每一行代码都了如指掌。我们理解一个项目的方式是自顶向下或者自底向上的，因为人脑同时能够处理的信息有限，无论哪种方式我们同时理解的都只能是项目的一部分。合理的抽象能够帮助我们更高效地理解项目的顶层架构和底层实现，降低心智上的负担。
>
>— 《2019 年总结· 拥抱变化》 面向信仰编程

> 当团队中有人使用通用语言进行交流时，其他人都可以明白他表达的准确含义和约束条件。通用语言和开发软件模型中使用的其他语言一样，在团队中无处不在。
>
> — 《DDD领域驱动设计》

#### 《Talks at Goole》关于复杂性的思想

- Working code isn't enough: must minimize complexity

- Complexity comes from dependencies and obscurity

- Strategic vs. tactical programming

- Classes should be deep

- General-purpose classes are depper

- New layer, new abstraction

- Comments should describe things that are not obvious from the code

- Define errors out of existence

- Pull complexity downwards

#### 云风关于写代码时的心理模型

1. 尽量分解问题，把字问题限定在单个函数/模块中。
2. 调用任何一个模块都假定它可能是有问题的，并考虑如果对方出现各种问题时，自身会有什么异常表现。
3. 把2的思考结果记下来并根据潜在的结果分类。这样在整体表现出bug时可以快速推测问题可能在哪里。

复杂的代码和数据结构，即使它们对外的接口简单；如果考虑其错误实现时可能的异常行为，就很复杂了。

#### How to skim a research paper

read in order, write a paper also focus these content:

1) Abstract
2) 1st paragraph of the intro
3) Last paragraph  of intro (for contributions)
4) 1st paragraph of the conclusion (it's usually one paragraph anyways)
5) figures /tables of results, and read their captions.

#### First Law of Software Quality

errors = (more code)²

E = mc²

