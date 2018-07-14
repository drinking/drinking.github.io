---
title: 薄荷TimeLine的优化
date: 2015-04-10 17:24:47 +0800
layout: post
current: post
cover:  assets/images/welcome.jpg
navigation: True
tags: [TimeLine Optimize]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---


## 简介

>薄荷APP是国内最受欢迎的健康减肥APP,是由上海薄荷信息科技有限公司创立，是中国领先的体重管营商。薄荷科技建立了中国最大最活跃的在线减肥平台已服务上百万的减肥用户。

Timeline是薄荷app主要的展示页面，由于历史原因，一直卡顿不理想。我从一开始就希望能改进这块性能及UI，给用户提供极致的体验。对于性能的优化也一直是我感兴趣的方向之一。现在终于有机会接触这块功能，便饶有兴致得进行一番研究并归纳总结，以便为之后的版本打好基础。

### 工具
*Xcode,Instruments-Time Profiler,KMCGeigerCounter，My Eye*

## 案例

### 1.无意义的IO消耗
这个问题的发现一度让我非常震惊。代码中充斥着类似这样的调用：

```objc
[[SQUtils getValueFromSettingPlist:@“leftSpace.width”] floatValue];
```
这个方法的作用是读取配置文件中的属性信息，包括Timeline和Cell的背景色，字体大小和颜色，间距等信息。方法本身没有做任何缓存，每次取值都需要经过读取、解析、拆分和取值的过程。
```objc
+ (id) getValueFromSettingPlist:(NSString*)key{
		NSString *localizedPath = [[NSBundle mainBundle] pathForResource:kSQScrollSettingsFilename ofType:@“plist”];
		NSData *plistData = [NSData dataWithContentsOfFile:localizedPath];
		id plist = [NSPropertyListSerialization propertyListFromData:plistData mutabilityOption:NSPropertyListImmutable format:&format errorDescription:&plistError];
		NSArray* keys = [key componentsSeparatedByString:@“.”];
		NSDictionary* dict = [NRSimplePlist valuePlist:kSQScrollSettingsFilename withKey:[keys objectAtIndex:0]];
		return dict[keys.lastObject];
}
```
这一方法本身没有什么问题，但是在Timeline的场景下，频繁创建Cell时IO操作的频率简直令人发质。更甚者颜色的ARGB属性都分次读取，我的下巴都要掉下来了。
```objc
SQRGBACOLOR(
[[SQUtils getValueFromSettingPlist:@“CellBackground.color-red”] floatValue],
[[SQUtils getValueFromSettingPlist:@“CellBackground.color-green”] floatValue],
[[SQUtils getValueFromSettingPlist:@“CellBackground.color-blue”] floatValue],
[[SQUtils getValueFromSettingPlist:@“CellBackground.color-alpha”] floatValue]);
```

如果没有特殊的需求，这些属性完全可以硬编码到源文件里。如果需求方期望保留配置文件以便将来在线动态更改，那么应当进行缓存`plist`对象，避免频繁IO操作。再进一步，可以使用单例，使各UIColor、UIFont的对象只构造一次，降低了构造的频率。

### 2.异步处理耗时操作

在TimeLine的正文中，往往会有可跳转的超链接或标签。传统的UILabel并不支持这些属性，可以通过NIAttributedLabel、TTTAttributedLabel等第三方库实现。这些库都是用官方名为`NSDataDetector`的类实现链接解析操作。解析是个比较耗时的操作，因此如NIAttributedLabel提供了异步处理的操作，解析完成后更新相关试图。

```objc
- (void)_deferLinkDetection {
  if (!self.detectingLinks) {
    self.detectingLinks = YES;

    NSString* string = [self.mutableAttributedString.string copy];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
      NSArray* matches = [self _matchesFromAttributedString:string];
      self.detectingLinks = NO;

      dispatch_async(dispatch_get_main_queue(), ^{
        self.detectedlinkLocations = matches;
        self.linksHaveBeenDetected = YES;

        [self attributedTextDidChange];
      });
    });
  }
}
```

我们的代码被没有利用好这样的异步功能，同步解析为降低滑动的流畅性又作出了应有的“贡献”。

### 3.缓存中无谓的性能消耗
缓存本来是性能提升的主要手段，但是不当的实现不能充分发挥缓存的性能优势。
```objc
//消耗占比 118*0.81/2352 = 4% 
- (void)cacheCells:(UITableViewCell*)cell{
    if (!_cellCaches) {
        _cellCaches = [[NSMutableArray alloc] init];
    }
    
    if ([_cellCaches indexOfObject:cell] == NSNotFound) {
        [_cellCaches addObject:cell];
    }

    if ([self.cellCaches count] > 20) {
        [self.cellCaches removeObjectAtIndex:0];
    }
}
```

此处缓存的容量为20，超出之后就会删除最前排的元素。这样的实现在TimeLine下拉的场景中，必然会频繁创建新元素，每缓存一次都会调用`removeObjectAtIndex`方法。而该方法的弊端如官方文档所述*To fill the gap, all elements beyond index are moved by subtracting 1 from their index*.不停移动数组中元素，从而导致无谓的消耗。

