---
title: 手写defineReactive
categories: vue
tags: [vue]
date: 2019-11-3
---

defineReactive是Vue响应式的核心。我们手写一个简单的defineReactive：

```
let obj = {
  c: 2
}
/**
* 将获取到的变量值渲染到视图
**/
function showGet(key, val) {
  console.log(key, 'get newVal', val)
  let bodyEle = window.document.body
  bodyEle && (bodyEle.innerText = bodyEle.innerText + '\n' + key + ' get newVal' + val)
}

/**
* 将更新后的变量值渲染到视图
**/
function showSet(key, newVal) {
  console.log(key, 'set newVal', newVal)
  let bodyEle = window.document.body
  bodyEle && (bodyEle.innerText = bodyEle.innerText + '\n' + key + ' set newVal' + newVal)
}

function defineReactive(obj, key) {
  let val = obj[key]
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function() {
      showGet(key, val)
      return val
    },
    set: function (newVal) {
      val = newVal
      showSet(key, newVal)
    }
  })
}

let keys = Object.keys(obj)
for (const key of keys) {
  defineReactive(obj, key)
}

obj.c = 3
console.log(obj.c)


```