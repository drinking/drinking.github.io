---
layout: post
title: "评测框架简述"
date: 2015-03-08 09:45:08 +0800
comments: true
categories: 
---

##动机

最近的开发需求是写一个评测的功能，根据用户回答一系列的问题，最终生成评测结果。题型包括选择题和输入题，但拥有各样的展示方式。一个答题页会放一道或两道题,根据选择的不同会跳转不同到题目。原先的实现方式是每一种答题页独立实现，相似的答题页和组件实现复用，也能基本满足需求。但是我想做一个拓展性更强的足以面对更多变化的框架，所以尝试性进行重构。

##组件化
组件化的概念就是将一个页面的视图进行拆分,通过布局文件使其能自由组合在一起。如同HTML通过标签铺陈一样，自上而下构建所解析的视图，并且拥有相应的属性。

```
    [ //JSON格式的Layout文件
     {"view":"TitleLabel","text": "您的性别和生日？"},
     {"view":"ChoiceView","questionid":"0"},
     {"view":"PickView","questionid":"1"},
     {"view":"BiChoiceView"},
    ]
```

HTML布局的高度是无限制的，可以上下滚动。而我们需要在限定的高度上进行布局，所以为了达到适配效果构建了一个BaseView。它有一个flexibleHeight的属性标明这个View是否固定高度，不固定视图的高度＝屏幕高度－固定视图的高度和，最终会填满整个屏幕。这些限定是通过iOS的autoLayout实现的。
![lceva screenshot]({{ site.url }}/assets/img/3-8/LCEvaStructDemo.png)

##组件层次
偏于展示的视图`UILabel`,`UIImageView`通过`BaseView`进行了一层封装，使其可以自适应高度。`QuestionView`是问题类视图的基类，它在原有增加了`loadQuestion`，`routePage`等方法，来处理相关的加载问题和跳转位置等操作。`SelectableView`对选择题视图的封装，通过KVC和KVO来处理选项之间的变化。
![lcevastcut screenshot]({{ site.url }}/assets/img/3-8/LCEvaStruct.png)

##题目结构

起初布局和题目是放在一个文件里，随着布局和题目属性的增加变得相当难以维护。所以将两者独立出来。以questionid来索引到相关的题目。题型的不同导致题目文件没有统一的标准，这部分的解析和加载就交给各自的视图去完成。以下代码中表示，编号为0的题目是一个二选一的题型，选择`爷们儿`或`软妹子`，并配有相应的图片和不同的跳转项。

```
    "0":{
        "type":"bichoice",
        "text":["爷们儿","软妹子"],
        "imgs":["male.png","female.png"],
      	  ...
        "nextpage":["page2","page3"],
        "prevpage":"page0"
    },
```

##总结

在准备初期我已经考虑过组件化的可行性，并花了2天时间成功搭起了框架。随着深入细节会出现一些未曾考虑过或者考虑不完善的状况，这时就需要及时调整和重构。编码量也比预期的要多出许多，之后也一定会有不小的调整。好在一切都在掌控范围内。当框架有血有肉后，就能呈现较好的效果，如图。
![screenshot]({{ site.url }}/assets/img/3-8/demo.png)