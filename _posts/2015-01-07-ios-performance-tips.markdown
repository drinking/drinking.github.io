---
layout: post
title: "iOS App性能提升的技巧"
date: 2015-01-07 13:18:43 +0800
comments: true
categories: iOS
---

从[25 iOS App Performance Tips & Tricks](http://www.raywenderlich.com/31166/25-ios-app-performance-tips-tricks)翻译了部分提高app性能的技巧

##中阶性能提升建议
###9)复用和延迟加载
更多的视图意味着更多的绘制，这些最终意味着更多CPU和内存的开销。在通过UIScrollView展示很多视图时开销尤为明显。

因此，效仿UITableView和UICollectionView的思路，并不在一开始就创建所有视图，在需要的展示时候创建，并不显示的视图加入重用队里里。

通过这个方法我们显示新的视图时，只需要设置复用视图的属性，而避免了新建视图会引起的内存分配和初始化开销。

延迟加载的方法也可以用在其它情景。比如你需要通过点击按钮来展示一个视图，至少有两种实现形式:

1. 在页面加载时创建并隐藏。需要时再显示出来。
2. 直到点击按钮时才创建并显示。

这两种方法各有优缺点。第一种会长期占用内存空间，但是显示速度快。第二种刚好相反，不占空间但展示慢。具体采用哪种方式可以根据应用场景权衡。

###10)缓存、缓存、缓存
一项普遍的的开发经验就是"缓存那些有必要缓存的东西"，即缓存那些不太会改变但是经常访问的数据。
具体可以缓存什么数据呢？比如服务器返回的数据，图片，甚至是计算后的值如UITableView的高度。
NSURLConnection已经根据HTTP请求头将资源存储到本地或内存中，你甚至可以人为设置NSURLRequest，让它只访问缓存的数据。

```objective-c
	+ (NSMutableURLRequest *)imageRequestWithURL:(NSURL *)url {
	    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
	 
	    request.cachePolicy = NSURLRequestReturnCacheDataElseLoad; // 这一选项会总是返回缓存的图片
	    request.HTTPShouldHandleCookies = NO;
	    request.HTTPShouldUsePipelining = YES;
	    [request addValue:@"image/*" forHTTPHeaderField:@"Accept"];
	 
	    return request;
	}
```

注意，你可以用NSURLConnection来进行URL请求，同样AFNetworking也可以。而且你不需要改变什么代码，因为它已经做得足够好。

如果你想了解更多关于HTTP缓存的知识，可以涉猎NSURLCache,NSURLConnection等相关知识，确保浏览了NSHipster网站上的[这篇文章](http://nshipster.com/nsurlcache/)
如果你需要缓存不包含在HTTp请求中的数据，可以涉猎NSCache。NSCache和NSDictionary表现的很像，但是当系统需要更多空间资源时，它会被释放。Matt Thompson在NSHipster写了一篇[很赞的文章](http://nshipster.com/nscache/)可以阅读一下。
想要了解更多关于HTTP缓存的知识，推荐阅读Google的[《 best-practices document on HTTP caching》](https://developers.google.com/speed/docs/insights/LeverageBrowserCaching)。

###11)考虑图形绘制
在iOS上有好几种方式可以实现漂亮的按钮。可以用全尺寸图片或可伸缩图片显示，再复杂点用CALayer，CoreGraphics甚至OpenGL来绘制。

这些效果的实现难度不同和当然性能也不同。有篇很棒的文章讲述[iOS图形性能](http://robots.thoughtbot.com/designing-for-ios-graphics-performance)，很值得阅读。同时，苹果UIKit项目组的成员Andy Matuschak也针对这篇文章给出了深度的分析。

简而言之，使用图片是最快的，因为iOS省去了绘制图形的过程，直接将图片渲染到屏幕上。问题是你需要把所有的图片都放入bundle中，这增加了app的大小。所以使用可伸缩尺寸的图片来绘制按钮更好。iOS会帮你绘制那些重复的纹理。你也不用针对不同尺寸的元素（如按钮）生成不同的图片。

毕竟通过使用图片你可能会丧失图片微调的能力，当你需要对不同的场景对视图进行微调时，你依旧需要处理大量的图片，不能通过代码直接调整。这会是一个繁复的过程。

所以，在绘制性能和app大小上，你需要根据自己的需要进行折中。



##高阶性能提升建议
###22)提升程序启动速度
启动速度提升主要的核心在于避免阻塞主线程，异步处理繁重的任务，如网络请求、数据库操作或者解析数据。同时避免加载臃肿的XIB文件，XIB文件在主线程中加载，而storyboard不存在这个问题，有需要可以尽可能考虑storyboard。

>注意，手机在通过Xcode调试时，watchdog并不会运行，所以要测试真正的启动速度时应断开连接。

###23)使用Autorelease Pool
NSAutoreleasePool是用于释放对象的程序块。通常情况下由UIKit来调用。但是在有些场景中我们需要认为创建。

比如当你需要创建大量的临时对象时，内存使用量会因此飙升直到未来某时刻统一被UIKit释放。这就意味这些临时对象存在的时间比需要的时间长。所以我们需要手动及时地释放这些对象。

```objective-c
    NSArray *urls = <# An array of file URLs #>;
        for (NSURL *url in urls) {
            @autoreleasepool {
                NSError *error;
                NSString *fileContents = [NSString stringWithContentsOfURL:url
                                         encoding:NSUTF8StringEncoding error:&error];
            /* 处理数据或其它创建操作 */
        }
    }
```

###24)选择性图片缓存
有两种常见的方式来加载app bundle中的图片。`imageNamed`和`imageWithContentsOfFile`。前者会先从系统缓存中寻找图片对象，如果没有则创建新的对象并缓存，适用于频繁使用的图片。后者没有缓存机制，适合不常使用的资源消耗较大的图片。

###25)避免使用Date Formatter
在大量需要用到NSDateFormatter的情景下，可使用类似单例的形式初始化，重复使用一个NSDateFormatter对象。如果想要进一步提高速度，则可以采用[C语言的方式](http://sam.roon.io/how-to-drastically-improve-your-app-with-an-afternoon-and-instruments)。还有一种更好的方式就是使用Unix timestamps。一个标示时间的浮点数，纪录了自1970年1月一日到现在的时间间隔。通过NSDate很容易的实现时间转换。

```objective-c
    - (NSDate*)dateFromUnixTimestamp:(NSTimeInterval)timestamp {
        return [NSDate dateWithTimeIntervalSince1970:timestamp];
    }
```