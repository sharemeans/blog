---
title: 防抖和节流
categories: js
date: 2020-5-8
--- 

## 防抖

短时间内连续触发的事件，不执行回调，给定一个冷却时间，这段时间内没有触发则时间结束后执行回调。即，持续触发不执行，不触发一段时间之后再执行。

常见场景：
* 输入框持续输入，输入内容远程查询
* 多次触发点击事件
* 滚动后获取滚动距离

```
const debounce = function (func, delay) {
    let timer = null

    return function() {
        clearTimeout(timer)
        timer = setTimeout(() => {
            func.apply(this, arguments)
        }, delay || 300)
    }
}
```

## 节流

函数一段时间内只执行一次。即，持续触发并不会执行多次，到一定时间再去执行。通过闭包保存开关状态。
常见场景：
* 自定义滚动条
* 页面resize

```
const throttle = funciton (func, delay) {
  let run = true
  return function () {
    if (!run) { // 如果开关关闭了，那就直接不执行下边的代码
      return
    }
    // 持续触发的话，run一直是false，就会停在上边的判断那里
    run = false 
    func.apply(this, arguments)

    // 定时器到时间之后，会把开关打开，我们的函数就会被执行
    setTimeout(() => { 
      run = true
    }, delay)
  }
}
```

参考资料:
[知乎：函数的防抖和节流是个啥？？？](https://zhuanlan.zhihu.com/p/72923073)