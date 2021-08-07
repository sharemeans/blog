---
title: uni-app开发注意事项
categories: uni-app
tags: [uni-app]
date: 2021-6-7
---  

### 对Vue原生语法的支持
1. 不支持v-model

### 内置组件

##### picker 日期

1. 无法自定义按钮颜色

### 样式
1. 组件之间的样式互不影响，跨组件的样式定义不会生效，如果想穿透组件层级覆盖样式，需要在页面级别的文件中定义样式
2. h5编译出来的样式默认使用了scoped，如果想要跨组件层级穿透样式，需要在样式定义前加上/deep/
3. 引入本地的font-family时，font-face定义需要将url直接赋值为绝对路径，或者转化为base64字符串，不能直接引用相对路径，否则会出现找不到字体文件的报错
4. class绑定不支持模板字符串拼接
```
:class="`rank-${index}`"
```
这种语法不支持，但是支持表达式拼接：
```
:class="'rank'+index"
```

### 自定义组件

1. 自定义组件上绑定原生事件需要用.native修饰符
```html
<we-button @click.native="cancel">取消</we-button>
```

2. 自定义组件绑定class和style样式不会透传到组件内部根元素上

3. 不能直接在自定义组件上使用v-slot作为插槽，需要包裹一层template

4. 自定义组件注册为全局组件时，本地开发需要重启。否则引用时属性传进去全是undefined

5. 自定义组件模式下，子组件修改父组件传入的对象的属性不会更新到视图。

### uni-ui版本问题

1. uni-popup组件，1.0.8版本时uni-popup容器没有设置z-index。1.1.9版本却设置了z-index为99，会出现样式兼容问题。因此最好固定版本。

### 插槽

uni-app目前作用域插槽仅支持解构插槽，且作用域插槽不能使用父组件的属性

### 扩展运算符
dcloud问题版本：<= 2.0.0-28720200819001
组件绑定属性时可以使用扩展运算符，但是，为自定义事件回调传参时，不能使用扩展运算符。


```html
<template>
	<view class="content">
			<text class="title" @click="handleClick({...myParams})">点我</text>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				myParams: {
					name: 'Mac'
				}
			}
		},
		onLoad() {

		},
		methods: {
			handleClick(params) {
				console.log('handleClick',params)
			}
		}
	}
</script>


```

