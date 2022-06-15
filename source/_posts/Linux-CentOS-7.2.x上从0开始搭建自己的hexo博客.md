---
title: Linux-CentOS-7.2.x上从0开始搭建自己的hexo博客
date: 2017-03-05
categories: Linux
tags: 
    - Linux
    - hexo
#description: 
---

Linux-CentOS-7.2.x上从0开始搭建自己的hexo博客
<!-- more -->

### 一、环境准备：

+ 1、安装nodejs
 - a、启用epel软件库（执行成功后就可以使用yum命令安装扩展软件源中的所有软件了）
```bash
yum install epel-release
```
 - b、yum安装npm，这步本身也会安装nodejs
```bash
sudo yum install npm 
```

+ 2、安装Git
**sudo yum install git - 不使用，因为yum安装的git版本是1.7.1太老了，使用方法2 （centos6之前需要安装依赖包）**
使用以下步骤安装最新版本的git：
  + a、更新yum系统：
```bash
sudo yum update
```
  + b、安装依赖包：
```bash
sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```
  + c、下载git源码并解压缩
```bash
wget https://github.com/git/git/archive/v2.3.0.zip
unzip v2.3.0.zip
cd git-2.3.0
```
  + d、编译安装 - 安装在“/usr/local/git”目录下
```bash
make prefix=/usr/local/git all
sudo make prefix=/usr/local/git install
```
  + e、查看是否安装成功及版本号
```bash
git --vsesion
```
  + *f、如果之前有安装git，这时候可能版本号还是之前旧的，因为她默认使用了/usr/bin下面的git。此时请继续往下看至步骤h*
```bash
whereis git // 查看git所在路径会出现类似：git: /usr/bin/git /usr/local/git /usr/share/man/man1/git.1.gz
```
  + *g、把编译安装的git路径放在环境变量里，替换/usr/bin下面的*
```bash
sudo vim /etc/profile
export PATH=/usr/local/git/bin:$PATH // 在文件的最后一行，添加左边的内容
```
  + *h、使用source命令应用修改*
```bash
source /etc/profile
```

  + i、配置Git
```bash
git config —global user.name “tangyongming"
git config —global user.email “790356596@qq.com”
```

### 二、安装hexo
```bash
npm install -g hexo-cli
```

### 三、建站
+ 1、新建所需文件
```bash
hexo init <folder>
cd <folder>
npm install
```
+ 2、_config.yml 配置网站
<http://www.cnblogs.com/xiaoxuetu/p/hexo-guide.html> 
+ 3、进入博客根目录，执行命令
```bash
hexo new hello-hexo
```
完成后会在 source/_posts目录下多出一个文件  hello-hexo.md，可以编辑它，按照markdown的语法
+ 4、生成静态页面
```bash
hexo g //会生成public目录，里面有生成的静态文件
```
+ 5、浏览博客效果
```bash
cd 到根目录，执行 hexo s // 等同于执行 hexo server
```
访问博客，一般默认地址是 http://localhost:4000/

### 四、至此，可以本地访问了，接下来是时候把他部署了

- 1、部署到github上
  + a、安装自动部署到github的插件
```bash
cnpm install hexo-deployer-git --save
```
  + b、_config.yml中配置
```bash
# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://17it.github.io
root: /
        
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/17it/17it.github.io.git
  branch: master
```
  + c、生成静态文件
```bash
hexo g
```
  + d、发布
```bash
hexo deploy // 发布到github
```
+ 2、部署到自己的域名
待续。。。



