---
title: Linux-CentOS-7.2.x下安装mysql（Navicat）
date: 2017-03-05
categories: Linux
tags: 
    - Linux
    - mysql
    - Navicat
#description: 
---

Linux-CentOS-7.2.x上安装mysql
<!-- more -->

### 方法一、使用yum
CentOS7的yum源中默认好像是没有mysql的。为了解决这个问题，我们要先下载mysql的repo源。

+ 1.下载mysql的repo源
```bash
$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```

+ 2.安装mysql-community-release-el7-5.noarch.rpm包
```bash
$ sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm
```
安装这个包后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo，/etc/yum.repos.d/mysql-community-source.repo。

+ 3.安装mysql
```bash
$ sudo yum install mysql-server
```
根据步骤安装就可以了，不过安装完成后，没有密码，需要重置密码

+ 4.重置密码
重置密码前，首先要登录
```bash
$ mysql -u root
```
登录时有可能报这样的错：ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘ (2)，原因是/var/lib/mysql的访问权限问题。下面的命令把/var/lib/mysql的拥有者改为当前用户：
```bash
$ sudo chown -R openscanner:openscanner /var/lib/mysql
```
然后，重启服务：
```bash
$ service mysqld restart
```
接下来登录重置密码：
```bash
$ mysql -u root
mysql > use mysql;mysql > update user set password=password(‘123456‘) where user=‘root‘;mysql > exit;
```

+ 5.需要更改权限才能实现远程连接MYSQL数据库
可以通过以下方式来确认：
```bash
root#mysql -h localhost -uroot -p
Enter password: ******
Welcome to the MySQL monitor.   Commands end with ; or \g.
Your MySQL connection id is 4 to server version: 4.0.20a-debug
Type ‘help;’ or ‘\h’ for help. Type ‘\c’ to clear the buffer.
mysql> use mysql; (此DB存放MySQL的各种配置信息)
Database changed
mysql> select host,user from user; (查看用户的权限情况)
mysql> select host, user, password from user;
+-----------+------+-------------------------------------------+
| host       | user | password                                   |
+-----------+------+-------------------------------------------+
| localhost | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
| 127.0.0.1 | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
| localhost |       |                                            |
+-----------+------+-------------------------------------------+
4 rows in set (0.01 sec)
```
由此可以看出，只能以localhost的主机方式访问。
解决方法：
```bash
mysql> Grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
(%表示是所有的外部机器，如果指定某一台机，就将%改为相应的机器名；‘root’则是指要使用的用户名，)
mysql> flush privileges;    (运行此句才生效，或者重启MySQL)
Query OK, 0 rows affected (0.03 sec)
```
再次查看。。
```bash
mysql> select host, user, password from user;
+-----------+------+-------------------------------------------+
| host       | user | password                                   |
+-----------+------+-------------------------------------------+
| localhost | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
| 127.0.0.1 | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
| localhost |       |                                            |
| %          | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
+-----------+------+-------------------------------------------+
4 rows in set (0.01 sec)
```
+ 6.在Mac或者windows上安装navicat for mysql，然后远程来连接Linux上的数据库。
**由于navicat需要图形化的界面，所以如果直接在Linux系统上安装Navicat，还需要安装插件wine（此插件在Linux下很容易出错，详见下面方法二）。**
<img src="http://onm9ileaw.bkt.clouddn.com/hexo/navicat.png" alt="Navicat连接远程mysql" title="Navicat连接远程mysql">

### 方法二、自己下载安装包安装（不推荐，容易出错）
+ Centos6.7安装Navicat：<http://blog.csdn.net/u010824591/article/details/50496553>
+ LINUX系统安装MYSQL命令：<http://www.cnblogs.com/phpxiebin/p/4988156.html>
+ 初始化mysql数据库提示缺少Data:dumper模块：<http://blog.sina.com.cn/s/blog_694864e60102vaij.html>
+ Navicat下载链接：<https://www.navicat.com/download/navicat-for-mysql>

### 扩展：Linux权限小知识
```js
-rw-------   (600) 只有所有者才有读和写的权限 
-rw-r--r--   (644) 只有所有者才有读和写的权限，组群和其他人只有读的权限 
-rwx------   (700) 只有所有者才有读，写，执行的权限 
-rwxr-xr-x   (755) 只有所有者才有读，写，执行的权限，组群和其他人只有读和执行的权限 
-rwx--x--x   (711) 只有所有者才有读，写，执行的权限，组群和其他人只有执行的权限 
-rw-rw-rw-   (666) 每个人都有读写的权限 
-rwxrwxrwx   (777) 每个人都有读写和执行的权限
```

给付权限的方法：
```bash
chmod 777(a+rwx) <folder or filename>
```


