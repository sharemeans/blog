---
title: white-space属性值表现
categories: css
tags: [css]
date: 2021-7-7
--- 

属性值 | 换行符 | 空格和制表符 | 文字换行 | 行尾空格
---|---|---|---|---
normal | 当作空白符，连续合并 | 连续合并 | 自动换行 | 删除
nowrap | 当作空白符，连续合并 | 连续合并 | 不自动换行 | 删除
pre | 保留，遇到即换行 | 保留 | 不自动换行 | 保留 
pre-wrap | 保留，遇到即换行 | 保留 | 自动换行 |	挂起
pre-line | 保留，遇到即换行 | 连续合并 | 自动换行 |	删除
break-spaces | 保留，遇到即换行 | 保留 | 自动换行 |	换行
