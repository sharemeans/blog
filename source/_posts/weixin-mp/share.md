---
title: 微信小程序分享能力整理
categories: 小程序
tags: [mp-weixin]
date: 2021-7-19
---  


## 小程序分享到朋友圈的方式整理

分享方式 | 分享文案格式 |朋友圈点击交互方式 |支持度 | 额度限制
----|----|----|----|----
APP内分享到微信朋友圈| 和公众号H5一致| 1. 点击朋友圈图文链接先打开一个H5页面<br>2. 用户手动点击页面内的按钮打开小程序 | 无限制，都支持| 1. 短期链接+长期链接每日上限50万<br>2. 长期链接总数不超过10万<br>3. 时长超过30天或者永久类型都称为长期链接
小程序直接分享到微信朋友圈（从小程序内自动打开朋友圈）| 和公众号H5一致 | 1. 点击朋友圈图文链接先打开对应的详情页面（H5页面，代码需要做适配）<br>2. 点击页面底部的“前往小程序”按钮打开小程序  | 目前仅安卓端支持 | 无额度限制
小程序生成海报分享到微信朋友圈（和app内生成海报交互一致，用户手动保存海报，手动打开朋友圈分享）  | 海报  | 长按海报图片识别  | 无限制，都支持  | 无额度限制

## 参考资料：
- https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/share-timeline.html#%E8%AE%BE%E7%BD%AE%E5%88%86%E4%BA%AB%E7%8A%B6%E6%80%81
- https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/url-scheme.html
- https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/short-link/shortlink.generate.html#HTTPS-%E8%B0%83%E7%94%A8
