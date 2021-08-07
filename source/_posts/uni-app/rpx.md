---
title: "uni-app rpx单位在h5环境转换为px的方法"
categories: uni-app
tags: [rpx, uni-app]
date: 2021-6-29
--- 

源码中的rpx单位，如`22rpx`编译后为`%?22?%`的格式。具体px值为多少，是根据运行时设备环境判断的。

运行时单位转换核心方法如下：

```
// newDeviceWidth通常不传，用默认值deviceWidth
function upx2px (number, newDeviceWidth) {
  if (deviceWidth === 0) {
    checkDeviceWidth()
  }

  number = Number(number)
  if (number === 0) {
    return 0
  }
  
  
  // 计算基准以globalStyle配置为准
  const config = __uniConfig.globalStyle || __uniConfig.window || {}
  const maxWidth = checkValue(config.rpxCalcMaxDeviceWidth, 960)
  const baseWidth = checkValue(config.rpxCalcBaseDeviceWidth, 375)
  
  // 容器像素宽度值（通常和设计稿宽度一致）
  const includeWidth = checkValue(config.rpxCalcIncludeWidth, 750)
  
  // deviceWidth为窗口像素宽度（css层面的像素，如iphone 6: 375px）
  let width = newDeviceWidth || deviceWidth
  
  width = number === includeWidth || width <= maxWidth ? width : baseWidth
  // BASE_DEVICE_WIDTH固定为750，因此代码中的值也要以750的设计稿为准
  let result = (number / BASE_DEVICE_WIDTH) * width
  if (result < 0) {
    result = -result
  }
  result = Math.floor(result + EPS)
  if (result === 0) {
    // 计算结果小于1px时特殊处理。
    
    if (deviceDPR === 1 || !isIOS) { // DPR不大于1，或者不是ios设备，则一律为1px
      result = 1
    } else { // DPR大于1，或者ios设备，则一律为0.5px
      result = 0.5
    }
  }
  return number < 0 ? -result : result
}

function checkDeviceWidth () {
  const {
    platform,
    pixelRatio,
    windowWidth
  } = uni.getSystemInfoSync()

  deviceWidth = windowWidth
  deviceDPR = pixelRatio
  isIOS = platform === 'ios'
}
```

该方法在将style插入DOM之前执行。
