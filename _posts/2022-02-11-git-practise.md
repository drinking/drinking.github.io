---
title: API版本控制的Git流实践
date: 2022-02-11 12:00:00 +0800
layout: post
current: post
cover:  assets/images/covers/1271644035550_.pic_hd.jpg
navigation: True
tags: [Git API Version]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true

---

本文意在梳理多人开发维护API版本的实践.

流程如图

![git flow](/assets/img/2022/1321644590829.jpg)

图中包含共6个分支,4条主要分支,2条临时分支.

五个序号标记点,每个标记点代表一次API版本的变更.

① main正式环境版本号: x.y.1

② feature开发分支独立版本号: x.y.2-SNAPSHOT

③ dev联调分支版本号: DEV-SNAPSHOT

④ sit测试分支版本号: SIT-SNAPSHOT

⑤ main正式环境版本号: x.y.2

git flow流程的解释

1. 从main切出分支进行新功能开发
2. 在API发生变更需求时,在feature分支将API版本号改成自己独占的版本号,此处需要工具限定或者内部协调,保证团队所有开发者此SNAPSHOT的认知和变更范围是一致的,不会引入此feature外的变更.
3. 将feature功能合并到dev进行发布联调和自测.次分支是公用分支,所以为了避免不同feature之间对API的变更版本号的混乱,可统一成DEV-SNAPSHOT.feature合并到dev后在dev分支进行DEV-SNAPSHOT的部署更新.
4. 同理来到提测环节,由于SIT分支收到保护,需要通过merge request的形式进行合并.所以需要先从SIT分支切出一个tmp分支,并入feature分支,同时把API的版本号在tmp分支上改为SIT-SNAPSHOT,并部署更新.提交merge quest合并到sit分支.
5. 同理合并sit分支,最终合并到main分支完成变更,同时更新版本号的最终定版.