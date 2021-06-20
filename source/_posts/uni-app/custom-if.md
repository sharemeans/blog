---
title: uni-app 自定义条件编译
categories: uni-app
tags: [条件编译, uni-app]
date: 2021-4-7
---   

package.json文件中添加如下配置：
```
"uni-app": {
    "scripts": {
      "mp-qly": {
        "title": "趣旅游小程序",
        "BROWSER": "",
        "env": {
          "UNI_PLATFORM": "mp-weixin"
        },
        "define": {
          "QLY": true
        }
      },
      "h5-qly": {
        "title": "趣旅游h5",
        "BROWSER": "",
        "env": {
          "UNI_PLATFORM": "h5"
        },
        "define": {
          "QLY": true
        }
      }
    }
}
```

在npm script中带上custom参数：
```
"dev:wx:qly": "uniapp-cli custom mp-qly",
"dev:h5:qly": "uniapp-cli custom h5-qly"
```

那么，这俩命令执行后
```
// #ifdef QLY
// #endif
```
这个条件编译是都会命中的。自定义条件编译适合在saas应用中针对业务做区分，同一个业务不同的平台保持一致性。

自定义条件编译有个缺点，就是不支持or判断
```
// #ifdef BOOKING || QLY
dosomething();
// #endif
```

以上条件编译只会命中BOOKING。不会命中QLY。

解决办法
1. 按照[官方的办法](https://github.com/dcloudio/uni-app/issues/1008#issuecomment-555409355)处理。
2. 分开处理
```
// #ifdef BOOKING
dosomething();
// #endif
// #ifdef QLY
dosomething();
// #endif
```

如果是ifndef呢？既不是，也不是。
```
// #ifndef BOOKING || QLY
dosomething();
// #endif
```
以上写法是不对的。可以通过嵌套解决：
```
// #ifndef BOOKING
// #ifndef QLY
dosomething();
// #endif
// #endif
```