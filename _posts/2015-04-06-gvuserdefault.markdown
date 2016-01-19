---
layout: post
title: "GVUserDefaults的源码研究"
date: 2015-04-06 15:37:53 +0800
comments: true
categories: 
---

###简介
GVUserDefaults是一个封装NSUserDefaults以达到仅仅通过使用property的set和get方法就可以实现本地存储。GVUserDefaults可以通过category对property进行分类，非常方便管理和使用。比如在`GVUserDefaults+user.h`下声明的property是用户本地数据，而`GVUserDefaults+event.h`下的就是事件相关的本地数据。

###概念梳理
GVUserDefaults的代码不多，核心功能用C语言和runtime相关的函数实现。因为对这些概念和函数不太了解，所以先梳理下。

####C语言函数

1. [char *strdup(const char *s)](http://linux.die.net/man/3/strdup)复制字符串到新开辟的内存空间并返回新位置的指针。
2. [char *strsep(char **stringp, const char *delim)](http://linux.die.net/man/3/strsep)作用类似NSString的componentsSeparatedByString，但是只进行一个分割，返回值指向分隔符之前的字符串位置，参数stringp更新至分隔符后的字符串位置。
3. [char *strstr(const char *haystack, const char *needle)](http://linux.die.net/man/3/strstr)找到haystack中第一次出现needle的字符串的指针位置。

####objc/runtime相关概念
1. [SEL](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/tdef/SEL)表示函数Selector的一个字符串指针，指代函数名。唯有通过sel_registerName注册的方法才能拥有Selector。编译器会根据源码编译时自动生成相应函数的Selector。在运行时，我们可以手动调用生成。
2. [IMP](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/tag/IMP)是指向函数的指针，该函数接收self和Selector参数。
3. [class_copyPropertyList](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/func/class_copyPropertyList)，[property_getName](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/func/property_getName)，[property_getAttributes](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/func/property_getAttributes)分别为获取类的property列表，名称和属性。属性由字符串组成，描述了property的声明时所有的特性。[官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtPropertyIntrospection.html#//apple_ref/doc/uid/TP40008048-CH101)给出了相关的格式和实例。
4. [sel_registerName](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/func/sel_registerName)用C的字符串注册一个方法，返回该方法的Selector。
5. [class_addMethod](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/func/class_addMethod)为一个类增加方法，提供Selector,IMP以及参数类型字符串作为参数。参数类型字符串标示着返回值和参数的数据类型。

###实现流程

GVUserDefaults的核心方法是generateAccessorMethods，在初始化时调用。通过遍历当前类的property列表，与事先实现的存取方法（见下）匹配，动态添加property的setter和getter方法。为了能够知道每个Selector对应存储的Key，通过一个叫mapping的字典以setter和getter的字符串作为Key，property的名称作为Value来进行索引。

如下为一个整型的setter和getter的方法实现。同样类型的property指向相同的IMP，通过SEL名称找到相关的Key以实现存取。
{% highlight objective-c %}
static void integerSetter(GVUserDefaults *self, SEL _cmd, int value) {
NSString *key = [self defaultsKeyForSelector:_cmd];
[self.userDefaults setInteger:value forKey:key];
}

static int integerGetter(GVUserDefaults *self, SEL _cmd) {
NSString *key = [self defaultsKeyForSelector:_cmd];
return (int)[self.userDefaults integerForKey:key];
}
{% endhighlight %}

###小结
GVUserDefaults的源码非常简短，却实现了一个令人眼前一亮的功能。自此我们便可以方便存取，再也不用多写琐碎的代码和记那些搞不灵清的KeyValue。这篇文章对源码进行了简短的剖析。当然读万卷书不如看源码，希望通过这篇文章的索引可以为对runtime陌生的同志提供帮助。
