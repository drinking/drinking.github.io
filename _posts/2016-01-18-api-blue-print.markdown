---
title: APIBlueprint To Swift Request
date: 2016-01-18 18:39:48 +0800
layout: post
current: post
cover:  assets/images/welcome.jpg
navigation: True
tags: [APIBluePrint Swift]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---

这是一个个人项目，目标是将描述Web请求的文档自动转换成用Swift实现的网络请求。前面一部分功能已由APIBluePrint完成，我编写的模块复杂将Json格式的文档描述文件转换成Swift代码。

### API Blueprint语言
> API Blueprint is simple and accessible to everybody involved in the API lifecycle. Its syntax is concise yet expressive. With API Blueprint you can quickly design and prototype APIs to be created or document and test already deployed mission-critical APIs.

#### Drafter
将API Blueprint的文档解析成JSON或YAML文件。AST的格式已经启用，以[Refract Parse Result](https://github.com/refractproject/refract-spec/blob/master/namespaces/parse-result-namespace.md)为主。

{% highlight text %}
drafter -t refract -f json blueprint.apib
{% endhighlight %}

#### Drakov
Drakov是一个Mock Server，解析API描述文档为测试提供伪数据服务。
{% highlight text %}
drakov —f blueprint.apib
{% endhighlight %}


#### Swift方面依赖的第三方库
- Alamofire 用于作为网络请求的基础框架
- SwiftyJSON 处理JSON时便利的工具
- Coolie 将JSON转换成Swift中的Model工具

### 进度


**已完成**
- 将JSON描述文件转换成便于使用的Swift对象
- 用Coolie将HttpResponse解析成Model
- Request with Params

**进行中**
- 基于Alamofire构建的基础Web请求服务

**待完成**
- Chain Request
- 增加代理层，处理特殊的转换请求


### 资料索引
- [ApiBluePrint语法](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md)
- [ApiBluePrint示例](https://github.com/apiaryio/api-blueprint/tree/master/examples)
- [Drakov](https://www.npmjs.com/package/drakov)
- [Drafter](https://github.com/apiaryio/drafter)
