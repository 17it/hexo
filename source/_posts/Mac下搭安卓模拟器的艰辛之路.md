---
title: Mac下搭安卓模拟器的艰辛之路
date: 2019-07-18
categories: Mac
tags: 
    - Mac
    - 模拟器
#description: 
---

需要在mac上搭建安卓模拟器，用来测试。
<!-- more -->

### 前车之鉴：
毫无疑问，首先想到的是用Android Studio里面的avd manager创建模拟器。兴致勃勃的折腾了半天，安装JDK、配置JAVA环境、安装Android Studio、配置gradle、下载安卓SDK。
最坑的就是下载安卓SDK，选了大概5个安卓版本的SDK，下载完后，一下子把磁盘干掉130G。
    坑一：安装完后，用avd新建模拟器，一路next，装完启动。然后把APP拖进去安装，直接报错，安装不上，提示『INSTALL_FAILED_NO_MATCHING_ABIS』，百度半天，知道大概是因为mac
的CPU是x86的，刚新建模拟器的时候默认选的是x86的镜像。然而APP运行需要arm支持，所以安装不上。
    坑二：重新建一个模拟器，在选择System Image那一步，选『other image』，也就是ABI那一列为"armxxxxx"的镜像。新建完，启动。直接提示『﻿Running an x86 based Android Virtual Device (AVD) is 10x faster.We strongly recommend creating a new AVD.』
意思就是基于x86的镜像会比刚选的arm的镜像快10倍，估算了下，也就是10s和1s的区别，将就用吧。一边吃瓜一边等开机，结果。。。10分钟过去了，还没开完机。。。好不容易开机了，APP安装进去，运行卡的要死，操作一下，基本都不动了。。。欲哭无泪啊
    备注：在windows电脑上，应该不会存在上面的问题。

### 摸索之路：
用AS里的avd建模拟器的方法算是废弃了，找了半天，看都推荐用 Genymotion ,说速度超快，接下来进入正题...
## 一、下载安装VirtualBox（版本 VirtualBox-6.0.10-132072-OSX.dmg，仅做备注，怕以后最新版本出现不兼容的问题）
## 二、注册Genymotion账号
## 三、下载安装Genymotion（版本 genymotion-3.0.2.dmg，仅做备注，怕以后最新版本出现不兼容的问题）
至此，安装完成，新建模拟器，运行，果然很流畅。但还是会有之前那个问题，也就是arm和x86兼容性。但较AS好的点是，可以用ARM Translation进行补丁，建模拟器的时候不用选arm的镜像（好像新建的过程中也没地方可选arm的镜像）。
于是把文章中的ARM Translation下载，拖拽到模拟器里安装，然后重启模拟器，安装自己的APP。果然就能安装上了，一阵窃喜。
但但但但是，运行APP就一直出现"xx应用已停止"的提示，也就是说虽然能安装，但是用不了，shit...
思索良久，想着会不会是ARM Translation版本的问题，直到看到了一篇文章，确认了这个想法。重新下载了模拟器版本对应的ARM Translation，然后就一切大吉了。

反思：既然Genymotion模拟器能用ARM-Translation进行兼容，按道理AS里面也应该可以借鉴，但没找到哪里可以进行设置，后续继续研究...

### 参考文章：
1.mac下AS的搭建以及环境配置： https://blog.csdn.net/weixin_28774815/article/details/81334687
2.Genymotion的安装、使用和注意事项： https://imququ.com/post/genymotion.html
3.Genymotion-ARM-Translation各版本下载： https://blog.csdn.net/GHY2016/article/details/83422620
