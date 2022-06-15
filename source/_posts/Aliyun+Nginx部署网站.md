---
title: Aliyun+Nginx部署网站
date: 2017-03-05
categories: Nginx
tags: 
    - Aliyun
    - Nginx
#description: 
---

从零开始基于阿里云服务器和nginx搭建网站
<!-- more -->

### 一、购买 阿里云ECS 云服务器
+ 选择 Linux CentOS 类型的
+ Mac下可以直接使用ssh连接服务器：ssh root@47.92.67.199，然后输入密码
 - windows下可以使用软件 SecureCRT

### 二、注册域名（阿里云网站上）
1、注册
2、阿里云管理控制台-域名与网站-云解析DNS-解析-新手引导设置-设置网站解析（设置邮箱解析-可选）
3、备案（比较耗时）

### 三、服务器安装 nginx
指导网站：<http://www.cnblogs.com/liaolongjun/p/5664005.html>
#### 安装nginx
```bash
yum install nginx
```
#### 启动nginx
```bash
service nginx start
```
#### 访问nginx
+ Linux: curl 127.0.0.1
+ Windows | Mac: 浏览器打开 127.0.0.1

#### 绑定域名
```js
server{
	listen    8001;
	server_name    17it.ren;
	location /
	{
		proxy_set_header Host $host;
        proxy_set_header X-Real-Ip $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://127.0.0.1:14802;
    }
}
server{
	listen    8002;
	server_name    17it.xyz;
	location /
	{
		proxy_set_header Host $host;
        proxy_set_header X-Real-Ip $remote_addr;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_pass http://127.0.0.1:14804;
    }
}
```
#### 重启nginx
```bash
server nginx restart
```
#### 访问测试
可依次访问上面绑定的地址： <17it.ren> <17it.xyz>


** .tk .ga域名都是免费的 **
** 然后空间的话，国内的有主机屋，国外的有三蛋 **

