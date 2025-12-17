---
title: proxy代理
date: 2021-03-16
categories: web
tags: 
    - web
    - proxy
#description: 
---

在项目开发中，一般情况是可以localhost调试。但是遇到以下的情况，我们只能通过测试环境/正式环境进行调试，为了更好的定位问题，我们需要把测试/正式环境连到自己本地的代码进行断点（也就是代理）。
1. 测试/线上环境有问题，本地没毛病
2. 需要用到域名的开发，比如H5里面微信授权登录
<!-- more -->

### 零、之前的解决方案
之前是通过修改服务器上nginx的配置，利用反向转发ssh -vnNT -R，把测试/线上的代码打到自己本地进行联调，主要配置如下：
+ 服务器上的nginx配置反向转发（放开下面#注释掉的三行配置）
```bash
server {
    listen       80;
    server_name h5-user-dev.gmtech.top;

    location / {
        root /home/work/www/fe/proprietor_h5/dist;
        proxy_set_header Host $host;
        index  index.html;
        try_files $uri $uri/ /index.html;

        location = /index.html {
            add_header Cache-Control "no-cache, no-store";
        }
    }

   #location ~ {
   #   proxy_pass http://127.0.0.1:7688;
   #}
}
```

+ 然后在本地终端里连接（192.168.238.178为服务器ip，需要输入服务器root账户的密码）
```bash
ssh -vnNT -R 7688:localhost:8001 root@192.168.238.178
```

此方法配置倒是很方便，不论是http还是https的访问，都打到了自己的机器上。但是，有两个很突出的弊端：
1. 只能同时到一个开发者的本地机器，如果多人同时开发就不适用了
2. 由于本地代码处于开发阶段，极其不稳定，测试同学没办法同时进行测试了

自然而然，我们想到的是本地进行代理，此处需要分两种场景：chrome上开发后台系统 和 微信开发者工具上开发H5
+ chrome上开发后台系统：要在chrome上进行代理挺简单，~~直接在"扩展程序"里安装代理插件，最常见的比如Simple Proxy，然后配置xxx.xxx.com代理到本地的开发环境127.0.0.1:8000（但是此方法不能配置https）~~ (此插件已停用，如需要可参考另一篇文章《手搓浏览器插件》)

+ 微信开发者工具开发H5：很遗憾，微信开发者工具不支持安装扩展程序；但是网上搜索说可以用Charles的Map Route，准备一试，打开Charles，然后发现开发者工具里的请求根本不走Charles，至此，放弃

** 到此，总结下：如果不用之前的解决方案反向转发，则chrome上可以通过插件进行代理，但是不支持https；微信开发者工具上没办法代理。

### 一、较完美的解决方案
思考：我们要实现的效果是本地访问网站的时候（不管事chrome还是微信开发者工具）走代理，其他人访问还是正常走服务器。

既然我们是拦截自己电脑，那么我们可以用系统的hosts，然而，系统的hosts代理没办法指定本地的端口，而且本地还没有https。

那接下来的问题就是处理本地代理支持端口（nginx反向代理）和https（自制证书并信任，步骤3和步骤4都是自制https证书，任选一种即可，推荐步骤4）。

### 二、实操

#### 1、本地配置hosts
可以直接修改系统 /etc/hosts 文件，添加配置：
```bash
127.0.0.1 h5-user-test.gmtech.top
127.0.0.1 h5-user-dev.gmtech.top
```

<img src="/proxy代理/图片1.png" width="50%">

#### 2、本地安装并启动nginx
+ 安装nginx：https://www.cnblogs.com/yy136/p/12690225.html
+ 设置开机自启动：https://www.cnblogs.com/kamback/p/8989822.html
+ 配置nginx：
```bash
server {
    listen       80;
    server_name  h5-user-dev.gmtech.top;

    location / {
	    proxy_pass http://127.0.0.1:8001;
    }
}
```

有了以上两步，我们就可以通过代理访问测试/线上环境的http了，此时，如果访问https，还是走不通，会出现下面报错；

<img src="/proxy代理/图片2.png" width="50%">

#### 3、mac自制证书(openssl)
参考：https://www.cnblogs.com/will-space/p/11913744.html
+ 3.1 生成根密钥
```bash
openssl genrsa -out cakey.pem 2048
```

+ 3.2 生成根CA证书
```bash
openssl req -x509 -new -key cakey.pem -out cacert.pem -days 3650
```

+ 3.3 创建证书请求
```bash
openssl genrsa -out http.key 2048
openssl req -new -key http.key -out http.csr
```

+ 附加用途
此步不可忽略，如果不加，在chrome里会报不能识别证书通用名称的错：NET::ERR_CERT_COMMON_NAME_INVALID

a. 新建http.ext
```bash
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName=@SubjectAlternativeName

[ SubjectAlternativeName ]
DNS.1=*.gmtech.top
DNS.2=xxx.xxx.com
```

b. 签发证书
```bash
openssl x509 -req -in http.csr -CA cacert.pem -CAkey cakey.pem -CAcreateserial -out http.crt -days 3650 -sha256 -extfile http.ext
```

至此，会生成http.crt和http.key两个文件，我们将之配置到本地nginx配置里面：
```bash
server {
    listen       80;
    listen	 443 ssl;
    server_name  h5-user-dev.gmtech.top;

    ssl_certificate      /usr/local/etc/nginx/crt/http.crt;
    ssl_certificate_key  /usr/local/etc/nginx/crt/http.key;

    location / {
    	proxy_pass http://127.0.0.1:8001;
    }
}
```

<img src="/proxy代理/图片3.png" width="50%">

+ 3.4 信任证书
我们需要把证书加入到本地keychain里面，并添加"始终信任"：

a. 找到刚才证书生成目录下的 cacert.pem 文件双击，添加到keyChain的证书里，右键选择"显示简介"

b. 找到信任 -> 使用此证书时: -> 选择始终信任

<img src="/proxy代理/图片4.png" width="50%">

c. 到此，可以通过本地代理访问线上http、https地址了。在chrome上地址栏会提示不安全，忽略即可；但是在开发者工具上就可以畅通无阻了，说明微信开发者工具赶chrome还是有不小的差距啊。

#### 4、mac自制证书(mkcert)
参考：https://github.com/FiloSottile/mkcert
+ 4.1 安装mkcert
```bash
brew install mkcert
```

+ 4.2 生成证书
```bash
mkcert -key-file key.pem -cert-file cert.pem "*.gmtech.top" 127.0.0.1 localhost
```

+ 4.3 配置到nginx (使用mkcert生成的证书会自动信任；生成的是两个.pem文件，跟openssl生成的文件后缀不一样，但在nginx里配置一样)
```bash
server {
    listen       80;
    listen	 443 ssl;
    server_name  h5-user-dev.gmtech.top;

    ssl_certificate      /usr/local/etc/nginx/crt/crt.pem;
    ssl_certificate_key  /usr/local/etc/nginx/crt/key.pem;

    location / {
    	proxy_pass http://127.0.0.1:8001;
    }
}
```

** 总结： **
1. 首先通过本地hosts配置让要代理的域名走127.0.0.1
2. hosts配置不支持端口，通过本地启nginx服务进行端口转发
3. 本地nginx要配置https需要证书，可以用openssl自制证书并信任

### 三、手机连电脑进行代理访问（只支持http）
1. 电脑安装spy-debugger
2. 本地开启spy-debugger
3. 手机绑本地spy-debugger代理：本地ip + 端口 9888
4. 访问目标域名

** 注：由于上述证书是mac自制的，在mac上可以添加信任，但是手机上没法信任该证书，所以在微信里访问https的时候，会被阻止 **
