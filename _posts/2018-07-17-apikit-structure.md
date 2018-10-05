---
layout: post
current: post
cover:  assets/images/covers/IMG_5530.JPG
navigation: True
title: 拼装玩具APIKit
date: 2018-07-17 10:00:00
tags: [APIKit structure]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---
APIKit是一个解析文档网页最终生成网络请求代码和模型代码的工具。

 其目的用来帮助程序员从开发API代码的繁复体力劳动中解放出来。哪怕现阶段生成的代码不能保证完全可运行，起码部分还是可以复制粘贴，比手写要快不少。另外结合Git管理，每天可以定期执行一边，可以帮你发现服务端对于接口字段的变更或者文档页面的有效性。保证客户端的代码实时更新。

之所以叫它拼装玩具，是因为APIKit本身只实现了读取文档HTML页面并解析成源数据。从加载配置文件到最终生成代码的过程，是处理数据流的一个过程，其中用到了诸多开源的工具，比如使用`quicktype`来把JSON生成模型。使用`Sourcery`结合自定义的模版，把网络请求元数据构造成iOS的代码。使用`clang-format`对代码进行格式化。另外`sed,grep`等命令也起到了格式化和胶水的作用。

APIKit算是这个工具的第三代了。以前尝试把所有的功能写在一个项目代码中，这样调用起来会比较集中和方便。但是增加了维护成本，在没有找到合适的开源代码，需要自己维护诸如模型转换，模版生成等代码。繁复且缺乏灵活性，就像是半吊子，啥都做了，却远不如专业工具的强大。所以最后明确了自己要做的仅仅是解析成那些工具所支持的元数据，其它事情交给他们就好了。而且使用`Sourcery`完全可以定义另一套模版就能给其它语言使用。

![ ]( /assets/img/2018/apikit_structure.png )

上图就是这个拼装玩具的流程图。之前浏览其它网站的时候，看到有哥们拿Sketch绘制流程图，效果甚好。这次也尝试使用了一下，确实方便许多。以后就更有动力写文章了哈。

### 参考资料
- [Code Beautifier in Xcode](http://blog.manbolo.com/2015/05/14/code-beautifier-in-xcode)