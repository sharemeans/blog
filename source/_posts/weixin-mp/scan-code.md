---
title: 小程序图片长按识别码兼容性整理
categories: 小程序
tags: [微信, 小程序]
date: 2021-6-30
---


码类型 | 公众号文章 | 小程序image标签 + show long press menu  | 小程序wx.previewImage  | 小程序webview内  | 小程序客服消息
---|---
视频号二维码 | Yes 8.0.2- NO（1）  8.0.2- NO Yes
个人赞赏码  | Yes  | 8.0.2- NO |   |  8.0.2- NO |  Yes
微信名片/群二维码 |  Yes  | 8.0.2- NO  | ios 8.0.6+android 8.0.3+（3） |  ios 8.0.6+android 8.0.3+（3）  | Yes
个人收款二维码 |  Yes |  8.0.2- NO  |   | 8.0.2- NO |  Yes
公众号（订阅号） | 二维码 |  Yes |  8.0.2- NO |   公众号文章：Yes | 其它：NO |  Yes
小程序码  | Yes  | Yes |  Yes |  Yes |  Yes
小程序二维码 |  Yes |  8.0.2- NO  | 8.0.2- NO  | 8.0.6- NO  | Yes
小商店码  | Yes  | Yes  |  Yes  | Yes
企业微信二维码  | Yes |  android  | 8.0.3- NO（亲测） | ios 8.0.7 |  Yes（亲测） |   ios 8.0.6+（亲测） | android 8.0.3+（亲测）  | Yes
普通网址二维码 |  Yes |  8.0.2- NO |   |  8.0.2- NO |  Yes

小程序内图片长按功能（非previewImage）：
版本 |  小程序内image组件show-menu-by-longpress属性  | 小程序webview内img标签长按出现菜单
7.0.x+  | 安卓/ios |  支持小程序码识别 
8.0.3 |  安卓：支持识别微信个人码、企业微信个人码、普通群码与互通群码（企业微信活码不支持） | （3） 
8.0.6  | 苹果：支持识别微信个人码、企业微信个人码、普通群码与互通群码（企业微信活码不支持），但点击弹窗菜单没有反应（bug） 
8.0.7  | 苹果：点击长按弹窗菜单没有反应的bug修复 


参考链接：
（1） https://developers.weixin.qq.com/community/develop/doc/0008ea7edb8f4845c39be413456c00?highLine=%25E8%25B5%259E%25E8%25B5%258F%25E7%25A0%2581%25E8%25AF%2586%25E5%2588%25AB
（2）https://developers.weixin.qq.com/community/develop/article/doc/00008e4f3bc538998bfb344ec56413
（3）https://developers.weixin.qq.com/community/develop/article/doc/000ae09dcfc8887e4b4c287e75b813
（4）https://mp.weixin.qq.com/s/QyJ4XKgaYH-517PEElhwrg
