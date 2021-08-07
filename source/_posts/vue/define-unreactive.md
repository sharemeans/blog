---
title: vue定义非响应式属性的方法
categories: vue
tags: [vue]
date: 2021-6-15
---

1. created钩子函数中定义
```
created() {
    // 注意data中不要声明该变量名
    this.testData = 'testData'
}
```
不能在mounted钩子中定义，否则，会在首次渲染template的时候报错
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628308848610-image.png)

2. 自定义options
```
<template>
  <div id="app">
    <p v-for="item in $options.list" :key="item.value">{{ item.value }}</p>
  </div>
</template>

<script>
export default {
  name: "app",
  data: () => {
      return {
      }
  },
  list: []
}
</script>
```


3. Object.freeze
```
<template>
  <div id="app">
    <div v-for="(item, index) in list" :key="index">
      {{ item.a }}
    </div>
  </div>
</template>

<script>
export default {
  name: "app",
  data() {
    return {
      list: Object.freeze([
        { a: 1 },
        { a: 1 },
        { a: 1 },
        { a: 1 }
      ])
    }
  },
  mounted() {
    this.list = [
      { a: 2 },
      { a: 2 },
      { a: 2 },
      { a: 2 }
    ].map(item => {
          return Object.freeze(item)
    })
    this.list[0].a = 111 // 此行代码不会生效
    console.log(this)
    console.log(this.list)
  }
}
</script>
```


在mounted钩子中，
组件实例上list具有getter和setter属性，说明list属性值是响应式的，就是说，直接修改List的值是可以的。

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628308871725-image.png)

输出的list指向的数组内不具有getter和setter属性，说明list值中的元素是非响应式的。
![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628308895336-image.png)

通过这种特性，可以让list整块修改触发视图响应，但修改/删除/新增数组内的某个元素DOM都是不会响应的。
Object.freeze是浅冻结，
```
console.log(Object.isExtensible(this.list)) // 输出false
console.log(Object.isExtensible(this.list[0])) // 输出true
```

如果要更高提升性能，关键还是要实现深冻结:
```
list: [
    { value: 1 },
    { value: 2 }
].map(item => {
  return Object.freeze(item)
})
```

注意： data属性加上_或者$前缀，该属性依旧是响应式，只是不能直接通过this访问

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-8-7/1628308928462-image.png)
