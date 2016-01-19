---
layout: post
title: 创建你的CocoaPod库简明教程
date: 2015-04-30 21:08:36 +0800
comments: true
categories: 
---

{% highlight text %}
pod lib create [pod name]
{% endhighlight %}
基于模板创建Pod库，会询问一些基本问题，比如是否提供Example，是否提供测试等。执行后会生成一系列文件，其中Pod文件夹中放入你的代码和资源。Example中提供相应的样例代码。

在Example文件夹下`pod install或pod update`来引入或更新你的代码库。之后就是漫长的编码测试阶段。
{% highlight text %}
coding—->debuging—>testing-
^                         |
|——————<——————<——————————-|

{% endhighlight %}

完成代码后，开始准备提交信息。在`.podspec`中完善你的库信息，包括项目描述以及存放代码的github地址。添加[Travis CI](https://travis-ci.org/profile)集成化测试（调用代码Xcode的测试框架）。测试通过的代码在github页面会显示`build passing`。

将代码发布到github上，打Tag并推送到远程仓库。Podspec文件中的version对应git中的tag，在更新时候需要记得统一。
{% highlight text %}
git add -A && git commit -m “Release 0.0.1.”
git tag ‘0.0.1’
git push —tags
{% endhighlight %}

通过`pod lib lint`检测你的Podspec信息是否正常。使用`—verbose`查看详细。`pod spec lint`会联网检测你的版本库和tag状态。

最后注册并发布到[CocoaPod](http://cocoadocs.org/),稍等片刻就可以在上面查到你的开源库了。
{% highlight text %}
pod trunk register
pod trunk push
{% endhighlight %}


#### 参考资料:
1. [Getting setup with Trunk](https://guides.cocoapods.org/making/getting-setup-with-trunk)
2. [Using Pod Lib Create](http://guides.cocoapods.org/making/using-pod-lib-create)
3. [Making a CocoaPod](http://guides.cocoapods.org/making/making-a-cocoapod.html)
4. [Specs and the Specs Repo](http://guides.cocoapods.org/making/specs-and-specs-repo.html)