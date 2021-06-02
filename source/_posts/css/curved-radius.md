---
title: 凹槽样式小结
categories: css
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

## curved border-radius

以上方法只是对一个简单的背景做圆角处理。针对以下场景，圆弧部分只是一个修饰，图片部分的轮播图可点击交互。

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-2/1622647130156-image.png)

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

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-2/1622648619027-image.png)

前者的效果两端较为平缓，而curved border-radius的两端比较陡峭，接近于垂直。

## 切图

从上面的对比结果来看，curved border-radius无法很好的实现我们想要的效果，那么，还有一个简单的办法，就是将有弧度的部分切成一张png图盖在banner上。

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-2/1622648935597-image.png)

搞了半天还是这个办法最简单啦。

## 水波纹效果

在寻找方案的时候看到了一个有意思的水波纹。

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-2/1622649326314-image.png)

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

