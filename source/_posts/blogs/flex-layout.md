---
title: flex上中下布局
categories: css
date: 2019-5-10
---

此文档只是说明如何实现，采用最新的弹性盒子布局规范，对旧写法不做考虑。

## 盒子组成

```
<body class="flex-ver">
    <div class="header red">header</div>
    <div class="container flex-auto orange">
        <div>content</div>
        <div>content</div>
        <div>content</div>
        <div>content</div>
    </div>
    <div class="footer blue">footer</div>
</body>
```
## 步骤

### 步骤一：body元素设置为flex容器，纵向主轴


```
.flex-ver {
    display: flex;
    flex-direction: column;
}
```

### 步骤二：header和footer固定高度

```
.header {
    height: 150px;
}
.footer {
    height: 100px;
}
```


### 步骤三：主体区域container设置为占用剩余高度


```
.flex-auto {
  flex-grow: 1;
  flex-basis: 0%;
}
```
## 完整代码

```
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="./flex.css">
</head>
<body class="flex-ver">
    <div class="header red">header</div>
    <div class="container flex-auto orange">
        <div>content</div>
        <div>content</div>
        <div>content</div>
        <div>content</div>
    </div>
    <div class="footer blue">footer</div>
</body>
<style>
    .header {
        height: 150px;
    }
    .footer {
        height: 100px;
    }
    .flex-item {
        height: auto;
    }
    .flex {
        display: flex;
        flex-direction: row;
    }
    .flex-ver {
        display: flex;
        flex-direction: column;
    }
    .red {
        background: red;
    }
    .blue {
        background: blue;
    }
    .black {
        background: black;
    }
    .orange {
        background: orange;
    }
</style>
</html>

```

## 整体效果

1. 主体内容高度自适应
2. 底部footer位于文档最底部，即使主体区域高度不足以占满视口，footer依旧贴在底部。

## 兼容性

火狐浏览器未能实现该效果，主体内容的高度根据内容自适应的，body的整体高度有可能低于视口高度。

## 解决火狐浏览器兼容性问题

注意，html不要设置为overflow: hidden，要设置为overflow:visible。

```
html, body , div {
    padding: 0;
    margin: 0;
}
html {
    height: 100%;
}
body {
    min-height: 100%
}
```


