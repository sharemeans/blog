---
title: 凹槽样式小结
categories: css
tags: [css]
date: 2021-6-2
---  
## border-radius

由于border-radius的最大半径是50%，无法直接实现大弧度。独立一个元素，使其尺寸是容器的1倍以上，再通过平易即可实现视觉上的凹槽效果。

```
<!--html-->
<div class="container">
  <div class="curved-bg"></div>
</div>
// css
.container {
  width: 300px;
  height: 300px;
  overflow: hidden;
}
.curved-bg {
  width: 300%;
  height: 300%;
  margin-top: -250%;
  margin-left: -100%;
  border-radius: 50%;
  background-color: rgba(0, 0, 0, 0.3);
}
```

以上方法只是对一个简单的背景做圆角处理。试想以下这样的场景：圆弧部分只是一个修饰，图片部分的轮播图可响应点击交互，如下图banner底部的大圆弧。

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-6-2/1622647130156-image.png)

针对这样的场景，border-radius方法依旧可以用，只不过需要用到`pointer-event:none`实现点击穿透。该属性绝大部分浏览器目前都支持。

## 切图

最简单的办法，就是将有弧度的部分切成一张png图盖在banner下方，由于这一个区域的高度较浅，不影响点击交互。

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-6-2/1622648935597-image.png)

## curved border-radius

可以通过椭圆圆角实现：

```
<!--html-->
<div class="container">
  <div class="curved-bg2"></div>
</div>
// css
.container {
  width: 300px;
  height: 300px;
  overflow: hidden;
}
.curved-bg2 {
    height: 100%;
    margin-top: -50%;
    background-color: rgba(0, 0, 0, 0.3);
    /* Curved corners */
    border-bottom-left-radius: 50% 10%;
    border-bottom-right-radius: 50% 10%;
}
```

利用了border-radius的裁剪形状。具体参考[border-top-left-radius](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-top-left-radius)

以上2种方法渲染结果对比：

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-6-2/1622648619027-image.png)

前者的效果两端较为平缓，而curved border-radius的两端比较陡峭，接近于垂直。视觉上curved border-radius效果一般。


## radio-gradient
用一个mask元素盖在banner上层，使用径向渐变，结合pointer-event:none实现点击穿透。
```
.mask {
  position: absolute;
  width: 500px;
  height: 500px;
  left: 0;
  top: 0;
  background: #fff;
  background: radial-gradient(ellipse closest-side at center, transparent 919rpx, #fff 920rpx);
  background-size: 100px 500px;
  background-position: -891rpx -456rpx;

  background-repeat: no-repeat;
  pointer-events: none;
}

```
* ellipse表示形状为椭圆。

* 椭圆怎么定义边界呢？
    * farthest-corner、closest-side 、closest-corner 、farthest-side。
        * closest-side表示background-size指定的背景画布的4条边，水平和垂直方向各自距离椭圆圆心最近的边和这个椭圆边界相切。farthest-side同理。
        * closest-corner首先选择最近的角，作为椭圆和画布边缘的相交点1，然后再在该角的2个邻边分别做相交点1的对称角，称为相交点2和相交点3。有了椭圆圆心，i以及3个交点，可以唯一确定一个椭圆的形状。这一点在很多网站上都没有讲清楚，经过实践总结出来的规律。
![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-6-3/1622731992192-image.png)
    * 除了farthest/closest、side/corner组成的关键字之外。可以具体定义水平和垂直的半径。

* center表示椭圆的圆心在background-size界定的范围中的相对位置。

在给颜色标注位置时，如果是具体数值而不是百分比，则数值是以椭圆水平方向的半径为最大值，MDN有个专业词`Virtual gradient ray`，翻译为虚拟渐变射线：

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-6-3/1622730040664-image.png)

```html
<p><strong>closest-side：</strong></p>
<div id="grad1"></div>
```
```css
#grad1 {
  height: 150px;
  width: 300px;
  background-color: red; /* 浏览器不支持的时候显示 */
  background-image: radial-gradient(closest-side at 50% 50%, red 70px, yellow 80px, black 150px); 
}
```

## 水波纹效果

在寻找方案的时候看到了一个有意思的水波纹。

![](https://sharemeans.oss-cn-guangzhou.aliyuncs.com/picture/2021-6-2/1622649326314-image.png)

实现原理就是多个方形容器旋转。
```
<div class="wave">
    水波纹效果
    <div class="wave1"></div>
    <div class="wave2"></div>
    <div class="wave3"></div>
</div>

.wave{
    position: relative;
    border: 1px solid silver;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    line-height: 50px;
    margin: 0 auto;
    font-size: 14px;
    text-align: center;
    overflow: hidden;
    animation: water-wave linear infinite;
}
.wave1{
    position: absolute;
    top: 40%;
    left: -25%;
    background: #33cfff;
    opacity: .7;
    width: 200%;
    height: 200%;
    border-radius: 40%;
    animation: inherit;
    animation-duration: 5s;
}
.wave2{
    position: absolute;
    top: 40%;
    left: -35%;
    background: #0eaffe;
    opacity: .7;
    width: 200%;
    height: 200%;
    border-radius: 35%;
    animation: inherit;
    animation-duration: 7s;
}
.wave3{
    position: absolute;
    top: 50%;
    left: -35%;
    background: #0f7ea4;
    opacity: .3;
    width: 200%;
    height: 200%;
    border-radius: 33%;
    animation: inherit;
    animation-duration: 11s;
}
@keyframes  water-wave{
    0% {transform: rotate(0deg);}
    100% {transform: rotate(360deg);}
}
```

