---
title: Hook objc_msgSend to hotfix
date: 2019-07-30 20:00:48 +0800
layout: post
current: post
cover:  assets/images/covers/IMG2019-07030.jpeg
navigation: True
tags: [objc_msgSend disassembly]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---

一切从戴铭老师的[《App 启动速度怎么做优化与监控？》](https://time.geekbang.org/column/article/85331?utm_term=zeusD4FM2&utm_source=web&utm_medium=infoq&utm_campaign=presell-161&utm_content=arti0318)一文说起。里面谈到了一种通过fishhook来hook objc_msgSend来实现App性能监控的方案。

objc_msgSend为了性能考虑是用汇编直接实现的方法，所以在Hook时需要与汇编打上交道。而汇编的资料非常的不全面，所以在学习过程中东拼西凑地理解，效率不是很高。花了一段时间阅读[GCDFetchFeed](https://github.com/ming1016/GCDFetchFeed)源码，其替换被戴铭总结为以下几步，每一步对于不了解汇编的人来说，内涵远并不是字面上那么直观。所以补充了一下我自己的理解，存在的错误还望指正。
1. 入栈参数，参数寄存器是 x0~ x7。对于 objc_msgSend 方法来说，x0 第一个参数是传入对象，x1 第二个参数是选择器 _cmd。syscall 的 number 会放到 x8 里。
2. 交换寄存器中保存的参数，将用于返回的寄存器 lr 中的数据移到 x1 里。
3. 使用 bl label 语法调用 pushCallRecord 函数。
4. 执行原始的 objc_msgSend，保存返回值。
5. 使用 bl label 语法调用 popCallRecord 函数。

补充-猜测-疑问
1. 在缺乏寄存器的硬件设备上，参数通常是存在stack中，如i386，至于到amd64由于寄存器数量增多，就有一定数目的参数保留在x0-x7，x8保留参数数量的意思，我猜测是用于去stack找寻后续的参数。[amd64 and va_arg - Made of Bugs](https://blog.nelhage.com/2010/10/amd64-and-va_arg/)一文中有对可变参数的进一步讲解。
2. 保存lr是至关重要的一步，它决定了hook的方法执行完毕后，能够返回当初调用objc_msgSend的位置，让调用的方法自然执行下去，并感觉不到任何差异。让一切仿佛没有变化的另一个很重要的步骤就是保存原始objc_msgSend调用前和调用后的寄存器环境。在代码里多次save()和load()就是基于此。`__asm volatile ("mov x2, lr\n");`这一步操作就是把lr的值作为第三个参数，传给`pushCallRecord`方法。
3. blr是bl（跳转到指定label时）同时设置了lr寄存器，所以第二部的lr寄存器里的值需要额外保存，以免被其他跳转操作覆盖。代码中将lr保存在堆上的某个地址中。`__asm volatile ("mov x12, %0\n" :: "r"(&before_objc_msgSend)); `之前需要保存x8,x9寄存器，因为其可能导致x8，x9寄存器的改变。
4. 执行原始的objc_msgSend前通过load还原环境。这一步反而是最好理解的。
5. 在popCallRecord时候为的是获取之前存的bl地址，并设置到bl寄存器中，结束调用。

所以能够实现hook objc_msgSend自然也就能够通过selector参数获取方法名，对于需要hotfix的方法进行处理，增加逻辑改变返回值。比如匹配到了加载到的hotfix_method_name，就可以直接通过blr方法跳过原始objc_msgSend，把返回值置空并且恢复lr。这样就成了一个最简单的hotfix,可以应用于某些一点就崩的方法，我们直接跳过，降低崩溃。尽管在业务逻辑上没有修复。减少崩溃率和提升用户体验，还是有一定效果。
```c
uintptr_t before_objc_msgSend(id self, SEL _cmd, uintptr_t lr) {
    static char *replaced = "hotfix_method_name";
    if(strcmp(sel_getName(_cmd), replaced) == 0) {
        return lr;
    }
    push_call_record(self, object_getClass(self), _cmd, lr);
    return 0;
}
```
另外可以尝试在跳过原始方法之后，能够在返回之前，hotfix方法中执行一些业务逻辑，做到返回特定类型的返回值。可以进行一定的尝试。

这个方法具有可行性，但是弊端也是很明显。不如JSPatch之类只针对有问题的方法Hotfix来得高效。毕竟这是在苹果费心写的汇编代码里强加逻辑，所有的message send都要走这么一层逻辑总是浪费了资源。另外，最重要的一点就是苹果对于这个方法的态度，决定它是否有机会得到应用。

### 参考资料
有些参考资料是后期整理时发现的，比如吴凯凯的博客里面做了比我更详细的讲解和实践。叹没有早日看到，就可以少走一点弯路。
- [ARM64 Function Calling Conventions](https://developer.apple.com/library/archive/documentation/Xcode/Conceptual/iPhoneOSABIReference/Articles/ARM64FunctionCallingConventions.html)
- [iOS ABI  吴凯凯的博客](https://wukaikai.tech/2019/06/19/iOS-ABI/)上篇的中文参考
- [Procedure Call Standard for the ARM 64-bit Architecture](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0055b/IHI0055B_aapcs64.pdf)
- [arm64程序调用规则 吴凯凯的博客](https://wukaikai.tech/2019/05/19/arm64%E7%A8%8B%E5%BA%8F%E8%B0%83%E7%94%A8%E8%A7%84%E5%88%99/#%E7%BB%93%E8%AF%AD)上篇的中文参考
- [c++ - Variable argument function: Bad access with va_arg at iOS arm64 - Stack Overflow](https://stackoverflow.com/questions/35536515/variable-argument-function-bad-access-with-va-arg-at-ios-arm64)
- [Exploring AArch64 assembler – Chapter 7](https://thinkingeek.com/2017/03/19/exploring-aarch64-assembler-chapter-7/)
- 最后才发现的汇编学习网站 [Programmed Introduction to MIPS Assembly Language](https://chortle.ccsu.edu/AssemblyTutorial/)
- ARM指令查询，获知汇编指令的含义，对寄存器的影响等。[ARM Information Center](http://infocenter.arm.com/help/index.jsp)
- [Introduction to X86-64 Assembly for Compiler Writers](https://www3.nd.edu/~dthain/courses/cse40243/fall2015/intel-intro.html)