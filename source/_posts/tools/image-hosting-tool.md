---
title: 推荐一个好用的图床工具
categories: 工具
tags: [图床]
date: 2022-4-5
---

最近突然发现gitee图床不能用了，以前博客里面的图片链接都失效了，原来是gitee添加了防盗链，无法直接跨域访问。

还能咋滴，薅羊毛的我有错在先无话可说。咱也不差那几块钱，阿里云OSS真真香，结合专业的图床工具使用更香，我推荐使用PicGo。

作为一款专业的图床工具，PicGo具有以下优点：
1.内置多款热门图床工具
- SM.MS
- 腾讯云
- 阿里云
- 七牛
- Imgur

2.支持自定义图床插件，目前已经有n款[开源插件](https://github.com/PicGo/Awesome-PicGo)。

3.有vscode插件版的[PicGo插件](https://github.com/PicGo/Awesome-PicGo#hammer_and_wrench-plugin-for-other-apps)。

4.使用方便，支持剪贴板上传和拖拽上传。上传结果可一键复制，而且支持多种链接格式（markdow，html，URL等）：

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture20220405205537.png)

开通阿里云OSS之后，立即把gitee图床的整个仓库丢上去了，然后对博客文件中的域名全局替换，非常方便。
