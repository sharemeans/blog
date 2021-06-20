---
title: 百度地图自定义标记
categories: 其它
tags: [百度地图sdk]
date: 2018-11-5
---

最近有个业务要用到百度地图以及自定义自定义图标功能。具体浏览地址：[锦江酒店-分销通](http://travel.bestwehotel.com/)需要实现的效果如下：

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-6/1622977029489-image.png)

需要满足的交互：
* 左侧列表滚动时，右侧地图中心位置的标记变为左侧鼠标所在的商品上
* 鼠标放置在地图标记上时，显示这个标记对应的酒店名称，点击这个标记跳转到酒店详情页

这个需求的核心任务是：
* 地图SDK选择
* 实现地图的自定义标记
* 自定义标记的状态变化
* 自定义标记的点击事件监听

## 地图SDK选择

可以选择的地图有：
* 腾讯
* 高德
* 百度
* 谷歌

我们的酒店有海外数据，由于目前（2018-11）腾讯和高德地图均未很完美的支持海外位置服务。剩下只有百度和谷歌。谷歌地图经使用发现有一些外部资源依赖被防火墙阻挡。因此最终选择了百度地图。

## 异步加载地图
```js
class BaiduMap {
  constructor(eleId, clickCallback, coordinate) {
    this.mapConfig = {}
    this.mapContainer = eleId
    this.map = null
    this.markers = null
    this.hotelList = ''
    this.hotel = ''
    this.clickCallback = clickCallback
    this.coordinate = coordinate
    if (!window.BMap) {
      BMapSource = this.loadMap()
      BMapSource.then(this.initMap)
    }
  }

  // 1.加载地图
  loadMap() {
    const AK = 'ySDvqVVO3wnmQS49H355c5dhl6ewk469'
    const BMap_URL =
      'https://api.map.baidu.com/api?v=2.0&amp;ak=' +
      AK +
      '&s=1&callback=BMapCallback'
    return new Promise((resolve, reject) => {
      // 插入script脚本
      let scriptNode = document.createElement('script')
      scriptNode.setAttribute('type', 'text/javascript')
      scriptNode.setAttribute('src', BMap_URL)
      document.body.appendChild(scriptNode)

      // 百度地图异步加载回调处理
      window.BMapCallback = function () {
        resolve(window.BMap)
      }
    })
  }

  // 2. 初始化地图
  initMap(BMap) {
    // ...
  }
}
```

## 实现地图的自定义标记
使用Marker类实现自定义标记。
* label.addEventListener监听mouseout,mouseover事件，改变mark样式，实现标记状态变化。
* Icon类添加标记图标
* Label类添加文本标记
* addOverlay方法将marker添加到地图上
* panTo方法将某个坐标移动到地图中心位置
* label.addEventListener监听click事件，实现点击跳转交互

```js
// 3. 批量添加标记
setBatchMarker() {
  // 为每个酒店添加一个标记
  this.markers = this.hotelList.map((hotel, index) => {
    let new_point = new BMap.Point(hotel.longitude, hotel.latitude)
    let marker = new BMap.Marker(new_point)
    // 设置标注图标
    let icon = new BMap.Icon(defaultIcon, new BMap.Size(30, 30))
    marker.setIcon(icon)

    // 创建marker默认的标记
    let content = `<div><span class="markerIndex">${index +
      1}</span><span class="markerLabel">${hotel.translatedName ||
      hotel.hotelName}</span></div>`
    let label = new BMap.Label(content, { position: new_point })
    label.setStyle({
      padding: 0,
      width: '30px',
      height: '30px',
      lineHeight: '30px',
      backgroundColor: 'transparent',
      border: 'none',
      color: '#fff',
      textAlign: 'center',
      overflow: 'hidden'
    })
    marker.setLabel(label)
    this.map.addOverlay(marker) // 将标注添加到地图中

    // 设置marker的鼠标事件（鼠标进入和离开的样式差异）
    // 鼠标经过时
    label.addEventListener('mouseover', function () {
      // 修改样式
    })
    // 鼠标离开时
    label.addEventListener('mouseout', function (e) {
      // 修改样式
    })
    // 点击
    label.addEventListener('click', function (e) {
      // 跳转
    })
    return marker
  })
  // 以列表的第一个为地图的中心点
  let new_point = new BMap.Point(
    this.hotelList[0].longitude,
    this.hotelList[0].latitude
  )
  // 将地图的中心点更改为给定的点，跳转到指定中心点进行渲染。如果该点在当前的地图视图中已经可见，则会以平滑动画的方式移动到中心点位置。
  this.map.panTo(new_point)
}
```
