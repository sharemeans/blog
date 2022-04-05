---
title: uni-app 打包为h5时rpx编译错误
categories: uni-app
tags: [rpx, uni-app]
date: 2021-6-29
---  

某天，产品给我看了一个线上h5页面的布局问题：有个组件的padding属性不见了。我以为这是一贯的粗心导致简单的样式问题，当我打开生产页面样式审查的时候，我傻眼了：
```
.comp-card .content--wrap[data-v-8ec74a42] { 
  padding: %?22?% %?24?%; 
}
```

### 暂时的解决方案
这个问题之前同事也遇到过，当时经过简单的实验，发现是uni-app较新版本才会出现。我们目前使用的uni-app版本都是`latest`，即使用较新的稳定版本。

既然没有定位到直接原因，干脆先固定到较低的版本号暂时解决问题。

### 分析问题特点
此时，`编译结果错误`，`uni-app版本号错误`这俩关键词已经在我脑海里扎根。

为了进一步研究问题特点和原因，我尝试在开发环境复现这个问题，结果发现只有在NODE_ENV的值为production时才能出现。

继续审查了其它元素，只发现了这一处样式异常。单独的异常很难分析出问题的原因在哪里。

分析到这里没有进展，直接去uni-app的Github和社区看看有没有相关issue，还真找到了几个：
1. https://github.com/dcloudio/uni-app/issues/1132
2. https://github.com/dcloudio/uni-app/issues/1069

这几个issue的最终解决办法就是使用@vue/cli 3.x的版本（我用的就是3.x啊喂！更何况uni-app现在已经支持@vue/cli 4.x了）

没有找到复现条件和demo，直接拿着这2个现象去提[issue](https://github.com/dcloudio/uni-app/issues/2600 "issue")。果然等来的是类似“按照你说的条件，没发现这个问题啊”这样的回复。

### 刨根究底
自己动手，丰衣足食。我们知道，webpack对css的处理方式通常是使用style-loader将css插入header标签中，uni-app也是如此：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-4-25/1619361955023-image.png)
上图中6215就是一个css模块：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-4-25/1619362025628-image.png)
这个css模块其实就是一个字符串，截取这段字符串格式化后的一部分：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-4-25/1619361866485-image.png)
可以得出结论：
> uni-app编译后的css中，rpx单位并没有直接转化成px，因为需要根据具体的设备类型做移动端适配。这个适配工作就在style标签插入html之前。

接下来，找到rpx转px的工具函数，在@dcloudio/vue-cli-plugin-uni/packages/h5-vue-style-loader/lib/addStylesClient.js文件中，名为`processCss`的方法：

```
var UPX_RE = /%\?([+-]?\d+(\.\d+)?)\?%/g
var BODY_RE = /\.\?%PAGE\?%/g
var BODY_SCOPED_RE = /\?%PAGE\?%\[data-v-[a-z0-9]{8}\]/g
var PAGE_SCOPED_RE = /uni-page-body\[data-v-[a-z0-9]{8}\]/g
var VAR_STATUS_BAR_HEIGHT = /var\(--status-bar-height\)/gi
var VAR_WINDOW_TOP = /var\(--window-top\)/gi
var VAR_WINDOW_BOTTOM = /var\(--window-bottom\)/gi
var VAR_WINDOW_LEFT = /var\(--window-left\)/gi
var VAR_WINDOW_RIGHT = /var\(--window-right\)/gi

function processCss(css) {
	var page = getPage()
	if (typeof uni !== 'undefined' && !uni.canIUse('css.var')) { //不支持 css 变量
		var offset = getWindowOffset()
		css = css.replace(VAR_STATUS_BAR_HEIGHT, '0px')
			.replace(VAR_WINDOW_TOP, offset.top + 'px')
			.replace(VAR_WINDOW_BOTTOM, offset.bottom + 'px')
            .replace(VAR_WINDOW_LEFT, '0px')
            .replace(VAR_WINDOW_RIGHT, '0px')
	}
	return css
		.replace(BODY_SCOPED_RE, page)
		.replace(BODY_RE, '')
		.replace(PAGE_SCOPED_RE, 'body.' + page + ' uni-page-body')
		.replace(/\{[\s\S]+?\}|@media.+\{/g, function (css) {
      if(typeof uni === 'undefined'){
        return css
      }
			return css.replace(UPX_RE, function (a, b) {
				return uni.upx2px(b) + 'px'
			})
		})
}
```
正则变量`UPX_RE`是生成px的关键点，replace链式调用的结尾需要对满足正则`/\{[\s\S]+?\}|@media.+\{/g`的部分做px单位转换。现在可以确认的是，这个正则没有完全覆盖我们的css模块中所有的样式。
### 解决啦
拿以下样式做一个验证：

```
.class-a[data-v-8ec74a42] {
  width: %?678?%;
}
@media only screen and (-webkit-min-device-pixel-ratio: 2) {
  .class-a[data-v-8ec74a42] {
    -webkit-transform: scaleY(0.5);
    -ms-transform: scaleY(0.5);
    transform: scaleY(0.5);
  }
}
.class-b[data-v-8ec74a42] {
  font-size: %?28?%;
  line-height: %?88?%;
}
```
发现`@media.+\{`部分会匹配从`@media`到`.class-b[data-v-8ec74a42] {`，剩下的部分因为无法匹配`\{[\s\S]+?\}`就被忽略了。

根据该文件的修改记录，看到了上一次修改的commit message：
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-4-25/1619363407452-image.png)
哦，人家大概是想处理这样的情况下吧：
```
@media screen and (max-width: 300rpx)
```
修正后的正则：
```
/\{[\s\S]+?\}|@media[^{]+/g
```
既然如此，那就赶紧提个[pull request](https://github.com/dcloudio/uni-app/pull/2614 "pull request")叭！


### 相关issue:
https://github.com/dcloudio/uni-app/issues/1132
https://github.com/dcloudio/uni-app/issues/1606
https://github.com/dcloudio/uni-app/issues/1069
https://github.com/dcloudio/uni-app/issues/2600



