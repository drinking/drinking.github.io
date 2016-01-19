---
layout: post
title: 薄荷App开发中用到的第三方库
date: 2015-08-06 10:12:43 +0800
comments: true
categories: 
---

这篇文章简述了我们在重构薄荷App时所采用的第三方库，这些库是一款功能完整的App必要的组成部分。
在这里只是简单讲述各个库的应用场景或实现原理，有些比较复杂的框架还请读者自行了解其设计思想和使用方法，恕不累述。


###界面开发的相关库

#### [Classy](https://github.com/cloudkite/Classy)
借鉴的CSS的方式，将UI相关的属性写在配置文件中，方便全局使用和修改。

原理：通过方法交换实现挂钩，在UIView进行didMoveToWindow调用时，采用NSInvocation和KVC的方式，改变UIView的属性。

不需要重启App达到实时更新的方式是利用dispatch_source_t侦听文件变化，继而调用相应的设置UIView的方法。以下代码在其他场景下有使用价值。

{% highlight objective-c %}
+ (dispatch_source_t)watchForChangesToFilePath:(NSString *)filePath withCallback:(dispatch_block_t)callback
{
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    int fileDescriptor     = open([filePath UTF8String], O_EVTONLY);

    NSAssert(fileDescriptor > 0, @“Error could subscribe to events for file at path: %@“, filePath);

    __block dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_VNODE, fileDescriptor,
                                                              DISPATCH_VNODE_DELETE | DISPATCH_VNODE_WRITE | DISPATCH_VNODE_EXTEND,
                                                              queue);
    dispatch_source_set_event_handler(source, ^{
        unsigned long flags = dispatch_source_get_data(source);
        if ( flags ) {
            dispatch_source_cancel(source);
            callback();
            [self watchForChangesToFilePath:filePath withCallback:callback];
        }
    });
    dispatch_source_set_cancel_handler(source, ^(void) {
        close(fileDescriptor);
    });
    dispatch_resume(source);
    return source;
}

{% endhighlight %}
另一块比较复杂的代码是解析类CSS文件,可以深入学习借鉴。

### [AsyncDisplayKit](https://github.com/facebook/AsyncDisplayKit)
Facebook推出的异步UI渲染的框架，保证了复杂的UITableView滑动流畅性。

### [PureLayout](https://github.com/smileyborg/PureLayout)
以简单友好的方式封装了Apple自己啰嗦的AutoLayout代码。自动布局的利器。

### [Pop](https://github.com/facebook/pop)
FaceBook出品的动画效果引擎。

### [MTStringAttributes](https://github.com/mysterioustrousers/MTStringAttributes)
对NSAttributedString的创建过程进行封装，以MarkUp的形式创建，之后采用Slash解析字符串，对相应的Tag施以预设的属性。

### [PXRotatorView](https://github.com/drinking/PXRotatorView)
自家封装ReactiveCocoa和iCarousel而成的轮播器组件。


####其它常见的通用组件
- CRToast
- iCarousel
- IDMPhotoBrowser
- DZNEmptyDataSet
- FSCalendar
- JSBadgeView
- MJRefresh

###数据存储及文件操作

### [Mantle](https://github.com/Mantle/Mantle)
在Model,NSDictionary和Json之间进行智能地转换，省去手工转换代码的枯燥劳动。

###[YTKKeyValueStore](https://github.com/yuantiku/YTKKeyValueStore)和[FMDB](https://github.com/ccgus/fmdb)
FMDB是对SQLite的一层封装，YTKKeyValueStore在FMDB基础上提供了KeyValue操作数据存储的方法。现今的移动客户端数据越来越倾向走网络，本地的数据采用KeyValue的形式满足绝大多数需求。

### [BHFileManager](https://github.com/drinking)
自家封装的进行文件创建，删除等操作的库。使用了[ZipArchive](https://github.com/ZipArchive/ZipArchive)进行文件压缩解压缩和[FCFileManager](https://github.com/fabiocaccamo/FCFileManager)进行基本的文件操作。

### [GVUserDefaults](https://github.com/gangverk/GVUserDefaults)
NSUserDefaults的拓展，以set和get的形式就可以使用持久化的对象，适合处理配置信息等一类信息量不大的数据，非常方便。

###网络层

### [BHAPI]()
自家实现的网络层通信库，实现了一套json格式的API文档转ObjectiveC代码的机制，统一管理。底层基于MKNetworkKit，也方便换成AFNetWroking。

### [AFNetWorking](https://github.com/AFNetworking/AFNetworking)
路人皆知的网络通信层框架

### [SDWebImage](https://github.com/rs/SDWebImage)
路人皆知的图片异步加载的库。此外，我们还应用这个库的图片存储接口，进行其他图片的缓存。

###其它工具库

###[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)
信号量的合并及串联，将异步代码从繁复的嵌套中脱离出来，以优雅的高内聚的方式执行。大大增加代码的可阅读性和可维护性。MVVM的设计思想，简化了传统MVC中Controller层的代码量，将一部分代码抽离到ViewModel层，和View层进行单向或双向的数据绑定。精简了业务代码，增加了复用性。RAC已经成为我们项目中，最不可或缺的库，当发现能采用如此优雅的代码解决问题时，你再也不想回到过去了。


### [JSPatch](https://github.com/bang590/JSPatch)
使用JS方式更改线上app的行为，主要用以修复Bug。

原理：通过改变消息转发的目标，将原生方法指向JS方法，达到方法替换的目的。详细原理在作者的博客中有阐述，请移步[bang's blog](http://blog.cnbang.net/)。

### [Tweaks](https://github.com/facebook/Tweaks)
设置一些全局调控的开关，比如QA和Release环境切换。另一点也是其主要的应用，进行UI数据的实时的微调。不过在使用Reveal调节界面后，也极少会用到。

### [DateTools](https://github.com/MatthewYork/DateTools)
时间判断和比较上很方便的库。

### [BHRouter]()
自家实现的以URL的形式进行页面间的跳转，去除了ViewController之间的耦合。支持如Http等其它协议，使得在Native和WebView之间自如跳转。
