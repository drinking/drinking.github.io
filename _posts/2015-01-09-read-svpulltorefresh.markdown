---
layout: post
title: "iOS源码阅读之SVPullToRefresh"
date: 2015-01-09 20:30:49 +0800
comments: true
categories: iOS
---

`SVPullToRefresh`是基于`UIScrollView`的扩展，动态添加了下拉刷新的视图。下面来简单看一下添加视图的实现过程。

首先在类别中声明一个`SVPullToRefreshView`的property。

```objc
@interface UIScrollView (SVPullToRefresh)
...
@property (nonatomic, strong, readonly) SVPullToRefreshView *pullToRefreshView;
...
@end
```
然而Category并不支持为类添加属性和成员变量，所以需要通过关联对象的方法来动态添加。我们发现在.m文件中引入了`#import <objc/runtime.h>`头文件，并用@dynamic来声明pullToRefreshView，@dynamic的作用是告诉编译器手动实现setter和getter方法。关于objective-C的其它保留字可以参考[唐巧的博文](http://blog.devtang.com/blog/2013/04/29/the-missing-objc-keywords/)。

```objc

#import <objc/runtime.h>

static char UIScrollViewPullToRefreshView;

@implementation UIScrollView (SVPullToRefresh)
...

@dynamic pullToRefreshView;

- (void)setPullToRefreshView:(SVPullToRefreshView *)pullToRefreshView {
[self willChangeValueForKey:@"SVPullToRefreshView"];
objc_setAssociatedObject(self, &UIScrollViewPullToRefreshView,
pullToRefreshView,
OBJC_ASSOCIATION_ASSIGN);
[self didChangeValueForKey:@"SVPullToRefreshView"];
}

- (SVPullToRefreshView *)pullToRefreshView {
return objc_getAssociatedObject(self, &UIScrollViewPullToRefreshView);
}

...
@end
```
核心部分的setter方法中，使用了`objc_setAssociatedObject `，其作用就是为一个对象动态添加没有声明的变量。以下是[SO](http://stackoverflow.com/questions/5909412/what-is-objc-setassociatedobject-and-in-what-cases-should-it-be-used)上的详细解释。

>objc_setAssociatedObject adds a key value store to each Objective-C object. It lets you store additional state for the object, not reflected in its instance variables.

>It's really convenient when you want to store things belonging to an object outside of the main implementation. One of the main use cases is in categories where you cannot add instance variables. Here you use objc_setAssociatedObject to attach your additional variables to the self object.

>When using the right association policy your objects will be released when the main object is deallocated.

这样一个自己实现的下拉刷新视图就可以动态添加到`UIScrollView`上。之后`SVPullToRefreshView `以弱引用保有其所在的scrollView变量，通过KVO侦听`UIScrollView`的`contentOffset`的变化，实现了自己的一套状态逻辑，最终达到我们所见的效果。具体实现细节还请参考[源码](https://github.com/samvermette/SVPullToRefresh)。