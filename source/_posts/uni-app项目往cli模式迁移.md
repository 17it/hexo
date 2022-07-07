---
title: uni-app项目往cli模式迁移
date: 2022-07-08
categories: UniApp 小程序
tags: 
    - 小程序
    - UniApp
#description: 
---

uni-app项目往cli模式迁移 TODO
<!-- more -->

1.cli模式run dev比run build后的包大的原因（watch？）
2.cli模式run dev --minimize 后打包慢？ -  勾选『预览及真机调试时主包、分包体积上限调整为4M』来规预览慢的问题（思考：为啥hbuilder加上压缩后打包快？）
3.往cli模式迁移步骤：https://uniapp.dcloud.io/quickstart-cli.html#install-vue-cli


todo:
1.基于cli搭建的项目的分包直接在pages目录配置？
2.zb-table和lime-charts注意事项（zb-table报错"egenerator-runtime"解决方案：https://article.itxueyuan.com/Owm5jb）
