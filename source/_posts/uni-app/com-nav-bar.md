---
title: uni-app 小程序自定义导航栏组件
categories: uni-app
date: 2020-11-20
--- 

该组件是基于uni-ui扩展组件uni-nav-bar修改。导航组件严格来说是有2个组件组成：状态栏组件，头部组件

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-31/1622442807912-image.png)

getStatusBarHeight和getMenuButtonBoundingClientRect方法可以获取小程序状态栏和头部胶囊信息，取值关系如下：

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-5-31/1622449017125-image.png)

需要注意的是，通过getMenuButtonBoundingClientRect方法top属性一般比getStatusBarHeight的值大，且不同设备具体差异大小不同。

关于状态栏组件，具体搜索文章`小程序状态栏组件`。

## 导航组件

该组件依赖于以下组件：
* com-icons iconfont组件
* com-status-bar 状态栏组件
  
以上组件可通过搜索名称查找相关代码。

导航组件具有以下功能：
* 滚动头部固定/跟随文档
* 自定义返回按钮、按钮后的文案、左侧/右侧区域宽度
* 头部固定时是否需要保持高度（避免业务方做高度兼容）
* 头部透明度（支持头部随着滚动过渡）
* 是否需要状态栏
* 自定义返回按钮点击行为
* 小程序环境自动判断是否需要显示回到首页按钮

#### 事件类型
事件名称 |  事件描述
---|---|
clickLeft | 点击左侧区域
clickRight | 点击右侧区域

#### 方法
方法名称 |  方法描述
---|---|
getHeight | 获取导航高度
  
使用时需要修改以下功能：
* icon组件对应的icon字体需要根据具体的应用修改
* HOME_PATH对应的值根据具体的应用修改

