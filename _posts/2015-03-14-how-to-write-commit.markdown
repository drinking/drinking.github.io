---
title: 书写规范的Git提交说明
date: 2015-03-14 18:22:48 +0800
layout: post
current: post
cover:  assets/images/welcome.jpg
navigation: True
tags: [Git]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---


一直以来我在使用Git进行提交时，书写信息都过于随意。这对于个人来说可能影响不大，但在团队合作中让别人不通过阅读代码就能理解你的意图，对提高工作效率是非常重要的。因此我希望在这方面可以做一些改进，翻阅了相关文章，进行了一些总结。


### 举个栗子

国外的一个项目，第一段是09年的提交。句首字母大小写随意，内容有的冗长不易阅读，有的过于简短难以理解。没有美感。

```shell
$ git log --oneline -5 --author cbeams --before "Fri Mar 26 2009"

e5f4b49 Re-adding ConfigurationPostProcessorTests after its brief removal in r814. @Ignore-ing the testCglibClassesAreLoadedJustInTimeForEnhancement() method as it turns out this was one of the culprits in the recent build breakage. The classloader hacking causes subtle downstream effects, breaking unrelated tests. The test method is still useful, but should only be run on a manual basis to ensure CGLIB is not prematurely classloaded, and should not be run as part of the automated build.
2db0f12 fixed two build-breaking issues: + reverted ClassMetadataReadingVisitor to revision 794 + eliminated ConfigurationPostProcessorTests until further investigation determines why it causes downstream tests to fail (such as the seemingly unrelated ClassPathXmlApplicationContextTests)
147709f Tweaks to package-info.java files
22b25e0 Consolidated Util and MutableAnnotationUtils classes into existing AsmUtils
7f96f57 polishing

```



到了14年提交就意简言概工整多了。看得出说明和代码一样一个持续完善的过程。

```shell
$ git log --oneline -5 --author pwebb --before "Sat Aug 30 2014"

5ba3db6 Fix failing CompositePropertySourceTests
84564a0 Rework @PropertySource early parsing logic
e142fd1 Add tests for ImportSelector meta-data
887815f Update docbook dependency and generate epub
ac8326d Polish mockito usage
```

通过对比容易归纳出规范的特性。

### 提交说明的结构
当提交内容简单时，尽量用的一句话描述"做了什么"。句子结构可以分解为(方括号中为可选):

`动作+组件+[原因-索引]`

* 动作:即提交的行为，句首字母大写。有文章认为只使用Update、Add、Remove三个动作就可以满足使用。但是为了更精准地描述，我还是倾向于使用其它动词，下图有常见的动作用词。每一次提交只应该有一个动作，多个动作请适当拆分。
* 组件:指的操作内容，应该用精准的名称，类名方法名等，方便快速定位。不应该提供宽泛的文件名。
* 原因及索引:必要情况下用精简的语句描述原因，有问题追踪系统的话可以指向相应的ID。

使用脑图我们可以更清晰理解句子的结构
![mindthought]({{ site.url }}/assets/img/3-14/CommitMessage.png)

### 优质说明的要素

[How to Write a Git Commit Message](http://chris.beams.io/posts/git-commit/)面向复杂的提交说明（需要对问题进行详细说明时）给出了7点主要的准则。部分已经应用于上面的简短说明。

* 将概述(Subject)和正文(Body)用空格分离
* 将概述字数限制在50个字符
* 行首字母大写
* 用祈使句描述概括
* 概述句末不需要标点
* 正文每行限制在72个字符
* 正文中说明"问题是什么"，"问题为什么产生"，而不是怎么解决的。

### 参考文章
这篇文章主要归纳自以下三篇文章，难免有疏漏。原文有很高的阅读价值。

* [THE COMMIT MESSAGE STANDARD](http://mikebell.io/the-commit-message-standard/)
* [How to Write a Git Commit Message](http://chris.beams.io/posts/git-commit/)
* [Commit Guidelines](https://gist.github.com/rmccue/daf72eaffe984f988a0a)
* [Keeping Commit Histories Clean](https://www.reviewboard.org/docs/codebase/dev/git/clean-commits/)
