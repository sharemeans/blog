---
title: git分支中的斜杠什么时候能用
categories: 工具
tags: [git]
date: 2021-9-26
---

#### 背景

在一次使用Jenkins部署时，出现了以下报错：

```
From http://gzgit.bestwehotel.com/frontend/wehotel-hotel-booking
   ed0c693..8c60655  feature/travel-saas-2.8.0 -> origin/feature/travel-saas-2.8.0
error: unable to resolve reference refs/remotes/origin/test/yangchun: Not a directory
 ! [new branch]      test/yangchun -> origin/test/yangchun  (unable to update local ref)
error: some local refs could not be updated; try running
 'git remote prune http://gzgit.bestwehotel.com/frontend/wehotel-hotel-booking.git' to remove any old, conflicting branches
```

这里面最核心的错误是`refs/remotes/origin/test/yangchun: Not a directory`



#### 分析

###### 代码构建选择的分支并非报错的分支

报错前执行了：

```
git fetch --tags --progress http://gzgit.bestwehotel.com/frontend/wehotel-hotel-booking.git +refs/heads/*:refs/remotes/origin/* # timeout=10
```

这个命令会获取所有远程分支，包括新建分支，而这个分支就是最近某个同事新建的，在Jenkins服务器上并不存在。在拉取这个分支时出现了问题才导致报错。

###### 具体错误是什么原因导致的呢？

新增的分支名为`remotes/origin`。我们开发过程中经常会使用`feature/myName`作为分支名，这种就没有问题，因此`/`符号作为分隔符是没有问题的。

去网上搜了一下，大致的结论是：
> 目录名称和文件名称不能一样

打开自己本地的`.git`目录会发现，凡是以`/`作为分隔符命名的分支，在`.git`目录下都是以目录的形式保留分支。如`feature/v2.0.0`分支，会在`.git/feature`目录下新建一个名为`v2.0.0`的文件。

既然如此，最开始遇到的报错，就应该是在创建文件的过程中出现的，根据错误信息，推测出来，`test`目录在创建过程中出错，因为遇到了1个同名的文件。

我去git仓库找了下，没发现有这个分支，那只剩下1种可能：
> test 分支之前是存在的，只是后来被删除了。

###### 解决办法

若是这个分支一定要保留，则需要清除Jenkins服务器上缓存的remotes/origin/*文件：

```
git remote prune origin
```
