---
title: uni-app 小程序自定义图标组件
categories: uni-app
date: 2020-11-20
---   

# 图标组件

支持的功能：
* 图标颜色
* 图标名称
* 图标边距
* 图标大小

#### 事件类型
事件名称 |  事件描述
---|---|
click | 点击事件

```
<!--图标组件
<com-icons name="icon-location" size="28rpx" padding="0 10rpx" color="#ffffff"></com-icons>
-->
<template>
  <text 
    class="uni-icons iconfont"
    :class="name"
    :style="{
      fontSize: size,
      color: color,
      padding: padding
    }"
    @click="_onClick"></text>
</template>
<script>
export default {
  name: 'uni-icons',
  props: {
    name: { // 图标类型，拼接前缀icon-
      type: String
    },
    size: { // 图标大小
      type: String,
      default: 'inherit'
    },
    color: { // 图标颜色
      type: String,
      default: 'inherit'
    },
    padding: { // 内边距
      type: String,
      default: '0'
    }
  },
  methods: {
    _onClick() {
      this.$emit('click')
    }
  }
}
</script>
<style lang="scss" scoped>
  @import 'path-to/iconfont.less';
</style>
```