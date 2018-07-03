---
title: 既生堆何生栈
comments: true
---
### 前言
这篇文章缘起于研究iOS字符串时查的一些资料，因为堆栈是一个通用的概念，所以单独拿出来作为一篇。

### 堆(Heap)和栈(Stack)
为什么堆栈总是被同时提及，而又傻傻分不清楚？在一般的中英词典中Stack(n. 堆；堆叠)和Heap(n. 堆；许多；累积)，都被翻译成堆，并没有出现栈这个词。而栈在中文词典里是“储存货物或供旅客住宿的房屋”的意思。所以推测译者是为了在中文里区分这两个“堆”而特意找的相近的词——“栈”。

那么在英文里Stack和Heap有什么区别呢？在朗文英语词典中，Stack意思是`a neat pile of things `，它指向了一个近义词Heap，意思是`a large untidy pile of things`。区别在于Stack是整齐地堆叠，Heap是一大物品的堆积。其实在中文翻译中已经有所体现，只是容易让人忽视，即“叠”和“积”的差异。闪耀着令人惊喜的语言智慧。

在计算机领域，数据结构和内存概念中都出现了Heap这个词，也很容易混淆。最初Heap是表示的一种树型的数据结构，根据根节点大于还是小于子节点，分为最大堆和最小堆。

更多的情况下，当我们说堆栈时，说的是一种内存的分配方式。如下图所示，

![ ]( /assets/img/2018/9c2VH.png )
> Stacks in computing architectures are regions of memory where data is added or removed in a last-in-first-out  manner.

>   Heap  is a common name for the pool of memory from which dynamically allocated memory is allocated.

Wiki也解释的比较清楚。Stack的内存分配逻辑比较清楚，先进后出。Heap分配方式就比较灵活，至于内存的管理，因语言和系统不同会存在差异，这里先不展开了。

### 堆栈存在的理由
堆栈从本质上讲是提供程序运行所需要的内存空间，但为什么会出现两种形式？因为程序不仅仅是由数据构成，还包括处理数据的逻辑方法。而这些方法的编码和执行顺序是非常符合后出先进的设计逻辑。因此栈的存在更多是为程序的方法服务，所以我们经常说的调用栈就是指可以查看程序调用顺序的内存空间，在栈顶部的就是最后执行的方法。方法中的临时变量如下图所示，也会存在栈中。理论上这些变量全部在堆(Heap)中生成也是可行的。
![ ]( /assets/img/2018/i6k0Z.png )

但是在栈中存储有两个优势，一个是由于栈的特性，栈中的方法执行完后，会实时从栈顶移除。栈中的临时变量的声明周期也因此结束，内存释放效率非常高。另一方面，栈的大小通常是固定的，在我的MacBook上通过`ulimit -s`查看是8MB。相较8GB的内存，为何栈大小只有区区8MB。其中的一种说法是，栈的调用，也就是程序执行的速度，不仅仅依赖于CPU，也同样依赖内存的读取速度。所以栈的存储会依赖于大小有限的高速缓存，如果栈过大，则高速缓存会出现大量内存空间的交换，反而降低了执行的速度。而珍贵有限的栈空间，就应该尽量避免存储容量大或者声明周期长的数据。

所以堆(Heap)就有它存在的场景了，内存空间大，数据存在时间长 （依赖于不同的内存释放策略），通过地址引用，也相对松散。

### 堆栈的方向
另一个经常让人困惑的点在于，常见的文章中，堆栈经常是以相反的方向进行内存分配。如下图所示，Stack由高地址向低地址索取空间，Heap则相反从低向高。但是也有很别处是Stack自上而下，Heap相反。所以是不是写错了？到底哪种才是正确的分配方案。其实根据系统架构的不同，这两种方式都是存在的。只是顺序的颠倒🙃️，没有其它不同。
![ ]( /assets/img/2018/1094457-20170112144012306-484648661.png )

至于为什么会放在一起，大概是在上古时期，内存有限的情况下，Heap和Stack能共享一段连续空间，根据自身情况，在保证安全的情况下，向对方霸占一些领地。从而更好的利用内存。
在现今的操作系统和硬件上，就很少出现这么紧俏的情况，而且Stack和Heap也不会在连续的物理内存中。Heap和Stack连续的情况更多是出现在以虚拟内存中，来表示程序的结构。

### 结语
这篇文章只是涉及堆栈的基础概念。因为在调研途中有不少有趣的收获，便记录于此。同时也认识到基础的重要和真相的朴素。因为时间有限，像内存的置换和管理，虚拟内存等概念没有进一步深入，其实都是教科书的基本概念。
存在的纰漏和不足还望指正。

### 参考资料
- [objective c - Are NSStrings stored on the heap or on the stack and what is a good way to initialize one - Stack Overflow](https://stackoverflow.com/questions/7376261/are-nsstrings-stored-on-the-heap-or-on-the-stack-and-what-is-a-good-way-to-initi)
- [iphone - Why do weak NSString properties not get released in iOS? - Stack Overflow](https://stackoverflow.com/questions/11107729/why-do-weak-nsstring-properties-not-get-released-in-ios)
- [objective c - Why isn’t my weak reference cleared right after the strong ones are gone? - Stack Overflow](https://stackoverflow.com/questions/15266367/why-isn-t-my-weak-reference-cleared-right-after-the-strong-ones-are-gone)
- [iOS中，对象释放机制以及__weak、__unsafe_unretained的一些问题 - Rain技术支持 - 开源中国](https://my.oschina.net/rainwz/blog/1835660)
- [c - How to find whether a given address is in heap or in stack - Stack Overflow](https://stackoverflow.com/questions/33798216/how-to-find-whether-a-given-address-is-in-heap-or-in-stack)
- [memory management - What and where are the stack and heap? - Stack Overflow](https://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap)
- [将stack翻译成"堆栈"实在是误人子弟 - veli - 博客园](https://www.cnblogs.com/idorax/p/6277906.html)
- [Why does the stack address grow towards decreasing memory addresses? - Stack Overflow](https://stackoverflow.com/questions/4560720/why-does-the-stack-address-grow-towards-decreasing-memory-addresses)
- [1. Why size of stack memory is fixed? - Stack Overflow](https://stackoverflow.com/questions/38727205/1-why-size-of-stack-memory-is-fixed)
