---
layout: post
title: "iOS源码阅读之TTTAttributedLabel"
date: 2015-01-14 21:49:27 +0800
comments: true
categories: 
---
[TTTAttributedLabel](https://github.com/TTTAttributedLabel/TTTAttributedLabel)是一个功能更为丰富的UILabel,支持`AttributedString`，识别特殊文本（如地址，电话，邮箱以及超链接等），并可以自定义这些文本的点击响应事件。为了了解其链接点击的实现方式，我决定对源码一窥究竟。

先看一下官方给出的给文字添加链接的参考代码。步骤很简单，在自己的Delegate实现中处理URL。

{% highlight objective-c %}

label.enabledTextCheckingTypes = NSTextCheckingTypeLink; 
label.delegate = self; 

label.text = @"Fork me on GitHub! (http://github.com/mattt/TTTAttributedLabel/)"; 

NSRange range = [label.text rangeOfString:@"me"];
[label addLinkToURL:[NSURL URLWithString:@"http://github.com/mattt/"] withRange:range]; 
{% endhighlight %}


`TTTAttributedLabel`的最终实现是基于`Core Text`，而`Core Text`中的字符串都是用`NSAttributedString`来表示。所以在调用`addLinkToURL:withRange`时自然需要在`NSAttributedString`字符串的相应区间添加属性，以区分URL链接，比如设成蓝色。源码可以看到作者是构造`NSTextCheckingResult`来存储range和URL的，这个是为了和自动识别出来的特殊文本统一格式，一并存储到名为`links`这个数组中，作为点击识别时起到重要数据源。

{% highlight objective-c %}

- (void)addLinksWithTextCheckingResults:(NSArray *)results
attributes:(NSDictionary *)attributes
{
NSMutableArray *mutableLinks = [NSMutableArray arrayWithArray:self.links];
if (attributes) {
NSMutableAttributedString *mutableAttributedString = [self.attributedText mutableCopy];
for (NSTextCheckingResult *result in results) {
[mutableAttributedString addAttributes:attributes range:result.range];
}

self.attributedText = mutableAttributedString;
[self setNeedsDisplay];
}
[mutableLinks addObjectsFromArray:results];

self.links = [NSArray arrayWithArray:mutableLinks];
}

{% endhighlight %}

`TTTAttributedLabel`使用`Core Text`完全重写了UILabel的`drawTextInRect`方法。实现`touchesBegan`，`touchesMoved`,`touchesEnded`来处理点击响应。
在触发`touchesBegan`事件后，获取点击字符位置的关键函数是`- (CFIndex)characterIndexAtPoint:(CGPoint)p`，其核心函数依旧是`Core Text`中的方法。

{% highlight objective-c %}

//首先创建一个framesetter对象，CTFramesetter是用来管理文体字体和绘制区域的一个重要的类。
CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString((__bridge CFAttributedStringRef)self.renderedAttributedText);

//创建一个绘制区域,这个方法应该已经将每个字的位置排好
CTFrameRef frame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, (CFIndex)[self.attributedText length]), path, NULL);
//之后获取这个区域的每一行
CFArrayRef lines = CTFrameGetLines(frame);
//遍历每一行，根据触摸点的位置来判断点击文字的索引值。这个索引值也就是字符在字符串中的位置了。
CGPoint relativePoint = CGPointMake(p.x - lineOrigin.x, p.y - lineOrigin.y);
idx = CTLineGetStringIndexForPosition(line, relativePoint);

{% endhighlight %}

获取了字符位置之后，遍历links数组，找到符合区域的链接。

{% highlight objective-c %}

- (NSTextCheckingResult *)linkAtCharacterIndex:(CFIndex)idx {
NSEnumerator *enumerator = [self.links reverseObjectEnumerator];
NSTextCheckingResult *result = nil;
while ((result = [enumerator nextObject])) {
if (NSLocationInRange((NSUInteger)idx, result.range)) {
return result;
}
}

return nil;
}

{% endhighlight %}

最后通过Delegate发送相应的链接给处理函数，就实现了最终的调用。这篇文章省略了很多实现的细节，如想了解还需细看源码以及了解`Core Text`。



###参考

关于`Core Text`的优秀国外优秀的[教程](http://www.raywenderlich.com/4147/core-text-tutorial-for-ios-making-a-magazine-app)。