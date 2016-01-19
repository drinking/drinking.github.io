---
layout: post
title: Swift中的String
date: 2015-03-22 22:41:51 +0800
comments: true
categories: 
---

在Swift的项目中涉及String操作时，会发现一些让人无所适从的变化`[NSString length]`方法不见了，`substringWithRange`的参数变成了`Range<String.Index>`等。


由于UTF－8和UTF－16是可变长编码，以往对字符串的操作，在某些情况下不能得到正确的结果。比如我们调用一个emoji表情的`[NSString length]`，得到的长度却2，和我们期待的1不相符。还有相关的索引不准的问题。[NSString 与 Unicode](http://objccn.io/issue-9-1/)给出了这些编码的异同，是篇很好的文章。

苹果出于安全和稳定性的考虑，避免程序员编码上的疏漏，就像引入Optional一样，对String进行了以上的修改。在Swift引入`String.Index`，避免我们通过不准确的整型对字符串进行操作。`String.Index`是`String`的一个内建结构体。实现的successor和predecessor方法帮我们计算索引的正确位置。此外还有个私有的_position属性来标示Index的真正位置。

```swift
extension String : CollectionType{
    struct Index : BidirectionalIndexType, Comparable, Reflectable {
        func successor() -> String.Index
        func predecessor() -> String.Index
        func getMirror() -> MirrorType
    }
}
```

text和text2的startIndex都是0，但是它们调用successor方法的结果不同，结果a占用了一个字符，🎾占用两个。如果我们凭直观来处理，难免会出错。
```swift
var text: String = “abc”
var text2: String = “🎾🏇🏈”
text.startIndex.successor() //return 1
text2.startIndex.successor()//return 2
```

每一个字符串的Index维持自身的一个环境，不能互用。advance是一个处理ForwardIndexType接口的函数，String实现了ForwardIndexType接口，能够自己计算增加的距离。text2计算出来的range，超过text的范围，会引起报错。

```swift
let startIndex = text2.startIndex.successor()
let range = Range(start: startIndex, end: advance(startIndex, 1))
text2.substringWithRange(range) //return 🏇
text.substringWithRange(range) //  error
```

长度的计算是通过countElements函数，不知道具体实现方式。根据Swift的提示，猜测是以O(N)的时间复杂度，以ForwardIndexType的形式循环计算出长度。
> Return the number of elements in x.
> O(1) if T.Index is RandomAccessIndexType; O(N) otherwise.

虽然操作上稍显繁琐，但我们充分运用advance,countElements等方法，能够封装出方便以及更稳定的方法。