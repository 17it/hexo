---
title: css图标 - webfont
date: 2017-03-05
categories: CSS
tags: 
    - CSS
#description:
---

对于网页中很多常见的小图标，我们可以使用图片来代替，但是图片太多不仅很难管理，而且还增加网络请求，下下策。所以会想到webfont。
<!-- more -->
### 引用步骤
- 把常用的图标或者自己个性化的图标制成webfont文件，其中以 16进制的编码表示，至于怎么制作，以后再说。
- 网页中引用该字体，样式中编写 伪元素样式（before,after）

//以下样式是为了兼容很多浏览器
```CSS
@font-face{
	font-family:'XXX';/*
 	给你的自定义WebFont命名 */
	src:url('xxx.eot');
	src:url('xxx.eot?#iefix')format('embedded-opentype'),url('xxx.woff')format('woff'),url('xxxn.ttf')format('truetype'),url('xxx.svg#micon')format('svg');
	font-weight:normal;
	font-style:normal;
	...
}
.icon{
	font-family: 'MyFont';
	speak:none; /* 无障碍阅读所需要的，告诉屏幕阅读器不要读这个字符 */
	font-size:14px;
	font-variant:normal;
	font-weight:normal;
	text-transform: none;
	...
}
.icon_open:before{
	content:"\f001"; /* 相应图标的编码 */
	...
}
```
HTML如下：
```HTML
<a href="open.html"><i class="icon icon_open"></i>Open</a>
```

### ** 拓展：优秀的图标网站 **
1.<https://icomoon.io/app/>
2.<http://ionicons.com/>
3.<http://fontello.com/>
