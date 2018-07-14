---
title: Swift大会填坑之旅
date: 2016-01-15 13:03:14 +0800
layout: post
current: post
cover:  assets/images/welcome.jpg
navigation: True
tags: [Swift]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---

中国第一届Swift大会已于2016年1月10日于北京结束。会上内容非常全面有价值，在[Github](https://github.com/atConf/atswift-2016-resources)有资料分享，会议视频在[慕课网](http://www.imooc.com/learn/600?from=http://each.dog)上可以看到。没有现场参与会议还是深表遗憾。此篇文章作为阅读讲义后的初步总结，如有疏漏还望指正。

### Think Functionally

`Functor`,`Applicative`和`Monad`是函数式编程的三个重要概念。ReactiveCocoa就是这三个概念的实践。`Functor`,`Applicative`和`Monad`是分别实现了接口（typeclas）的数据类型，并依次包含。最终使函数和数据可以通过统一的接口，处于一种平等的地位，无缝灵活地组合调用。这也是通向声明式编程的一种途径。帮助理解这三个概念的文章推荐雷纯峰的[《Functor、Applicative 和 Monad》](http://blog.leichunfeng.com/blog/2015/11/08/functor-applicative-and-monad/)。

傅若愚主讲的《Objective-C to Swift》和包涵卿《Functional Programming》中均给出了用函数式思想流式处理异步代码的实践。包涵卿讲稿中涉及到的Promise和Nomad这两个很相似的概念，这两个概念又分别衍生出框架[PromiseKit](https://github.com/mxcl/PromiseKit)和[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)。`Promise`倾向于对异步过程进行封装，通过一个队列维持依次执行的顺序。`ReactiveCocoa`倾向于对数据的封装，以组合和数据流的形式执行方法。

翁阳《Swift改善既有代码的设计》的实现方式粗看和`Monad`很像，实际上并不涉及数据流的传播，只是通过重载操作符巧妙地“隐藏”了异步操作中的嵌套，代码相互独立。在可读性上确实有了质的提升。

### Protocol Struct and Enum

随着`Protocol`重要性的增加，Swift已经具备纯正的面向接口开发能力。`Protocol`已经全面支持`Class`、`Strut`和`Enum`，可以自由组合并通过`extension`来拓展行为，相比OC中通过`Category`添加协议的主要作用是隐藏实现细节。Swift中接口作为一种数据类型可代表一类具有相同行为的对象，可以直接传递和调用，非常方便。

`Struct`和`Enum`的使用频率有非常大的增加,尤其是`Struct`。在Swift标准库中定义了多大102个结构体而类只有5个。`Struct`和`Enum`已经具备同`Class`一样的`Properties`、`Methods`和`Protocol`能力，区别在于结构体和枚举类型不能继承，在参数传递过程中是复制传递，这能避免类对象通过引用传值引起不安全的多线程操作。结构体的效率也比类要高得多。所以在Swift中多推荐首选结构体。

李洁信在主讲的《Pop in Swift》中给出了两个用`Protocol`解耦代码的例子。其中第二例子，MVVM中ViewModel使用`Protocol`对View渲染的代码进行解耦，增强了代码的复用性，非常值得借鉴。

### Performance
周楷文小哥带来的《Faster App》给出了优化程序性能的方案，讲的比较全面，且不限于Swift语言。性能优化通常都会涉及到`后台执行耗时操作`和`缓存复用`以及为了效率使用更底层的函数，具体方案根据使用场景而定。

- UI层面：
	- AutoLayout适用的场景。在频繁复用、变化和动画的场景下并不推荐，尤其在UITableView和UICollectionView下极其耗费性能。
	- 后台线程处理图片解码和裁剪，缓存及复用。在UITableView中的复杂的View可以缓冲甚至转换成图片来替代。
	- 使用TextKit来进行文字排相关操作。
	- 动画和视频选用恰当的底层函数。
- 网络操作：使用缓存和压缩传输数据。此外JSON数据和Model的映射上速度上Manual>YYModel>Mantel，在一般场景中并不需要考虑这个性能，会使用功能强大的Mantel。
- 文件操作：异步存取数据。
- 数据库操作：对比了CoreData和Realm。除了不是原生的支持外体积较大外，Realm在性能上完爆CoreData。

### Swift Style And Building a Framework
gregheo小哥讲了一些关于Swift的编码规范，如何让代码清晰可读性强。在他的[Github](https://github.com/raywenderlich/swift-style-guide)上有很具体的规范可以参考。

猫神的主题是《如何打造一个让人愉快的框架》，讲述了打造第三方框架需要注意的几个方面：

- API需命名清晰易懂，尽可能详细描述函数的行为，比如`recursivelyFetch`就比`fetch`来的具体。
- 添加前缀不与其它库冲突，比如`f1_method`,`f2_method`。
- 使用`Cocoapods` `Carthage` 和 `Swiftmanager`的注意事项。
- 版本管理1.0.0对应(major 公共 API 改动或者删减).(minor 新添加了公共 API).(patch bug 修正等)

最后猫神推荐了一个持续集成的方案[fastlane](https://github.com/AFNetworking/fastlane)，自动化部署frameworks。

### 总结
本来以为只是写个概要，怎奈讲稿的信息量比想象的大得多。光是去理解函数式编程从一个坑跳到另一个坑，而且适时选择不继续跳坑，这才断断续续花了近一周才完成这篇文章。对函数式编程和Swift特性的理解还不全面，需要进一步巩固。现在迫不及待想实践中应用学到的知识了。

### 参考文献

- [Swift中Struct和Class的对比](http://stackoverflow.com/questions/24232799/why-choose-struct-over-class/24232845)

- [Reactive Programming](https://www.coursera.org/course/reactive)