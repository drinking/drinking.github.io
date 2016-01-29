---
layout: post
title: APIBluePrint实践
date: 2016-01-18 18:39:48 +0800
comments: true
categories:
---

### API BluePrint语言
以Markdown格式写就的API描述文档，语法[参考链接](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md)，示例[参考链接](https://github.com/apiaryio/api-blueprint/tree/master/examples)

### Drakov
[Drakov](https://www.npmjs.com/package/drakov)是一个Mock Server，解析API描述文档为测试提供伪数据服务。`-t`选择解析成`ast`（已弃用）还是推荐的 [Refract API Description namespace](https://github.com/refractproject/refract-spec/blob/master/namespaces/api-description-namespace.md)形式的markdown。

{% highlight text %}
// -t 为解析成 Refract API Description namespace
drakov —f blueprint.apib
{% endhighlight %}

### 解析器[Drafter](https://github.com/apiaryio/drafter)
将API Blueprint的文档解析成JSON或YAML文件。文件格式参考[Refract Parse Result](https://github.com/refractproject/refract-spec/blob/master/namespaces/parse-result-namespace.md)。

{% highlight text %}
drafter -t refract -f json blueprint.apib
{% endhighlight %}


###解析准备