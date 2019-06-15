---
title: 代码如诗
date: 2019-06-15 21:39:48 +0800
layout: post
current: post
cover:  assets/images/covers/IMG_3297.JPG
navigation: True
tags: [SDWebImage]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---

好的代码就像诗歌一样，阅读时能够透过代码看到作者的思想，流畅精准、优美得令人陶醉。SDWebImage（以下简称SD）作为iOS平台上使用范围最广的图片加载框架，就是这样一篇优美的作品。本文只是作为阅读之余的一篇小记，不能面面俱到，如有不当还望指正。

## 从一个典型的加载函数开始
loadImageWithURL是SD中最具代表性的函数之一，其囊括了加载一个图片资源的全部流程。以下让我们通过这个流程，去品味代码中的韵味。

### 一、判断URL的有效性，操作是否可以继续执行
除了对url本身格式正确性的判断，SD缓存了失败的url，如果失败过且不具备重试条件的请求，则从一开始直接本地返回false，避免走网络带无谓的消耗。

### 二、获取缓存Key及其设计方案
这里涉及了缓存名的设计。缓存命名采用将url转换成md5并加文件后缀的形式。代码此处考虑到了操作系统对文件名长度的限制，对md5取前16位，计算后缀的长度，如果后缀名过长就舍弃。这也许也是一开始不用url作为key的原因。此外通过代理`cacheKeyFilter`，让使用者有改变Key的能力，比如对同样路径不同时间戳的url其实使用的同样的图片资源，没有必要重复缓存图片，所以在`cacheKeyFilter`用户可以手动去除多余的参数，达到最终生成同一个Key的效果，降低缓存的冗余。
```objc
NSURL *keyURL = [NSURL URLWithString:key];
NSString *ext = keyURL ? keyURL.pathExtension : key.pathExtension;
// File system has file name length limit, we need to check if ext is too long, we don't add it to the filename
if (ext.length > SD_MAX_FILE_EXTENSION_LENGTH) {
    ext = nil;
}
NSString *filename = [NSString stringWithFormat:
                  @"%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%02x%@",
                      r[0], r[1], r[2], r[3], r[4], r[5], r[6], r[7], r[8], r[9], r[10],
                      r[11], r[12], r[13], r[14], r[15], ext.length == 0 ? @"" :                               
                      [NSString stringWithFormat:@".%@", ext]];
```

### 三、读取缓存及其设计方案
但凡知道缓存概念的人，都能说出其设计的核心思想，即先从内存中读取数据，如果没有命中就尝试从硬盘中读取，依旧没有的话，就从网络中请求。再进一步，如果从硬盘或网络中获取到数据，则可以采用LRU一类的算法，把数据缓存在内存中。这样回答固然挑不出什么毛病，但是若要亲自抡胳膊上阵设计一套代码，那就欠缺太多了。
SD中的缓存主要由SDImageCache和SDMemoryCache类实现，SDImageCache负责处理内存和硬盘的读写操作，SDMemoryCache则是封装了NSCache，作为缓存的核心存储单元。其不仅依赖了NSCache的特性，同时内置了以NSMapTable为核心的弱引用缓存，在Memory Warning时释放NSCache中的缓存，而通过NSMapTable引用一些正在被使用且不被释放的资源。尽可能保证资源能够被命中。

`queryCacheOperationForKey`是读取缓存的核心处理逻辑。在内存命中失败后，随即尝试从硬盘中查找，且是最大限度得查找，在默认路径和自定力路径(customPaths)下都尝试查找完整名称和去除后缀文件。此处的customPaths实现上非常简单只是一个路径数组，但是大大扩展了缓存读取的范围，也有效得划分了存储区域。硬盘搜索是相当耗时的操作，所以这部分行为被封装在一个Block里，通过SDImageCacheQueryDiskSync参数来判断使用者想要同步或者异步在自己创建的ioQueue中加载。在Block中又以@autoreleasepool包裹，保证数据资源即使释放。

### 四、网络请求
下载图片的网络请求操作主要交给`SDWebImageDownloaderOperation`处理，包含了图片解码、渐进式下载、身份认证、后台下载等重要功能。通过`NSURLSession`实现网络请求和下载，自己作为其`Delegate`，一并处理其网络请求中的数据验证，解码等工作。具体的细节非常多，就不一一展开。这里提一个比较有意思的点，通过`NSOperation`和自定义的`finished`和`executing`等字段明确当前操作的状态，在此基础上就来避免重复的Operation，或者提高那些高频请求的优先级，让它能在`NSOperationQueue``中优先执行，这在图片繁多的请求场景下，尤其有用。

## 其他设计思想
### 对线程安全的关注
对于可能存在线程安全的数组的操作，都采用了锁支持。无论是数组或者字典。而且对于锁的用途进行了细分，防止不必要的锁等待，浪费计算资源。下例为两个不同的锁直接
```objc
#define LOCK(lock) dispatch_semaphore_wait(lock, DISPATCH_TIME_FOREVER);
#define UNLOCK(lock) dispatch_semaphore_signal(lock);
LOCK(self.failedURLsLock);
isFailedUrl = [self.failedURLs containsObject:url];
UNLOCK(self.failedURLsLock);
[self callCompletionBlockForOperation ...]
LOCK(self.runningOperationsLock);
[self.runningOperations addObject:operation];
UNLOCK(self.runningOperationsLock);
```
除了上述广为人知的线程不安全，还有对其它API的线程安全的考据。
```objc
// NSURLCache's `cachedResponseForRequest:` is not thread-safe, see https://developer.apple.com/documentation/foundation/nsurlcache#2317483
@synchronized (URLCache) {
        cachedResponse = [URLCache cachedResponseForRequest:self.request];
}
```

### 对操作流程状态的把控
几乎每一个流程都有与之相关联的NSOperation类，又NSOperation来提供判断工作状态的基本字段`cancelled` `executing` `finished` `ready`，这样可以避免执行取消的，重复的或者同时的请求执行，最大限度合理使用硬件资源，还没有执行的operation又可以通过`queuePriority`来适当调整请求优先级。


        if (self.options & SDWebImageDownloaderIgnoreCachedResponse) {
            // Grab the cached data for later check
            NSURLCache *URLCache = session.configuration.URLCache;
            if (!URLCache) {
                URLCache = [NSURLCache sharedURLCache];
            }
            NSCachedURLResponse *cachedResponse;
            // NSURLCache's `cachedResponseForRequest:` is not thread-safe, see https://developer.apple.com/documentation/foundation/nsurlcache#2317483
            @synchronized (URLCache) {
                cachedResponse = [URLCache cachedResponseForRequest:self.request];
            }
            if (cachedResponse) {
                self.cachedData = cachedResponse.data;
            }
        }

这篇文章因为中途事情太多，几经中断，导致在编辑器中躺了好几个月，后来想要再拾起也没有当时的连续性了，导致文章有点虎头蛇尾。之后有时间需要再完善下SDWebImage的一个执行流程图，可以更加清晰得看到各功能是如何有机架构起来的。