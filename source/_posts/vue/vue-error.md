---
title: 常见的vue 报错信息
categories: vue
date: 2020-7-9
---


### 模板渲染初始化问题

```ruby
Proxy.render Cannot read property '0' of undefined
```
上面这个报错是指在构建DOM的时候，有个变量是undefined，但是这个变量被当成Array类型并访问了下标值。

这种情况通常出现在数据初始化的时候，某些属性不存在，但是访问了这个不存在的属性的子属性

### 计算属性绑定v-model
```ruby
Computed property "" was assigned to but it has no setter.
```

通常这个报错的原因是，用一个计算属性绑定到v-model上面。

```ruby
//  template部分
<el-rate
 v-model="rate"
 disabled></el-rate>
// js部分
computed: {
    rate() {
      if (this.detail) {
        return parseFloat(this.detail.starRating) || 0
      } else {
        return 0
      }
    }
  }
```

解决办法

1. 尽量避免计算属性用在v-model上，因为计算属性一般是通过现有的值计算出来的。如果可以通过其它方式修改计算属性的话，数据状态会比较混乱。
2. 计算属性默认是只有getter的，就是说只能取值，不能设值。computed支持手动添加set函数。具体参考[vue 计算属性](https://cn.vuejs.org/v2/api/#computed)
