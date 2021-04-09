---
title: 黑苹果系统升级 Hackintosh upgrade to Big Sur from Catalina
date: 2021-04-09 12:00:48 +0800
layout: post
current: post
cover:  assets/images/covers/WechatIMG11.jpeg
navigation: True
tags: [clover hackintosh Big Sur Catalina ]
class: post-template
subclass: 'post tag-getting-started'
author: Drinking
comments: true
---

此次升级把一年前通过Clover安装的黑苹果Catalina系统升级到Big Sur. 期间也花费了不少时间,试了不少错,好歹最后也成功直接升级成功,没有白费力气.这里就简单记录一下升级过程和经验给需要的人留作参考.

### 升级Clover
升级Colver主要参考文章 [How to Update Clover for BigSur Compatibility using OpenRuntime and Quirks](https://www.insanelymac.com/forum/topic/345789-guide-how-to-update-clover-for-bigsur-compatibility-using-openruntime-and-quirks-v5123/) ** 注意逐字逐句阅读 **
主要流程简单概述为
1. 备份当前在用的的EFI文件夹,放到另外的存储设备上
2. 将最新的Clover安装到独立的U盘,升级Clover Configurator
3. 按照文章指引删除无用的EFI和kext文件
4. 从[OpenCore](https://dortania.github.io/OpenCore-Install-Guide/config.plist/#selecting-your-platform)中勾选对应硬件.**注意Clover的Quirks区域中也包含该文章三块配置,文章的图片用颜色区分,以及部分名称映射也有图片标识,反复细读!**
5. 插入U盘,设置BIOS从U盘启动,尝试配置是否能正常进入系统.

#### 踩坑
正常进去系统后发现不能正常关机和重启,影响升级操作.尝试了不下于以下几种网络上查到的方案.
1. ACPI中设置fixshutdown或者清空fix相关的选项
2. 使用slide=0和EmuVariableUefi.efi
3. 逐个尝试AptioMemoryFix.efi,OsxAptioFix2Drv-free2000.efi,OsxAptioFix2Drv.efi,OsxAptioFix2Drv.efi,OsxAptioFix3Drv.efiOsxAptioFix3Drv.efi
4. 逐个替换旧Covler中的kext文件
5. 删除网卡驱动

最后参考文章[ Shutdown & Power Management don't work after Clover 5120+ & OCQuirks](https://www.tonymacx86.com/threads/shutdown-power-management-dont-work-after-clover-5120-ocquirks.304793/page-2)提供的SSDT-PMC.aml文件放置在ACPI>patched文件夹中**得以解决**

#### 安装系统
参考文章[Update Directly to macOS Big Sur](https://www.tonymacx86.com/threads/update-directly-to-macos-big-sur.304629/). 
期间重启时找不到安装入口,需要设置Colver配置,将Gui中Hide Volumn中的preboot删除,这样就可以显示出MacOS install xxx via xxx 的启动项, 一直走这个入口升级,直到完全升级完成, 到最后进入新的BigSur系统.

以上是对这次升级的一次记录,忘能帮助有需要的朋友.