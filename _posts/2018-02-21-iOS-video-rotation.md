---
title: iOS视频旋转探究
comments: true
---

### 传统视频旋转方案
屏幕旋转是视频类App横屏观看时是一种常见的操作。主流文章的实现是在竖屏界面通过设置UIDevice的方向来旋转当前的ViewController（下文简称VC）。

```objc
[[UIDevice currentDevice] setValue:[NSNumber numberWithInteger:UIDeviceOrientationPortrait] forKey:@"orientation"];
```
这种方法调用了私有方法，并强制改变了设备的方向。且不说私有方法的调用的实现是否合理，主动设置设备方向就容易出现横竖状态不一致的情况。其影响是整个设备包括自己App的所有ViewControllers。

最理想的方案是只针对当前VC旋转，不影响其他的。苹果官方给出的方案只是预设好的属性，并且只在VC初次加载时读取生效。之后就除了用上面的UIDevice的方法外，没有其他方式使得新的配置生效。

```objc
//是否支持旋转屏幕
- (BOOL)shouldAutorotate {
    return NO;
}
//支持哪些方向
- (NSUInteger)supportedInterfaceOrientations {
    return UIInterfaceOrientationMaskLandscape;
}
//默认显示的方向
- (UIInterfaceOrientation)preferredInterfaceOrientationForPresentation{
    return UIInterfaceOrientationLandscapeLeft;
}
```


### 一种自然过渡的视频旋转方案

该方法主要参考今日头条提到的一种实现方案——[iOS端一次视频全屏需求的实现](https://techblog.toutiao.com/2017/03/28/fullscreen/)，虽然最终他们选择了神奇的depreciate方法。但我依旧觉得方法二是安全合理去耦合的理想方案。而且他们出现的bug我比较难复现，并且随着iOS版本提升，这个问题也将不复存在。下面进入正题，以图片的形式来说明一下旋转过程：

既然无法改变当前VC方向，那能不能再创建一个仅支持横屏的VC。在视频横竖屏切换时，把竖屏的视频自然过渡到横屏去？答案当然是肯定的。这里我们用到了UIViewControllerAnimatedTransitioning自定义一个transition操作。所以在点击横屏时，其实是自定义的Present操作，视频播放器从前面一个移除，经过一系列变化后添加到横屏VC的过程。

这个旋转过渡效果的核心是——“障眼法”。它的主要步骤如下图所示，退出横屏是一个逆操作的过程，思路一致。
![iOS-video1](/assets/img/2018/iOS-video1.jpeg)

我们截取当前视频的一帧画面并覆盖到Player上面，即上图VideoImage。

进行present操作，是一个预先配置好的横向VC。然而它的坐标系已经发生了变化。如果我们直接把截图 VideoImage按照它原来的位置放进去，会发生上图中间的情况，即图片已经发生了旋转。这不是我们想要的。我们想要在transtion的时候，有个旋转动画，让VideoImage可以自然旋转到全屏。而这个旋转操作是发生在landscape ViewController的里面。所以需要先将图二的VideoImage进行一个逆时针旋转操作。使得从portrait到lanscape过度时用户是无感知的。这里需要注意计算旋转后的位置与之前的完全切合。

![iOS-video2](/assets/img/2018/iOS-video2.jpeg)


在进入横屏VC后，我们将进行另一段transform的动效。让VideoImage可以旋转放大到横屏的位置，如上图中间所示。在这个过程中，已经将真正的VideoPlayer默默添加到新的VC并隐藏在VideoImage下面。最后隐藏VideoImage即可。

最终效果也就如下图所示:
![video-transition](/assets/img/2018/video-transition.gif)


### 其他
1. 视频资源的比例和播放器视图的比例不一致，会出现黑边的情况。这个时候如果带着黑边的截图旋转，也会发生拉伸不一致的情况。所以发现这个问题之后，就针对视频资源的比例，只截取有效图像，旋转时候等比放大。能够基本保证放大后和player的位置一致。

### 总结

- 安全且副作用小。不需调用私有API和对整个设备选择。
- 松耦合。在任何VC中都可以方便添加video player并且进行全屏操作，不用改变当前VC的任何逻辑。
- 这种横屏方式毕竟是一个截图障眼过程，和市面上还是有些差异。比如播放器的控制按钮，进度条什么的并没有截图出来，因为在截图旋转拉伸过程中，这些组件会进行拉伸，实际上在传统的方案中是不会有太大变化。

### 参考资料
- [浅谈iOS视频开发](http://www.cnblogs.com/booksky/p/5213198.html)
- [iOS屏幕旋转的解决方案](https://www.jianshu.com/p/c973817d40c8)
- [iOS端一次视频全屏需求的实现](https://techblog.toutiao.com/2017/03/28/fullscreen/)

