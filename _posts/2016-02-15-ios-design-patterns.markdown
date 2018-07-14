---
title: 一句话描述iOS中的设计模式
date: 2016-02-16 12:54:40 +0800
layout: post
current: post
cover:  assets/images/covers/DSC_7935.JPG
navigation: True
tags: [design pattern]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---

本文是对[《iOS Design Patterns》](http://www.raywenderlich.com/46988/ios-design-patterns)中涉及到的设计模式的简要概括。

虽然设计模式种类繁多，但是总体上又可以分为三类

- Creational: Singleton and Abstract Factory.
- Structural: MVC, Decorator, Adapter, Facade and Composite.
- Behavioral: Observer, Memento, Chain of Responsibility and Command.

`Creational`侧重于类实现的功能。`Structural`侧重于对象之间的组合结构。`Behavioral`则是提供了对象之间的一种交互方式。


### MVC设计模式
是iOS开发中最常见的设计模式，主要是将`Model`-`View`-`ViewController`的数据层和视图层分开，由控制层负责联系两边的行为。随着`ViewController`层的不断扩大，出现了`MVVM`的模式，使得`ViewModel`和`View`之间能够直接绑定吧，简化`ViewController`中的代码，为其瘦身。

### Singleton模式
一个类唯一的实例，可以全局访问，共享数据状态。

### Facade模式
提供简单的接口，把一系列复杂的接口进行封装。

### Decorator模式
在不改变原有类的基础上，对其进行扩展，增加行为和职责，如`Category`。另一种实现则是增加一层封装，在封装过程中，在保留原有类的基础上添加新的属性和行为，有种层层嵌套的感觉。

在文章中`Delegation`也被划分为`Decroator`的一种。可以理解为`Delegation`是主动暴露其接口由第三方类来实现的一种装饰模式。

### Adapter模式
将一个类的接口适配成另一个类可以调用的接口。侧重点调和两边不同的接口。
我之前一直将`Delegate`和`Adapter`混淆。`Adapter`侧重在适配不同的接口，是一座桥梁。Apple通过`Protocol`来实现`Adapter`的作用，其产物就是`Delegation`，隐藏了显式的`Adapter`。

### Observer模式
通过侦听发射者来获取其动作和状态，常见的为Notification和KVO。

### Memento模式
封装一个对象的内部状态或数据，使其可以在必要的时候保存或加载。封装保证了内部数据的隐秘性。

### Command模式
`Target-Action`模式，将对一个对象的某个操作行为进行封装，传递给其他对象或者队列，使得该行为适当时刻被传递的对象或者队列触发。

## 参考文献

- [Memento设计模式](http://www.dofactory.com/net/memento-design-pattern)
- [Cocoa Design Patterns](https://developer.apple.com/legacy/library/documentation/Cocoa/Conceptual/CocoaFundamentals/CocoaDesignPatterns/CocoaDesignPatterns.html#//apple_ref/doc/uid/TP40002974-CH6-SW5)