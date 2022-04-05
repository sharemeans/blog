---
title: 微信小程序中scroll-view问题总结
categories: 小程序
tags: [小程序, css]
date: 2021-6-17
---


## ios设备中fix定位元素

### 在scroll-view中会被遮挡
scroll-view内部如果有fix定位元素，由于ios设备有弹动功能，一旦scroll-view被拉到不包含fix定位的元素之外，fix定位元素会被遮挡

### scroll-view中无法置于顶层
scroll-view内部的fix定位元素，无论z-index设置为多少，都无法置于scroll-view外部fix定位元素的上层。

### 解决办法
这个问题在h5和小程序中都存在。原因是：
scroll-view元素在ios下的样式包含以下属性：
```
 -webkit-overflow-scrolling: touch
```

该属性的作用是让touch滚定行为更加流畅：
https://developer.mozilla.org/en-US/docs/Web/CSS/-webkit-overflow-scrolling

但是，该属性值为touch时会影响容器内的fixed元素层级，改为auto就不会影响了，但是touch属性值目前是有必要存在的。
https://developers.weixin.qq.com/community/develop/doc/0000667484c96844b83ac9c7651809?_at=1617789574414
https://developers.weixin.qq.com/community/develop/doc/0004aeafeccb789ac219e474756000
解决方式是将fixed定位元素移到scroll-view外面。

### safari 13.0以上的版本解决办法

safari 13.0以上的版本就不需要该属性了
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-8-7/1628308610008-image.png)
(https://developer.apple.com/documentation/safari-release-notes/safari-13-release-notes)

了解了原因之后，我们分析下ios系统版本占比：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-8-7/1628308669260-image.png)

(数据来源：腾讯大数据->腾讯移动分析 MTA)

可见97%的用户版本号>13，可以放心的移除了。拿个ios 13真机试下，果然没问题。


## 内部垂直方向margin在安卓设备出现双滚动条
```
<template>
<scroll-view
    class="product-wrap"
    scroll-y
    enable-back-to-top
        >
        <view class="product-item"></view>
 </scroll-view>
 </template>
 
 <style>
 .product-wrap {
     height: 100%;
 }
 .product-item {
     margin-top: 20rpx;
 }
 </style>
```


以上代码会导致**安卓设备**页面滚动出现双滚动条。需要为内部元素加一层包裹，称为BFC，防止margin塌陷:
```
<template>
<scroll-view
    class="product-wrap"
    scroll-y
    enable-back-to-top
        >
        <view class="product-container"><view class="product-item"></view></view>
 </scroll-view>
 </template>
 
 <style>
 .product-wrap {
     height: 100%;
 }
 .product-container {
     overflow: hidden;
 }
 .product-item {
     margin-top: 20rpx;
 }
 </style>
```