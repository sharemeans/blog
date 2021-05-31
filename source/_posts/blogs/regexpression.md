---
title: 有趣的正则表达式
categories: js
date: 2019-4-16
--- 

## 字符串拼接

假设有这样一种场景：

有三个字符串类型的变量，姓名(name)，性别(sex)，年龄(age)。想把这三个变量值用逗号拼接成1个字符串userInfo。

常规的方法可能就是类似这样：
```
let userInfo = `${name}，${sex}，${age}`
```
问题是，这三个变量可能都为空。你肯定不希望userInfo是"，，10"或者"小明，，10"或者"，，"

简单的思考后你可能会这样做：
```
let userInfo = ''
name ? userInfo += name + '，' :
sex ? userInfo += sex + '，':
age ? userInfo += age + '，':
```
本着足够懒的想法，仔细考虑了一番之后，我总结出的规律就是：
1. 开头结尾的连接符删除
2. 中间有2个或以上的连接符都要删减至1个

所以可以这样写：
```
let userInfo = `${name}，${sex}，${age}`.replace(/(^，)|(，$)/g, '').replace(/，{2,}/, '，')
```



