---
title: centos搭建rabbitmq（一）
date: 2018-09-03
categories: Nginx
tags: 
    - Linux
    - Nginx
    - NodeJS
    - RabbitMQ
#description: 
---

centos搭建rabbitmq: centos 6
<!-- more -->

### 一、安装 erlang
前情提要：可以直接通过yum install erlang 安装erlang，但是安装的版本很低，当安装rabbitmq的时候会提示需要erlang版本>=19.3，这TM就尴尬了，所以直接图简单安装erlang的办法是不可行的 。

+ 说说安装时候遇到的坑，由于系统是centos6，比较老的版本，使用 erl -version 查看版本的时候发现系统原本没有安装erlang，即使很low的版本。一时手抖，直接用 yum install erlang 安装了，安装过程倒是很顺滑。但是它安装的却是很低的版本（B04之类），在安装最新版本的 rabbitmq 的时候会提示版本过低。

+ 填坑1：想着直接用 yum update erlang ，提示已是最新版本（因为系统里yum的包里面的erlang），放弃...

+ 填坑2：查看rabbitmq官方文档（http://www.rabbitmq.com/install-rpm.html#downloads），说可以安装 rabbitmq团队diy的零依赖版本（Zero-dependency Erlang from RabbitMQ），大致意思也就是简化版本，只有rabbitmq需要的一些库。于是开始按照文档来，参照（https://github.com/rabbitmq/erlang-rpm），通过 Bintray Yum Repositories 方式安装，先新建了 /etc/yum.repos.d/rabbitmq-erlang.repo ，加入了文档里的配置 （选的是 Erlang 20.x on CentOS 6），然后 yum install erlang。此时提示 /usr/... 下面的某些 .x86_64 后缀的文件跟已安装的冲突，安装失败。继续...

+ 继续填坑2：首先想到的是 yum remove erlang，然后再 install，仍然提示冲突。使用 yum list installed | grep erlang，发现下面出现一大片内容，也就是提示冲突的内容。WTF，于是把这些之前旧版本的残留依赖全部 remove掉 （ yum remove erlang-*.x86_64 ），然后再安装就OK了。

总结下正确的安装姿势：
+ 1、查看已安装erlang信息
```bash
erl -version
```
如果有，会显示相应的版本号（我们继续第二步）
+ 2、如果已安装版本过低，则先卸载 （如果没有已安装的，跳过此步）
```bash
yum list installed | grep erlang
```
一般情况下会出现很多的依赖包，别心软，都卸载，不然会出现上面的"填坑2"
```bash
yum remove erlang-*.x86_64
```
+3、安装rabbitmq官方推荐的"Zero-dependency Erlang from RabbitMQ"版本
安装官方文档的链接，跳到 https://github.com/rabbitmq/erlang-rpm ，有两种安装方式 YUM && RPM，我们选 YUM 的 Bintray Yum Repositories 方式安装。跟着文档一步一步来就OK（本文选用 To use Erlang 20.x on CentOS 6 ）。
```bash
vim /etc/yum.repos.d/rabbitmq-erlang.repo
```
```bash
[rabbitmq-erlang]
name=rabbitmq-erlang
baseurl=http://dl.bintray.com/rabbitmq/rpm/erlang/20/el/6
gpgcheck=1
gpgkey=http://dl.bintray.com/rabbitmq/Keys/rabbitmq-release-signing-key.asc
repo_gpgcheck=0
enabled=1
```
注：原文档中baseurl 和 gpgkey 是https的链接，有可能会报错（没找到原因），改成http链接就可以了。
```bash
yum install erlang
```
到此 erlang 就安装成功了，我们可以用以下命令查看下版本（Erlang (SMP,ASYNC_THREADS,HIPE) (BEAM) emulator version 9.3.3.3）
```bash
erl -version
```

### 二、安装 rabbitmq
去 http://www.rabbitmq.com/download.html 下载对应版本的包，本文使用 https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.7/rabbitmq-server-3.7.7-1.el6.noarch.rpm 版本举例。

+1、下载对应包
```bash
wget https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.7/rabbitmq-server-3.7.7-1.el6.noarch.rpm
```

+2、在下载目录，直接yum安装就行（注意install后面参数为下载的包全名）
```bash
yum install rabbitmq-server-3.7.7-1.el6.noarch.rpm
```

到此 rabbitmq 安装成功。

### 三、基本使用命令

+1、启动RabbitMQ服务
```bash
service rabbitmq-server start
```
+2、状态查看
```bash
rabbitmqctl status
```
+3、启用插件
```bash
rabbitmq-plugins enable rabbitmq_management
```
+4、重启服务
```bash
service rabbitmq-server restart
```
+5、添加帐号:name 密码:passwd
```bash
rabbitmqctl add_user name passwd
```
+6、赋予其administrator角色
```bash
rabbitmqctl set_user_tags name administrator
```
+7、设置权限
```bash
rabbitmqctl set_permissions -p / name ".*" ".*" ".*"
```

** 刚安装完成的时候，启用插件，浏览器里输入 http://xx.xx.xx.xx:15672 ，然后用默认的用户名和密码 guest guest 登陆的时候发现不成功（提示该用户仅能在localhost域名下登陆）。此时可以新建一个用户，并且给它赋予管理员权限，然后登陆就OK。
```bash
rabbitmqctl add_user admin admin
rabbitmqctl set_user_tags admin administrator
```

本文重点参考： https://www.jianshu.com/p/f54dc259a9ed
             https://github.com/rabbitmq/erlang-rpm
             http://www.rabbitmq.com/install-rpm.html#downloads
             https://segmentfault.com/a/1190000010832825

#### node项目中使用rabbitMq示例请参照下一节。

