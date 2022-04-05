---
title: 一招教你如何快读找到网站某事件响应的代码位置
categories: 工具
tags: [devtool]
date: 2022-4-1
---

作为一枚有追求的前端开发开发，我们无时无刻不抱着“学习”的心态浏览别人的网页。

举个简单的例子，你想要实现一个RGB转Hex的工具，虽然对你来说实现它很简单，但是你实在是太懒了，看到了一个[在线工具](https://www.sioe.cn/yingyong/yanse-rgb-16/)，萌生了歹念的你可以这么做：

###### 第一步：打开performance面板
打开devtools，进入performance面板，并开启“screenshots”

###### 第二步：做好录制准备
把输入框内容填充好


![](https://s2.loli.net/2022/04/05/CGpiNaqcebxZI2D.png)

###### 第三步：录制并快速操作页面

点击performance面板的录制按钮，并立即点击网页的“转换”按钮。转换完成立即停止录制。

###### 第四步：找到转换那一刻的截图

开启“screenshots”的目的就是在于快速找到关键的时间段。放大该时间段的main面板，可以看到js的调用栈：

![](https://s2.loli.net/2022/04/05/TclohiKQ1pzrF2A.png)

选中click事件，展开最底部Call Tree一层层的扒开代码即可。