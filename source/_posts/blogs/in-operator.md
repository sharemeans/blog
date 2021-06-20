---
title: in 操作符
categories: 工具
tags: [javascript]
date: 2020-3-20
---

用来判断对象中是否拥有某个属性。

## 使用场景

### for 循环
```js
let obj = {a: 1, b:2, c:3}

for(let key in obj) {
  console.log(obj[key])
}
```

### 判断是否有某个属性
```js
let obj = {a: 1, b:2, c:3}
if('a' in obj) {
  console.log('obj has property "a"')
}
```

## 和 hasOwnProperty的区别
```
in操作符可以判断所有可以通过点操作符访问到的属性，即可以访问对象本身以及原型链上的属性。hasOwnProperty只能判断实例拥有 的属性，不能判断原型链上的属性。
```

由于原型链上的属性是只读的，所以delete操作符只能删除属于对象本身的属性。