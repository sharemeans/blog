---
title: uni-app实现文本长按复制
categories: uni-app
tags: [长按复制, uni-app]
date: 2021-7-20
---  

### 文本长按复制的实现原理

小程序所有的标签css都具有user-select:none属性。可通过user-select:text使其支持长按复制。

真机调试的时候样式审查，看到的text标签具有user-select:text：

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-6-17/1623928631379-image.png)

虽然审查出来样式是有user-select:text，实际上在ios设备上依旧无法长按选择。这个样式审查估计是假的吧。

开启了user-select:text的text标签，由inline布局变为inline-block，需要开发者自己做样式适配（[官方说明](https://developers.weixin.qq.com/community/develop/doc/00086ee03a0bd096595ac5e905ac00)）

##### 安卓
* css中的user-select:text有效
* text标签的user-select有效

##### IOS
* css中的user-select:text无效
* text标签的user-select有效

因此还是要用text标签的user-text属性。

## 富文本的长按复制

富文本要用到rich-text标签。但是该标签不支持user-select属性。

唯一的办法就是将富文本中的文本标签都改成text标签。

插件市场有很多富文本解析插件。

## mp-html

[文档](https://ext.dcloud.net.cn/plugin?id=805#detail)

优点：
* 支持图片预览
* 文案长按复制
* 全端支持

缺陷：
* selectable:true 对ios无效，selectable:force才对ios有效。
* selectable:force对h5无效，所以h5需要额外用user-select:text样式对容器处理。
* lazy-load在小程序端有问题。图片并没有渲染出来。

## H5模式下的长按复制
以上都是小程序环境的处理方法。

H5 模式下，text标签的user-select属性无效，需要在css中设置user-select:text属性