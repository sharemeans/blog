---
title: 小程序web-view内嵌h5的能力整理
categories: 小程序
tags: [mp-weixin]
date: 2021-6-7
---  

## 个性化布局能力
web-view自动铺满整屏，无法自定义web-view窗口大小。且小程序页面没有任何组件能够高于webview的层级。

## 通信能力
小程序向webview通信：url传参

webview通信向小程序通信：jssdk.postmessage。非实时触发，触发条件：小程序后退、组件销毁（移除webview组件）、分享

关于小程序后退、组件销毁、分享的时机，需要实际测试方知晓具体的限制。

实时通信：暂时未提供

## 支付能力
不支持H5内支付功能


## 参考资料
- [小程序webview内嵌H5支付页面，H5能正常支付吗](https://developers.weixin.qq.com/community/develop/doc/000ca683120980b88c98c12395b000)

- [webview内的jssdk支持的api列表说明](https://developers.weixin.qq.com/miniprogram/dev/component/web-view.html)
