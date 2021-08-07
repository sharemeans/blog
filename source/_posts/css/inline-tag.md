---
title: 纯css实现行内标签
categories: css
tags: [css]
date: 2020-6-18
--- 

有一个需求要实现下图的布局效果：

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-15/1623756154070-image.png)

商品标签和商品标题融合在一起。

## float元素

一想到文字环绕，一开始想到的就是float。
* 彩色标签使用float:left
* 文字使用inline或者block

文字为什么不能用inline-block呢？
```
<style>
   .left {
       float: left;
       color: red;
   }
   .right {
       display: inline-block;
   }
</style>
<div class="page">
    <div class="left">特别长特别长</div>
    <div class="right">特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长特别长</div>
</div>
```
结果：

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-18/1623999303996-image.png)

可见，float:left和display:inline-block并列时，后者会换行。MDN解释说`float意味着使用块布局`。用这句话解释上面的情况说得过去。

但是，如果把right改成下面这样：
```
.right {
   display: block;
   <!--或者-->
   display: inline;
}
</style>
```
就会变成

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-18/1623999523016-image.png)

看来`float意味着使用块布局`这一点还要结合`float本身是被用来设计文字环绕`的说法结合才能解释的通float的行为。

## 实现文字环绕

关于文字环绕使用inline还是block呢？

如果想用inline，text-align就不生效。有2个选择：
1. text-align作用到父元素
2. 使用文字使用block+text-align

```
<template>
<div class="product-item-title">
  <span class="category-tag">{{product.categoryName}}</span>
  <span class="product-name">{{product.productName}}</span>
</div>
</template>

<style lang="less">
// 方法1
.product-item-title{
    text-align: justify;

    .category-tag {
      float: left;
      // 高度和位置可能需要微调
      // ...
    }

    .product-name {
      line-height: 40rpx;
    }
}

// 方法2
.product-item-title{
    .category-tag {
      float: left;
      // 高度和位置可能需要微调
      // ...
    }

    .product-name {
      display:block;
      line-height: 40rpx;
      text-align: justify;
    }
}
</style>
```

## 多行省略的文字环绕
如果需求再升级：

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-18/1623999965411-image.png)

多行省略使用了box布局：
```
/*多行溢出显示省略*/
.muti-line-ellipsis(@row: 2){
  -webkit-line-clamp: @row;
  -webkit-box-orient: vertical;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;  
  white-space: pre-wrap!important;
}
```

采用前面的方法1，将muti-line-ellipsis作用在product-item-title上，在小程序上有问题，h5没问题。

采用前面的方法2，将muti-line-ellipsis作用在product-name上，在小程序上有问题，h5没问题。

二者均没有文字环绕效果，看来float:left的方法需要弃用了。

## inline

研究了下，发现，其实多个inline元素也是可以实现文字环绕效果的，因此有了接下来的最佳实践：

```
<template>
<div class="product-item-title"><span
class="category-tag">{{product.categoryName}}</span><span
class="product-name">{{product.productName}}</span>
</div>
</template>

<style lang="less">
.product-item-title{
    text-align: justify;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;  
    white-space: pre-wrap!important;

    .category-tag {
    }

    .product-name {
    }
}
</style>
```

需要注意的是，product-item-title内、元素之间不能有换行，否则行内元素就不能在同一行了。效果如下：

![](https://gitee.com/ndrkjvmkl/picture/raw/master/2021-6-18/1624008388023-image.png)