---
title: 判断浏览器是否是隐私模式
categories: javascript
tags: [javascript]
date: 2019-3-7
---

### 很旧的版本

较旧的版本，无痕模式禁用了FileSystem API。
```js
var fs = window.RequestFileSystem || window.webkitRequestFileSystem;
if (!fs) {
  console.log("check failed?");
} else {
  fs(window.TEMPORARY,
      100,
      console.log.bind(console, "not in incognito mode"),
      console.log.bind(console, "incognito mode"));
}
```

但是据说现在不能用了。

### 新一点的
无痕模式下，TEMPORARY 存储配额较低。[参考文章](https://mishravikas.com/articles/2019-07/bypassing-anti-incognito-detection-google-chrome.html)

```js
if ('storage' in navigator && 'estimate' in navigator.storage) {
    const {usage, quota} = await navigator.storage.estimate();
    console.log(`Using ${usage} out of ${quota} bytes.`);

    if(quota < 120000000){
        console.log('Incognito')
    } else {
        console.log('Not Incognito')
    }   
} else {
    console.log('Can not detect')
}
```
但是个人使用chrome实践之后发现这个方法其实也不行，隐私模式下输出结果为：
```
Using 0 out of 536504813 bytes.
```

### 现在的无痕模式

看到stackoverflow的一个[回答](https://stackoverflow.com/questions/2860879/detecting-if-a-browser-is-using-private-browsing-mode)解释说，chrome76+的版本都不允许任何方式检测无痕模式。

看看现在chrome对无痕模式对解释：

> 在无痕模式下，您的浏览记录、Cookie、网站数据以及您在表单中输入的信息都不会保存到您的设备中。也就是说，您的活动不会显示在 Chrome 浏览器的历史记录中，因此与您共用设备的人不会看到您的活动。网站会将您视为新用户；只要您不登录，网站就无法确定您的身份。

localstorage和sessionStorage还是可以用的，只不过cookie不会保存。等下，cookie不会保存是什么概念，我们再看看：

> 每次您关闭所有无痕式窗口时，Chrome 都会舍弃与此浏览会话相关的所有网站数据和 Cookie。

也就是说，cookie用是可以用，但是窗口关闭的话不会保存。如果我们cookie的有效期不是session类型的，会受到影响。本来你的cookie设定7天后过期，结果今天用完关闭窗口cookie就没了。这样做也正是无痕模式的初衷，而且不影响使用。

### 总结

现在的浏览器已经没有必要判断无痕模式了。目前的无痕模式的重点在于使用后不留下记录，不影响使用过程。



