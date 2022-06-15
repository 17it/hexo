---
title: yarn和npm对比
date: 2020-09-08
categories: NODE JS
tags: 
    - NODE
    - JS
#description: 
---

之前开发项目一直使用的npm，没出啥篓子，直到今天...
<!-- more -->
事情是这样的：
 + 项目框架： vue, vant
 + 开发时使用：npm
 + 测试环境部署：gitlab + hook（npm）
 + 正式环境部署：walle（yarn）
    
事故：项目最开始是用的npm包管理，安装的vant版本是^2.8.4，本地开发和测试环境都正常，上线到正式环境后，出现一个bug。找了半天，发现是vant最新更新到了2.10.5，导致的一个bug。查看测试环境和正式环境对应的vant版本，发现测试环境是2.8.4，而正式环境已经变成了最新的2.10.5。比较诧异为什么会这样，于是有了下面的探索...

#### 一、yarn、npm对比

可以参照 https://stackshare.io/stackups/npm-vs-yarn 这篇文章，大致可以归纳为下面几点：
 + yarn 安装速度 快于npm
 + yarn self-update 支持yarn自身升级，npm不支持
 + yarn 支持离线下载之前安装过的包，因为yarn是把所有的包放到用户磁盘，而不是单个项目
 + yarn 支持平行安装，npm只是串行，这也是yarn安装速度更快的原因

#### 二、package-lock.json  yarn.lock

一个项目肯定会多人同时开发，因此每个人install的包版本可能会不同，此时会导致各种奇怪的bug。为了使大家机器上的package版本一致，npm引入了package-lock.json，yarn引入了yarn.lock。它俩的原理都差不多，都是每次install后会在各自的lock文件里注明各个包的版本、下载链接，然后再其他人电脑上Install的时候，会优先用lock文件里的版本。但是它们也有区别：如下。

#### 三、实验

1.安装一个新的package，以vant@2.8.4为例
```bash
yarn add vant@2.8.4

npm install vant@2.8.4
```
过程都很顺利，安装完成我们对比package.json发现，**_yarn安装的版本前面没有^，也就是yarn是安装的指定版本，而npm安装的前面带有^_**。

2.重新执行一遍install
```bash
yarn 

npm i
```
此时，两边表现正常，都还是2.8.4，因为有lock文件的限制。

3.修改yarn安装的包版本号，在2.8.4前面加上^，变成^2.8.4，再执行安装
```bash
yarn
```
我们发现，_**node__modules里面和yarn.lock的vant的版本变成了最新的2.10.6，但是package.json还是^2.8.4**_。这就是开头讲到的那个线上bug产生的原因了。由于项目前期是通过npm安装的，所以vant的版本为^2.8.4。然后测试环境部署用的npm，加上package-lock.json的限制，测试环境node_modules里面仍然是2.8.4版本。而正式环境是用yarn部署的，就自动更新成最新版本的2.10.6了。

到此，我们大概就明白了两者的包管理流程，以及对应的lock文件的作用了：
+ **yarn安装的时候会指定版本(前面不加^)，如果加上^，会每次下载最新的版本，同时更新yarn.lock**。
+ npm安装的时候会在版本前加^，然后靠着package-lock.json里来限制不更新为最新版本(**不全对**)。

4.为了验证上面的猜想，我们继续试验：

  4.1、删除package-lock.json文件，然后
  
```bash
npm i
```
我们发现新生成的package-lock.json以及node_modules里面的vant都是2.8.4。咦？是不是我们上面的猜想错了？

  4.2、我们继续，保留package-lock.json，删除node_modules里面的vany目录，然后npm i。

我们发现新生成的package-lock.json以及node_modules里面的vant都是2.8.4。
  
  4.3、我们再进一步：，**同时删除package-lock.json和node_modules里的vant目录，然后npm i**。
  
至此，我们发现node_modules里和package-lock.json里都变成了最新的2.10.6。所以我们可以确定的是，_**如果仅删除package-lock或者仅删除node_modules里面的vant目录，npm i的时候，不会进行升级**_。

* 注：可能各个版本的npm有差异（上面实验时npm的版本是6.11.3）

5.继续把两个项目里package.json里面vant的版本都更改成2.10.6，然后install，两者都升级成2.10.6。这里都在意料之中。

6.我们再近一步，上面5中已经升级成最新的2.10.6了，我们现在来进行降级处理。
+ 1、手动改package.json里的版本为固定版本2.8.4（注：前面没有^），然后执行install。（结果：yarn降级成功，npm仍然是最新的版本）。
+ 2、继续删除package-lock.json文件，然后install，发现npm仍然是最新的。
+ 3、删除package-lock.json文件和node_modules里vant目录，然后install，发现降级成功。

#### 四、结论
+ yarn以package.json为准，所以yarn add xxx@xxx的时候，在package.json里生成的是固定版本（前面不带^），依次来保证每个开发者安装的包版本一致。
+ npm同时受package-lock.json和node_modules目录的影响，所以npm i xxx@xxx的时候，在package.json里生成的是向后兼容的版本（前面带^）。因此，为了保证每个开发者安装的版本一致，**我们需要把package-lock.json加到git版本管理里**。

#### 五、常见场景
对于比较稳定可靠的package，不论是哪种方式，都影响不大，比如ant-design。
对于稍微小众一点的package，有可能小版本的升级会出现bug，这种情况下，最好是固定一个版本，不让随意升级。此时有两种方案：
+ 1.通过yarn add来安装，安装后默认指定固定版本，后续就算你删除node_modules或yarn.lock，再重新add，都不会自动升级。
+ 2.通过npm i来安装，此时package.json里不是固定的版本，因此就必须保证package-lock.json和node_modules不会同时删除后再install（**实际中很容易这么操作**）。如果这样操作后，会安装最新的包，并且package.json里还是显示的旧版本，会有点莫名其妙。

  2.1、当然，你也可以在npm i xxx@xxx之后，手动把package.json里的兼容版本改成固定版本（去掉^），但经验告诉我们，一般不会那么做。
  
  
** 题外话：在过程中，遇到一个问题，就是删除package-lock.json后，再重新npm install生成的package-lock.json并没有自动加入git版本管理（项目里的.gitignore里没有加上)。最后发现是全局把package-lock.json加入gitignore里了，参照：https://cloud.tencent.com/developer/article/1478104 