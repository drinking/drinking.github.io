---
title: "阅读Chats小记"
date: 2015-02-03 14:38:39 +0800
layout: post
current: post
cover:  assets/images/welcome.jpg
navigation: True
tags: [Swift]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---

[Chats](https://github.com/acani/Chats)是一个用Swift语言写的聊天app的一个Demo，初学者能够从中了解如何构建一个实时通信app的雏形。Demo功能本相对简单，没有实现数据储存和网络连接。我之前有用Objective-C写过聊天应用的经验，打算用Swfit重写一遍。希望通过阅读这部代码能比较不同人的实现方式，找到可以借鉴的实现。

#### 生成气泡

```swift
let bubbleImage = bubbleImageMake()

func bubbleImageMake() -> (incoming: UIImage, incomingHighlighed: UIImage, outgoing: UIImage, outgoingHighlighed: UIImage) {
    let maskOutgoing = UIImage(named: "MessageBubble")!
    let maskIncoming = UIImage(CGImage: maskOutgoing.CGImage, scale: 2, orientation: .LeftMirrored)!

    let capInsetsIncoming = UIEdgeInsets(top: 17, left: 26.5, bottom: 17.5, right: 21)
    let capInsetsOutgoing = UIEdgeInsets(top: 17, left: 21, bottom: 17.5, right: 26.5)

    let incoming = coloredImage(maskIncoming, 229/255.0, 229/255.0, 234/255.0, 1).resizableImageWithCapInsets(capInsetsIncoming)
    let incomingHighlighted = coloredImage(maskIncoming, 206/255.0, 206/255.0, 210/255.0, 1).resizableImageWithCapInsets(capInsetsIncoming)
    let outgoing = coloredImage(maskOutgoing, 43/255.0, 119/255.0, 250/255.0, 1).resizableImageWithCapInsets(capInsetsOutgoing)
    let outgoingHighlighted = coloredImage(maskOutgoing, 32/255.0, 96/255.0, 200/255.0, 1).resizableImageWithCapInsets(capInsetsOutgoing)

    return (incoming, incomingHighlighted, outgoing, outgoingHighlighted)
}

func coloredImage(image: UIImage, red: CGFloat, green: CGFloat, blue: CGFloat, alpha: CGFloat) -> UIImage! {
    let rect = CGRect(origin: CGPointZero, size: image.size)
    UIGraphicsBeginImageContextWithOptions(image.size, false, image.scale)
    let context = UIGraphicsGetCurrentContext()
    image.drawInRect(rect)
    CGContextSetRGBFillColor(context, red, green, blue, alpha)
    CGContextSetBlendMode(context, kCGBlendModeSourceAtop)
    CGContextFillRect(context, rect)
    let result = UIGraphicsGetImageFromCurrentImageContext()
    UIGraphicsEndImageContext()
    return result
}
```

#### 其它

```swift
//用类名来作为复用Cell的标识
tableView.registerClass(MessageSentDateCell.self, forCellReuseIdentifier: NSStringFromClass(MessageSentDateCell))

//最简单的实现键盘消失的方法，有.Interactive和.OnDrag两种不同的形式。
//如需点击时消失键盘，还需额外实现touch事件。
tableView.keyboardDismissMode = .Interactive
```