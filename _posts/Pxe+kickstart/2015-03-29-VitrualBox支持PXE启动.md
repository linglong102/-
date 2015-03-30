---
layout: post
title: VitrualBox支持PXE启动
category: 自动化部署
tags: Pxe+kickstart
keywords: VitrualBox支持PXE启动
description: 
---

>**Data:2015/3/29 6:02:27**

默认情况下VirtualBox下载安装后是不支持PXE启动的，启动的时候是会报下面错误：

	“FATAL:No bootable medium found!System halted.”
- 这是因为缺少pxe扩展包所导致，通过下面这个地址下载扩展包，windows环境中通过双击形式安装。

### 下载地址：

	http://dlc-cdn.sun.com/virtualbox/4.3.26/Oracle_VM_VirtualBox_Extension_Pack-4.3.26-98988.vbox-extpack
- 注：根据版本的不同进入相应的版本目录下下载Oracle_VM_VirtualBox_Extension_Pack-x.x.xx-xxxx.vbox-extpack

![](http://i.imgur.com/pU1nZoo.png)
