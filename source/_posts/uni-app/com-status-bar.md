---
title: uni-app 小程序状态栏组件
categories: uni-app
date: 2020-11-20
---   
小程序状态栏高度通过wx.getSystemInfoSync().statusBarHeight获取是最准确的。单位是px。wx.getMenuButtonBoundingClientRect()方法获取的top属性时不准确的，不能作为参考。

```
<template>
	<view :style="{ height: statusBarHeight }" class="uni-status-bar">
		<slot />
	</view>
</template>

<script>
const statusBarHeight = uni.getStatusBarHeight()

export default {
	name: 'status-bar',
	data() {
		return {
			statusBarHeight: statusBarHeight + 'px'
		}
	}
}
</script>

<style lang="scss" scoped>
	.uni-status-bar {
		width: 750rpx;
		height: 20px;
	}
</style>
```

