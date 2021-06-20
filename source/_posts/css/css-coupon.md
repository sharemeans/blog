---
title: 纯css实现优惠券样式
categories: css
tags: [css]
date: 2020-5-3
--- 

核心知识点：
* radial-gradient实现优惠券卡片左右2边的缺角
* background可以叠加多个效果

```html
<div class="ticket">
  · 海外留学项目最高立减<span>5000</span>元现金 <br>
  · 语言培训项目最高立减<span>2000</span>元现金 <br>
  · 国际部项目最高三年<span>全额奖学金</span>现场发放
</div>
```

```css
.ticket{
  width: 3.45rem;//内容层的宽度
  height: 1.14rem;//内容层的高度
  margin: 0 auto;
  box-sizing: border-box;
  padding: 0.25rem 0.3rem;font-size: 0.15rem; 
  line-height: 1.5;
  color: #fff;
  background: radial-gradient(circle at 0 0.57rem, transparent 0.15rem, #ffbcbe 0.06rem) top left, linear-gradient(0.25turn, #ffbcbe, #000), radial-gradient(circle at 0.17rem 0.57rem, transparent 0.15rem, #000 0.16rem) bottom right;  background-size: 0.17rem 1.14rem, 2.95rem 1.14rem, 0.17rem 1.14rem;
  background-size: 0.17rem 1.14rem, 2.95rem 1.14rem, 0.17rem 1.14rem;
  background-repeat: no-repeat;
  background-position: 0.1rem 0px,0.26rem 0px,3.2rem 0px;
}
.ticket span{
  color: #eb6877;
  text-decoration: underline;
}
```