```html
<template>
	<view class="uni-navbar">
		<view :class="{ 'uni-navbar--fixed': fixed, 'uni-navbar--shadow': shadow, 'uni-navbar--border': border }" :style="{ 'background': backgroundColor, opacity: opacity }"
		class="uni-navbar__content">
			<com-status-bar v-if="statusBar" />
			<view :style="{ color: color,background: backgroundColor, height: headerHeight, lineHeight: headerHeight }" class="uni-navbar__header uni-navbar__content_view">
				
				<!-- 左侧（默认点击行为：返回上一页。可重写） -->
				<view @click="onClickLeft" class="uni-navbar__header-btns uni-navbar__header-btns-left uni-navbar__content_view" :style="{width: leftIconWidth}">
					<!-- 回到首页按钮（分享场景）（与其它按钮或者文案互斥） -->
					<!-- #ifdef MP-WEIXIN -->
					<view 
						class="uni-navbar__content_view"
						:class="{'header-icon-with-bg': showIconBg}" v-if="showHome" @click="goHome">
						<com-icons :color="color" name="icon132" size="40rpx" />
					</view>
					<!-- #endif -->

          <!-- 返回按钮 -->
					<view
						class="uni-navbar__content_view"
						:class="{'header-icon-with-bg': showIconBg}" v-if="!showHome && leftIcon && showLeftBack" >
						<com-icons :color="color" name="icon9" size="40rpx" />
					</view>

          <!-- 左侧显示文案 -->
					<view
						class="uni-navbar-btn-text uni-navbar__content_view"
						:class="{ 'uni-navbar-btn-icon-left': leftIcon }"
					 v-if="!showHome && leftText.length">
						<text :style="{ color: color, fontSize: '28rpx' }">{{ leftText }}</text>
					</view>
          <!-- 左侧区域插槽（具名插槽） -->
					<slot name="left" />
				</view>

				<!-- 标题区域 -->
				<view class="uni-navbar__header-container uni-navbar__content_view">
					<view class="uni-navbar__header-container-inner uni-navbar__content_view" v-if="title.length">
						<text class="uni-nav-bar-text" :style="{color: color }">{{ title }}</text>
					</view>
					<!-- 标题插槽（无名插槽） -->
					<slot />
				</view>

				<!-- 右侧按钮区域 -->
				<view
					@tap="onClickRight" class="uni-navbar__header-btns uni-navbar__header-btns-right uni-navbar__content_view"
					:style="{width: rightIconWidth}">
					<view
					class="uni-navbar__content_view"
					:class="{'header-icon-with-bg': showIconBg}"
					v-if="rightIcon">
						<com-icons :color="color" :name="rightIcon" size="56rpx" />
					</view>
					<!-- 优先显示图标 -->
					<view class="uni-navbar-btn-text uni-navbar__content_view" v-if="rightText.length && !rightIcon.length">
						<text class="uni-nav-bar-right-text">{{ rightText }}</text>
					</view>
					<!-- 右侧区域插槽（具名插槽） -->
					<slot name="right" />
				</view>
			</view>

			<!-- 副标题（样式完全自定义） -->
			<view class="uni-navbar__sub-header">
				<slot name="sub-nav"></slot>
			</view>
		</view>

		<!-- 标题区域占位 -->
		<view class="uni-navbar__placeholder" v-if="fixed && holdPlace">
			<com-status-bar v-if="statusBar" />
			<view class="uni-navbar__placeholder-view" :style="{'padding-top': subHeaderHeight, height: headerHeight}"/>
		</view>
	</view>
</template>

<script>
// 首页路径
const HOME_PATH = 'pages/index/index'

var CustomNavbarHeight = uni.getCustomNavbarHeight()
	/**
	 * NavBar 自定义导航栏
	 * @description 导航栏组件，主要用于头部导航
	 * @tutorial https://ext.dcloud.net.cn/plugin?id=52
	 * @property {String} title 标题文字
	 * @property {String} leftText 左侧按钮文本
	 * @property {String} rightText 右侧按钮文本
	 * @property {String} leftIcon 左侧按钮图标（图标类型参考 [Icon 图标](http://ext.dcloud.net.cn/plugin?id=28) type 属性）
	 * @property {String} rightIcon 右侧按钮图标（图标类型参考 [Icon 图标](http://ext.dcloud.net.cn/plugin?id=28) type 属性）
	 * @property {String} leftIconWidth （扩展属性）左侧按钮区域宽度 用来满足设计稿要求
	 * @property {String} rightIconWidth （扩展属性）右侧按钮区域宽度 用来满足设计稿要求
	 * @property {String} color 图标和文字颜色
	 * @property {String} backgroundColor 导航栏背景颜色
	 * @property {Boolean} fixed = [true|false] 是否固定顶部
	 * @property {Boolean} holdPlace = [true|false]  （扩展属性）固定在顶部时，是否需要占位
	 * @property {Boolean} opacity = 0-1  （扩展属性）头部透明度 用于头部滚动渐变
	 * @property {Boolean} statusBar = [true|false] 是否包含状态栏
	 * @property {Boolean} shadow = [true|false] 导航栏下是否有阴影
	 * @property {Boolean} border = [true|false] 导航栏下是否有边框线
	 * @event {Function} showLeftBack （扩展属性）是否显示返回按钮
	 * @event {Function} clickLeftBack （扩展属性）点击左侧区域是否关闭当前窗口
	 */
export default {
	name: "NavBar",
	props: {
		title: {
			type: String,
			default: ""
		},
		leftText: {
			type: String,
			default: ""
		},
		rightText: {
			type: String,
			default: ""
		},
		leftIcon: {
			type: String,
			default: ""
		},
		leftIconWidth: {
			type: String,
			default: "132rpx"
		},
		rightIcon: {
			type: String,
			default: ""
		},
		rightIconWidth: {
			type: String,
			default: '132rpx'
		},
		fixed: {
			type: [Boolean, String],
			default: false
		},
		holdPlace: {
			type: Boolean,
			default: true
		},
		color: {
			type: String,
			default: "#000000"
		},
		opacity: {
			type: [String,Number],
			default: "1"
		},
		backgroundColor: {
			type: String,
			default: "#FFFFFF"
		},
		statusBar: {
			type: [Boolean, String],
			default: false
		},
		shadow: {
			type: [String, Boolean],
			default: false
		},
		border: {
			type: [String, Boolean],
			default: true
		},
		clickLeftBack: { // 点击返回按钮区域是否返回上一页/路由
			type: [String, Boolean],
			default: true
		},
		showLeftBack: {
			type: [Boolean, String],
			default: true
		}
	},
	data() {
		return {
			showIconBg: false, // 是否显示按钮的背景色
			showHome: false, // 是否显示回到首页按钮
			showNavBar: true,
			headerHeight: CustomNavbarHeight + 'px', // 头部高度，默认是40
			subHeaderHeight: '' // 子头部高度
		}
	},
	watch: {
		color() {
			this.handleColorChange()
		}
	},
	created() {
		this.handleColorChange()
	},
	mounted() {
		if(uni.report && this.title !== '') {
			uni.report('title', this.title)
		}
		this.updateHeight()
		// #ifdef MP-WEIXIN
		this.judgeHome()
		// #endif
	},
	updated() {
		this.updateHeight()
	},
	methods: {
		updateHeight() {
			this.getSubHeaderHeight()
			this.getHeight()
		},
		handleColorChange() {
			let color = this.color && this.color.toLocaleLowerCase()
			// 只有背景色透明才设置按钮的背景颜色
			let bgTransparent = this.backgroundColor == 'transparent' || this.backgroundColor == 'inherit'
			// 设置状态栏字体颜色
			if (color && (color === '#ffffff' || color === '#fff')) {
				uni.setNavigationBarColor({
					frontColor: '#ffffff',
					backgroundColor: 'transparent'
				})
				if (bgTransparent) this.showIconBg = true
			} else {
				uni.setNavigationBarColor({
					frontColor: '#000000',
					backgroundColor: 'transparent'
				})
				this.showIconBg = false
			}
		},
    /**
  	 * 获取节点offset值 （而非小程序提供的相对可视窗口的offset值）
  	 * @param {string} selector 节点 与 select()方法一致
  	 * @param {string} parent 节点相对滚动节点，非必传，不传以 viewport为默认值
  	 * @param {string} context 查询上下文（页面/组件实例），含有自定义组件的页面，或者自定义组件中调用，必传
  	 */
  	getOffset(selector, parent, context) {
      let query = context ? context.createSelectorQuery() : uni.createSelectorQuery();

  		let parentQuery = parent ? query.select(parent) : query.selectViewport()

  		query.select(selector).boundingClientRect()
  		parentQuery.scrollOffset()

  		return new Promise((resolve, reject) => {
  			query.exec(([selectRect, parentRect]) => {
  				if (selectRect) {
  					resolve({
  						top: selectRect.top + parentRect.scrollTop,
  						left: selectRect.left + parentRect.scrollLeft,
  						height: selectRect.height,
              width: selectRect.width,
              bottom: selectRect.bottom
  					})
  				} else {
  					console.warn(`[mp::getOffset]: 获取目标元素${selector}的offset信息失败`)
  					resolve({
  						top: 0,
  						left: 0,
  						height: 0,
  						width: 0
  					})
  				}
  			})
  		})
  	},
		async getSubHeaderHeight() {
			let subHeaderOffset = await this.getOffset('.uni-navbar__sub-header', null, this)

			this.subHeaderHeight = subHeaderOffset.height + 'px'
		},
		async getHeight() {
			let navOffset = await this.getOffset('.uni-navbar__content', null, this)

			return navOffset
		},
		// #ifdef MP-WEIXIN
    /**
     * 判断是否需要显示home按钮（只有小程序需要）
     **/
		judgeHome() {
      // 页面栈栈顶，且路径为非首页，则显示home按钮
			let pages = getCurrentPages()
			if ((pages.length <= 1) && HOME_PATH !== pages[0].route) {
				this.showHome = true
			}
		},
		goHome() {
			uni.reLaunch({
				url: '/'+HOME_PATH,
			})
		},
		// #endif
		onClickLeft() {
			if (this.clickLeftBack) {
				uni.navigateBack()
			} else {
				this.$emit("clickLeft");
			}
		},
		onClickRight() {
			this.$emit("clickRight");
		}
	}
};
</script>

<style lang="scss" scoped>
	$nav-height: 44px;
	.uni-nav-bar-text {
		font-size: $uni-font-size-lg;
	}
	.uni-nav-bar-right-text {
		font-size: $uni-font-size-base;
	}

	.uni-navbar {
		position: relative;
		width: 100%;
		font-family: PingFangSC-Medium, PingFang SC;
		box-sizing: border-box;
	}

	.uni-navbar__content {
		position: relative;
		width: 100%;
		background-color: $uni-bg-color;
	}

	.uni-navbar__content_view {
		display: flex;
		align-items: center;
		flex-direction: row;
	}

	.uni-navbar__header {
		display: flex;
		flex-direction: row;
		width: 100%;
		height: $nav-height;
		line-height: $nav-height;
		font-size: 32rpx;
	}

	.header-icon-with-bg {
		display: flex;
		justify-content: center;
		align-items: center;
		align-content: center;
		background: rgba($color: #000000, $alpha: 0.2);
		border-radius: 100%;
		height: 52rpx;
		width: 52rpx;
	}

	.uni-navbar__header-btns {
		display: flex;
		flex-wrap: nowrap;
		padding: 0 12px;
		justify-content: center;
		align-items: center;
		box-sizing: border-box;
	}

	.uni-navbar__header-btns-left {
		display: flex;
		box-sizing: border-box;
		justify-content: flex-start;
	}

	.uni-navbar__header-btns-right {
		display: flex;
		box-sizing: border-box;
		justify-content: flex-end;
		margin-right: 10rpx;
	}


	.uni-navbar__header-container {
		flex: 1;
		font-weight: 600;
	}

	.uni-navbar__header-container-inner {
		display: flex;
		flex: 1;
		align-items: center;
		justify-content: center;
		font-size: $uni-font-size-base;
	}


	.uni-navbar__placeholder-view {
		height: $nav-height;
		box-sizing: content-box;
	}

	.uni-navbar--fixed {
		position: fixed;
		z-index: 998;
	}

	.uni-navbar--shadow {
		box-shadow: 0 1px 6px #ccc;
	}

	.uni-navbar--border {
		border-bottom-width: 1rpx;
		border-bottom-style: solid;
		border-bottom-color: $uni-border-color;
	}
</style>

```

