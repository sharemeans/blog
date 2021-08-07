---
title: margin-bottom在ios设备的失效问题
categories: css
tags: [css]
date: 2021-8-1
--- 

以下代码，html和body高度都是100%。子元素son被内容撑开高度。

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
    <div class="son">
      The footer is fixed at the bottom and supossed to revealed with the scrolling, so the previous section has a margin-bottom but it doesn't work, only in Safari. Everywhere else is ok, even in I.E. I tried to add overflow: auto in the page-wrapper, but everything gets weird in all browsers with elements dissapear and appear. I also have read that removing height: 100% in the body and html may fix that, but that is not an option for me, because i need the images to fix the browser height.
      // ...请填充内容至超出一屏高度
    </div>
  <style>
    html, body {
      height: 100%;
      margin: 0;
      padding: 0;
    }
    .son {
      margin-bottom: 100px;
      background-color: cyan;
    }
  </style>
</body>
</html>
```

* 当内容不超过整屏时，通过审查元素可以看到margin存在。
* 当内容超过整屏时（包含margin）,在ios设备中，页面最底部盒子元素的margin-bottom会失效（safari和chrome浏览器都实效）。

如果把html和body的高度删除：

```
html, body {
    margin: 0;
    padding: 0;
}
```
就正常了。

如果改成超出一屏的固定高度，会怎么样呢？
```
html, body {
  height: 1000px;
  margin: 0;
  padding: 0;
}
```
发现margin还是未生效。

因此，对盒子模型的margin折叠行为在ios设备上的表现作出以下猜测：
> 当元素属于页面最后一个子元素时，margin-bottom就会一层一层的渗透到祖先元素，直到html为止。当html高度固定时，子孙元素塌陷出来的margin不会影响它的滚动高度。

解决办法：
* 方案1:不固定html和body的高度，可以只设置min-height:100%。让其被内容自然撑开。
* 方案2:在body内加一个BFC容器，这样就可以防止margin塌陷到外层：

```
<body>
<div class="parent">
    <div class="son">
      // ...
    </div>
</div>
</body>
<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
}
.parent {
  overflow-y: scroll;
}
.son {
  margin-bottom: 100px;
  background-color: cyan;
}
</style>
```
理论上可以直接将body变为BFC，或者给body加个padding-bottom，但是实际上并无效果。因此需要在body内加多一层容器使其成为BFC。