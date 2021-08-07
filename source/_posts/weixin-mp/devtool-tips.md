---
title: 微信开发者工具的小tips
categories: 小程序
tags: [微信开发者工具, mp-weixin]
date: 2021-6-12
--- 

## 默认选项开关
project.config.json可以配置开发者工具的一些选项开关，不用每次不厌其烦的等模拟器渲染完再去一个个的关闭了。

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628309327902-image.png)
[project.config.json](https://developers.weixin.qq.com/miniprogram/dev/devtools/projectconfig.html)

其中，packOptions可以配置代码上传时的忽略目录。

## open-data组件
用户个人信息展示可以使用<open-data>组件，无需再去调用各种api了。

指定对应的type即可显示对应的信息。
支持的类型有：
```
groupName
userNickName
userAvatarUrl
userGender
userCity
userProvince
userCountry
userLanguage
```

还可以在open-data上绑定class,style设置样式。
