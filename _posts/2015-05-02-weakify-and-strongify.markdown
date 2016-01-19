---
layout: post
title: weakify和strongify探究
date: 2015-05-02 23:02:26 +0800
comments: true
categories: 
---

@weakify和@strongify是一组非常简洁搭配使用的宏，用来避免因循环引用而导致内存泄露。由开源项目[libextobjc](https://github.com/jspahrsummers/libextobjc)提供，被ReactiveCocoa广泛应用而进一步被熟知。

由于之前对宏不甚了解，看这两个宏的实现时非常头大。好在猫神的[宏定义的黑魔法 - 宏菜鸟起飞手册](http://onevcat.com/2014/01/black-magic-in-macro/)给了非常大的帮助，是篇非常好的入门文章，这里就不再累述了。优雅的宏能够帮我们节省工作量，实乃居家旅行之必备神器。

@**weakiy和@strongify本质上是对传统方法的简化和强化**。

传统的写法，获取self类型，声明weakSelf的变量并在最前面设置weak属性。


{% highlight objective-c %}
__weak __typeof__(self) weakSelf = self;
{% endhighlight %}

@weakify则省事多了，真是不得不感叹：不想偷懒的程序员不是好Geek啊！
简洁的背后是宏的功劳，得到预编译后的代码，在DEBUG模式下显示如下。

{% highlight objective-c %}
@autoreleasepool {} __attribute__((objc_ownership(weak))) __typeof__(self) self_weak_ = (self);
{% endhighlight %}
在Release模式下。和传统方式基本如出一辙。
{% highlight objective-c %}
try {} @catch (…) {} __attribute__((objc_ownership(weak))) __typeof__(self) self_weak_ = (self);
{% endhighlight %}

只是开头的字段实在莫名其妙，光看代码完全不能理解其用意。搜了很多地方，最后在[libextobjc](https://github.com/jspahrsummers/libextobjc/blob/8942c6bca6a06717ee9edc0bf02b24b9d8ac4d77/extobjc/EXTScope.h)宏定义的文件注释中找到了说明（论一手资料的重要性）。其作用仅仅是不让编译器产生warnings。在DEBUG模式下用autorelease来保持编译器分析的完成性，而在release环境下用try/catch是为了避免产生过多的autorelase pool影响性能。实乃没有良策的折中（这些都是哪门子的奇淫技巧啊）。

>Details about the choice of backing keyword:

>The use of @try/@catch/@finally can cause the compiler to suppress return-type warnings.
>The use of @autoreleasepool {} is not optimized away by the compiler,resulting in superfluous creation of autorelease pools.

>Since neither option is perfect, and with no other alternatives, the compromise is to use @autorelease in DEBUG builds to maintain compiler analysis, and to use @try/@catch otherwise to avoid insertion of unnecessary autorelease pools.


weakSelf在Block中被引用，因其是弱引用的关系存在被释放的风险。
{% highlight objective-c %}
__weak __typeof__(self) weakSelf = self;
dispatch_group_async(_operationsGroup, _operationsQueue, ^
{
[weakSelf doSomething];//原有对象被释放，weakSelf有变成nil的风险 (待考证释放和nil是什么样的过程）
} );
{% endhighlight %}
所以引入了@strongify，release环境下预编译后的代码如下。self_weak_赋值给新声明的局部变量self，此次赋值覆盖了全局的self，使self成为强引用局部变量，在函数执行完后才会释放，保证了运行中的安全。

**注意self_weak_是由@weakify生成的变量，@strongify中引用了该变量，在宏的实现中。self_weak_隐藏在宏的实现中，非常隐晦。所以@strongify是不可能独立出现的，编译也不会通过。**
{% highlight objective-c %}
@try {} @finally {}
 __attribute__((objc_ownership(strong))) __typeof__(self) self = self_weak_; 
{% endhighlight %}


但是在调用@strongify之前，self_weak_就有可能已经失效，所以进行一层判断还是挺有必要的。
{% highlight objective-c %}
@weakify(self)
dispatch_group_async(_operationsGroup, _operationsQueue, ^
{
    @strongify(self)
    if(self){
        //安全生产，放心使用
    }
} );
{% endhighlight %}

最后@strongify还有一点值得注意的地方，在声明局部self变量时会出现编译器`-Wshadow`的警告，此处进行了一层处理，选择无视该警告。
{% highlight objective-c %}
#define strongify(…) \
    ext_keywordify \
    _Pragma(“clang diagnostic push”) \
    _Pragma(“clang diagnostic ignored \”-Wshadow\””) \
    metamacro_foreach(ext_strongify_,, __VA_ARGS__) \
    _Pragma(“clang diagnostic pop”)
{% endhighlight %}


#### 参考资料：
1. [suggest use strongify](http://stackoverflow.com/questions/17104634/referring-to-weak-self-inside-a-nested-block)
2. [Asynchronous Design Patterns with Blocks, GCD, and XPC](https://developer.apple.com/videos/wwdc/2012/?id=712)
3. [libextobjc](https://github.com/jspahrsummers/libextobjc/blob/8942c6bca6a06717ee9edc0bf02b24b9d8ac4d77/extobjc/EXTScope.h)
4. [how strongly works](http://stackoverflow.com/questions/28305356/ios-proper-use-of-weakifyself-and-strongifyself/29436402#29436402)
5. [reactiveCoca weakify and strongify](http://stackoverflow.com/questions/21716982/explanation-of-how-weakify-and-strongify-work-in-reactivecocoa-libextobjc/21716983#21716983)
6. [Making all self references in blocks weak by default](https://coderwall.com/p/vaj4tg/making-all-self-references-in-blocks-weak-by-default)