```objc
//消耗占比 129*0.81/3494 = 3%
- (void)cacheCells:(UITableViewCell*)cell {
    static NSInteger index = 0;
    if (!_cellCaches) {
        _cellCaches = [[NSMutableArray alloc] initWithCapacity:20];
//Edited on 2015-4-18,之前没有加初始化
		for (int i=0;i<20;i++){
				cellCaches[i] = [NSNull null];
			}
    }

    if ([_cellCaches indexOfObject:cell] == NSNotFound) {
        _cellCaches[index] = cell;
        index = index == 19 ? 0 : ++index;
    }
}
```

我使用了一个会循环移动的索引，指向当前存储得位置，来避免`removeObjectAtIndex`带来的的开销。由对比结果可以看出，尽管降低1%，但仍有3%的时间被消耗。从分析工具中可以看到，是被踢出缓存的对象在调用`cxx_destruct`方法进行内存释放。相关知识点可以参考[这篇文章](http://blog.sunnyxx.com/2014/04/02/objc_dig_arc_dealloc/)。

### 4.cell的复用

在iOS开发实践中通常对UITableView的cell采取复用机制。因为对象的创建通常伴随着一定的开销，当cell过于复杂时尤甚。在原来的实现中，我们可以看到复用代码的使用。但是由于同时使用自定义的cache（第三点所讲），导致实际上`dequeueReusableCellWithIdentifier`是没有任何意义的。自定义的cell的配置是在初始时进行，并不支持复用。如果不是在cache中就得重新创建。因而，在不断下拉的过程中，实际上是都是在创建新的cell。
```objc
    ONESQScrollCellViewMixVotesWithDestroyButton *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier];

    ONESQCellModelExtra* model = [self.cells objectAtIndex:indexPath.row];
    if (model) {
        cell = (ONESQScrollCellViewMixVotesWithDestroyButton*)[self loadCellFromCache:model.index];
    }

```
所以，为了实现cell的复用，我必须重构cell使其可配置。在初始化时通过`buildContentView`创建Cell中的视图元素。新增的`configure`方法针对model实现视图的配置，包括UILabel高度的计算和ImageView宽高的设置和相关内容的更新等。cell可配置之后，就可以很自然地运用系统的复用机制。

```objc
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
  static NSString *CellIdentifier = @“SQTimelineCell”;
	ONESQScrollCellViewMixVotesWithDestroyButton *cell = (ONESQScrollCellViewMixVotesWithDestroyButton *)[tableView dequeueReusableCellWithIdentifier:CellIdentifier];
  ONESQCellModelExtra* model = [self.cells objectAtIndex:indexPath.row];
	if (cell == nil) {
        cell = [[ONESQScrollCellViewMixVotesWithDestroyButton alloc] init:model reuseIdentifier:CellIdentifier];
        cell.delegate = self;
	}
  [cell configure:model];
	return cell;
}
```

在实现了真正的cell复用后，第三点中的cache重要性就大大降低。最终被我弃用。

#### 复用所引发的问题
复用后TimeLine的流畅性有了显著的提升。但是在快速滑动时，出现了莫名的崩溃。经过分析得出是由第二条中的detectingLinks引起的。detectingLinks针对富文本字符串进行分析，是一个异步过程。由于Timeline的快速滑动，复用的cell被新的字符串替代，而分析完成后需要根据分析结果对字符串颜色字体等进行渲染。由于前后字符串的不一致，最终出现了outOfRange的溢出异常。

### 5.弃用autoLayout
为了使cell能够适配不同尺寸屏幕，在重构过程中我使用了自己比较熟悉和依赖的autoLayout布局框架Masonry。然后自己没有认识到autoLayout的性能为题。autoLayout的本质是计算各视图之间的一元二次方程，而当view嵌套过多时，计算的时间是成指数级增长，[文章](http://pilky.me/36/)给出了相关的测试数据。

在遇到autoLayout的瓶颈后，立即转向最传统也最高效的setFrame布局方式。最后用KMCGeigerCounter对比两种实现。使用autoLayout的情形，丢帧率达到20%左右。而直接通过计算和setFrame的方式，丢帧率很少超过10%。性能的提升可见一斑。AutoLayout是把双刃剑，我们需要根据不同的场景来进行适当取舍。

### 总结
经过一周左右的重构后，TimeLine终于可以流畅地呈现。这其中遇到的性能问题都可以尝试用缓存、异步处理、cell复用以及弃用低效的实现，这些都是很基础的方案。性能优化是一个权衡的过程，比如复用了cell就不再适用detectingLinks方法。比如为了性能弃用简单的autoLayout实现。

尝试使用优秀的高性能框架比如Facebook的AsyncDisplayKit，其异步渲染工作，能更容易应对复杂的场景，是开发者的福音。

优化是一个永不止步，除了性能还有UI优化、业务逻辑优化等多方面。有些能看出来，有些则不那么明显。希望在之后的编码过程中能进一步优化自己的代码。开发优质的产品。