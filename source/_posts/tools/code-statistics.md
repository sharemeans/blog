---
title: git 统计代码量
categories: 工具
tags: [git]
date: 2021-7-7
--- 


```
git log --format='%aN' | sort -u | while read name; do echo -en "{\"name\":\"$name\",\t"; git log --author="$name" --since='2021-04-01' --until='2021-06-30' --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 + $2 } END { printf "\"add\": %s, \"remove\": %s, \"result\": %s}\n", add, subs, loc }' -; done
```

以上方法有缺陷：
- 只能统计所有人在某个仓库下、特定分支、特定时间段的代码量，通常统计master
- 如果是多版本并行开发，若存在分支未合并到master的情况则统计不准确
- 需要统计者先拉取仓库到本地

代码量统计本身就是个比较不人性化的评估标准，先占个坑，如果真有看到有人写出一个完整的统计脚本或者自己有空就再说。

