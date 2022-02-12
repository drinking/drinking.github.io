---
title: API版本控制的Git流实践
date: 2022-02-11 12:00:00 +0800
layout: post
current: post
cover:  assets/images/covers/1381644665262_.pic.jpg
navigation: True
tags: [Git API Version]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true

---

本文意在梳理多人开发场景下,管理API版本的实践.此处API是指在Spring开发过程中,server所依赖的api的版本.其他语言环境如有类似依赖,也可参考.

![git flow](/assets/img/2022/1321644590829.jpg)

文中讨论的流程为了在开发过程中,避免api的冲突.话不多说先上图.图中包含共5个分支,3条常驻分支,一条功能分支和1条临时分支.dev和sit分支的作用可能与概念和实际场景存在差异,此处仅用于举例.

- main分支,对应线上的代码
- feature分支,代表开发新功能的分支
- dev分支,代表测试联调分支,可能有多个feature同时开发并入
- sit分支,持续集成测试介入分支.此处仅用于合并和测试上线的内容,真实场景sit分支可能存在多期内容同时测试,然后通过UAT分支来测实际上线内容

四个序号标记点,代表不同的API版本节点,以1.0.0为例.

① main: 1.0.0 ② feature: 1.0.1-SNAPSHOT ③ dev: DEV-SNAPSHOT ④ sit: 1.0.1

git flow流程的解释

1. 从main切出分支进行新功能开发
2. 在API发生变更需求时,在feature分支将API版本号改成独占的版本号,保证变更范围仅受当前分支影响.此处需要工具限定或者内部协调,保证团队所有开发者认知一致.
3. 将feature功能包含api变更合并到dev.此分支是公用分支,所以为了避免同时开发的feature分支对API的版本号造成混乱,可统一成DEV-SNAPSHOT.并在合并后的dev进行DEV-SNAPSHOT api的部署.囊括此api所有的变更.
4. 当dev分支测试充分后,来到预上线的环节.由于SIT分支受到保护,需要通过merge request的形式进行合并.所以需要先从SIT分支切出一个tmp分支,并入feature分支,解决冲突后,把API的版本号在tmp分支上改为定版的1.0.1,并提交部署做最后上线前的测试.
5. 测试完成准备发线上后,将sit分支以PR的形式并入依旧受保护的main分支.有些公司main分支仅做归档,就会通过UAT或其他分支发布.

以上就是针开发过程中API命名方案的实践.同时如果server依赖多个api,那就需要维护好server与多个api分支的依赖关系,避免上线过程中的遗漏.对于这种情况如有更好的管理方式或者本文的其他疑问均欢迎邮件讨论.