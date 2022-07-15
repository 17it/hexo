---
title: uni-app项目优化小结
date: 2022-07-15
categories: UniApp 小程序
tags: 
    - 小程序
    - UniApp
#description: 
---

用uni-app开发的小程序，主包打包后20xx kb了，一直在超限边缘来回试探，每次新加页面都提心吊胆的。
<!-- more -->
直入主题，开始优化，以下是一些优化方案~

#### 一、减小vendor.js

发现打包后vendor.js高达700多k，借助webpack-bundle-analyzer查看依赖文件发现，项目里引入了moment.js，然后它引入了很多的本地化语言包，占了很大体积(256k)。

##### 1.引入webpack-bundle-analyzer
参照：https://www.cnblogs.com/chiang28/p/10044010.html 

##### 2.webpack打包精简没用的其他语言包
参照：https://blog.csdn.net/sinat_17116395/article/details/81013243

##### 3.看效果
todotodotodo

#### 二、分包

通过第一步，我们就能看到每个文件大小占用情况，所以把那种占用体积大并且不是tarBar里配置的页面进行分包。

uni-app分包文档：
    https://uniapp.dcloud.io/collocation/manifest.html#%E5%85%B3%E4%BA%8E%E5%88%86%E5%8C%85%E4%BC%98%E5%8C%96%E7%9A%84%E8%AF%B4%E6%98%8E
微信小程序官方分包文档：
    https://developers.weixin.qq.com/miniprogram/dev/framework/subpackages/basic.html

+ 记得加上preload，这样不会出现点子包了卡顿的情况：https://uniapp.dcloud.io/collocation/pages.html#preloadrule

##### 常见问题

+ 1.可能会报错 pages/abc/abc 不应该在分包 subPackages['1']中
    原因是pages/abc/abc可能是在tarBar配置里，然后在subPackages['1']中的root配置的是pages/abc。把pages/abc/abc移走，不跟子包一个目录即可

##### 分包配置示例

```bash
"subPackages": [
    {
        "root": "pages/mine",
        "pages": [
            {
                "path": "userAbout/index",
                "style": {
                    "backgroundTextStyle": "dark",
                    "navigationBarTitleText": "账号信息"
                }
            },
            {
                "path": "feedback/index",
                "style": {
                    "backgroundTextStyle": "dark",
                    "navigationBarTitleText": "意见反馈"
                }
            }
            ...
        ]
    }
],
"preloadRule": {
    "pages/center/center": {
        "network": "all",
        "packages": ["pages/mine"]
    }
}
```

#### 三、看整体效果

todo

经过这两步，整体效果还是不错的，主包从最开始的20xx，直接降到了14xx~ 后续继续优化
