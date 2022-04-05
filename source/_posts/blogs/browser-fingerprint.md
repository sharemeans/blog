---
title: 了解浏览器指纹
categories: 浏览器
tags: [指纹]
date: 2022-3-30
---

浏览器指纹识别是指，网站服务提供方，通过浏览器暴露的api，配合各种奇技淫巧，从而识别出是否是同一个用户。

#### 应用场景
- 防恶意爬虫
- 电商网站防恶意刷单，防RPA
- 广告推荐

#### 发展历史
指纹识别的技术直接决定了识别的准确度：

- 1.第一代：同一个设备，同一个浏览器，同一个网址的用户。主要通过cookie识别
- 2.第二代：同一个设备的用户，通过userAgent，canvas，WebRTC，FontList等方法确认设备硬件的唯一性。
- 3.第三代：同一个用户在不同设备的识别。通过分析用户的行为从用户画像上确定唯一性。

#### 信息墒和稳定性

不同浏览器指纹识别方法有不同的权重，其权重根据稳定性和信息墒来决定，且有些方法在单浏览器和跨浏览器识别上具有不同的信息墒。

稳定性：跨浏览器的值是否一致
信息墒：信息熵可以表示信息的价值，数值越大表示信息越可靠，辨识度越高


参考资料：
- [浏览器指纹追踪技术简述](https://zhuanlan.zhihu.com/p/94158920)
- [WebRTC 泄漏真实 IP 地址](https://www.vmlogin.cc/blog/137.html)
- [浏览器指纹检测](https://browserleaks.com/)
- [2.5代指纹追踪技术—跨浏览器指纹识别](https://mp.weixin.qq.com/s/gR7CcICIPV8S1Jop3i_8WQ)
- [浏览器指纹：原来我们一直被互联网巨头监视](https://cloud.tencent.com/developer/article/1602369)
- [浏览器指纹](https://docs.multilogin.com/l/zh/category/o40iDKTlyQ-)

指纹识别网站：
- [https://amiunique.org/fp](https://amiunique.org/fp)
- [WebRTC 泄漏真实 IP 地址](https://www.vmlogin.cc/blog/137.html)
- [fingerprintjs（开源）](https://fingerprintjs.com/)