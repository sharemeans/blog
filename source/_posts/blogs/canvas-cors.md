---
title: canvas跨域限制
categories: canvas
tags: [canvas]
date: 2022-3-15
---

最近使用canvas图片时遇到了跨域问题。当绘制的图片资源和当前地址跨域时，使用toBlob导出图片时会有跨域错误且导出失败。仔细看了一下[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML/CORS_enabled_image)，在此做下总结：


canvas可以绘制的图片来源有：

- HTMLImageElement
- SVGImageElement 
- HTMLVideoElement
- HTMLCanvasElement
- ImageBitmap 
- OffscreenCanvas

被绘制进入canvas的图片，如果未满足同源策略（响应方未返回cors头部，跨域资源如果来源于HTMLImageElement或者SVGImageElement，且标签并未携带"crossOrigin"属性），则会导致画布被污染。

被污染的画布会被禁止以下行为：

- 调用getImageData()
- 调用toBlob()
- 调用toDataURL()

为什么canvas需要受到cors策略的约束呢？经过查阅各方资料，判定原因为：

> 图片的信息包含了地理时间等不可见的用户隐私，跨域访问并未经授权的情况下，直接读取图片信息可能会导致用户隐私泄露。

