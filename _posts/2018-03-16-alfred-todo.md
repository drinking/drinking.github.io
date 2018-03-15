---
title: 一款Alfred上的To-do List插件
comments: true
---

## 简介
[Alfred](https://www.alfredapp.com/)被誉为MacOS上的效率神器，关于它的介绍太多了，就不累述。

[To-DO List](https://en.wikipedia.org/wiki/Wikipedia:To-do_list)也是众所周之以代办事项形式罗列的清单。也有很多成熟的，需要安装以及付费的软件。而且不一定有相关Alfred快速启动的支持。即使To-do List软件有快捷键支持，可苹果的快捷键在我电脑上，Alfred用到一个，欧路词典占了一个，还有一个留给了输入法，所以到最后很是捉襟见肘。

最基础的Todo-List其实是一个很简单的形式，哪怕用普通的文本就能够实现。Markdown也是支持形如下面格式的渲染:

```markdown
-[ ] 要做的事情
-[x] 完成的事情
```

## 写入To-do List项
所以为什么不能自己实现一个Alfred插件，来以文本的形式来操作呢？实现之前当然看看有没有现成的了。很不幸，要么支持不完善，要么只是第三方软件的支持。所以自己来撸一个咯。不过其中有一篇文章给了最基础的例子[Stupid Simple Alfred Todolist – Adam Gray – Medium](https://medium.com/@dashedstripes/stupid-simple-alfred-todolist-efa551732adb)，证明了这种思路的可行性。实现很简单，获取Alfred的参数，然后通过shell文件即可，一行搞定。

```shell
echo -e — [] {query}\ >> ~/TODO.txt
```

![1521133195667](/assets/img/2018/1521133195667.jpg)

## 勾选项
简单归简单，还是有一些缺陷。最主要的是没有check功能。我们完成一项任务要去勾掉它，这也是To-do List上最有成就感的一个过程。怎么能没有呢？所以第二步，要自己实现一下。check功能的实现思路也简单：
1. 读取To-do List文件。
2. 根据参数匹配到对应的且未完成的行。
3. 将未完成`- [ ]`标记替换为`- [x]`已完成标记。
4. 将文件重新写入

实现这些本身不是什么难事，只是在Alfred的开发环境下，需要依赖它定义的数据结构来进行参数的传递，需要一定的学习成本。好在又有合适的工具可以帮助简化这一步骤——[sindresorhus/alfy: Create Alfred workflows with ease](https://github.com/sindresorhus/alfy)，一个用Node实现的Alfred脚手架，我本来想用Swift实现，怎奈资源还是匮乏，时间不够还需站在巨人肩膀上。alfy帮你定义好了输入输入的结构，你只需要以`alfy.input`和`alfy.output`的形式在js获取和传递就是了，不需要关注底层，非常友好。所以顺着alfy的文档，和用了Node的fs文件操作模块。也是用了几行就实现了check的功能。另外还有`alfy.cache.set('path','/user/typed/path');`针对不同用户的路径设置，文件会存储到相应的位置。

```js
'use strict';
const alfy = require('alfy');
var fs = require('fs');

var filename = alfy.cache.get('path');

fs.readFile(filename, function(err, data) {
	if(err) throw err;
	var array = data.toString().split("\n");
	const results = array.filter(word => (word.toString().indexOf(alfy.input) > -1 && word.toString().indexOf('x') === -1))
						.map(x => ({title: x,arg:x}));
	alfy.output(results);
});

```
实现效果如图，没有参数的时候，会过滤出所有未勾选的选项。

![WX20180316-010346@2x](/assets/img/2018/WX20180316-010346@2x.png)

当有参数时，就快速查找到需要的，然后选择想要的回车，既可以做出更改。
![WX20180316-010527@2x](/assets/img/2018/WX20180316-010527@2x.png)

## 结合MarkEditor
因为Markdown默认支持To-do List格式的渲染，而且我之前发现的一款[MarkEditor](https://www.markeditor.com/?lang=zh_cn)的Markdown软件，既可以支持菜单栏书写，同时能够直接点击`-[ ]`和`-[x]`标签进行选择和反选。非常人性化。

![WX20180316-011258@2x](/assets/img/2018/WX20180316-011258@2x.png)

这样一个能用的`Alfred To-do List workflow`就完成了。

## 其他可以改进之处
- 增加时间选项，根据预设时间，触发提醒。
- 当前只是简单粗暴地针对整个文件读取和写入，当数据量大时，还是有通过数据流方式读取和输出的余地。
- 去重复

## 总结
好像一不小心安利了不少app，工具还是每个人用的最趁手的好，Alfred已经是我工作中不可或缺的一部分，而且越来越喜欢集成不同的功能进来。现在还有个方向是信息获取这块，有人做过通过Alred获取知乎，豆瓣或者股票的消息。如果没有，比如我想获取某直播吧数据，完全可以自己来实现，针对不友好的HTML格式文本就可能中间需要自己解析一下了。
这个插件暂时自己处使用，如果有人有需要的话，可以索取，之后有时间也会去发布一下。

