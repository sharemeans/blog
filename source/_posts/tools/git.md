---
title: git 常用命令
categories: 工具
tags: [git]
date: 2019-12-1
---

## 本地代码新建远程仓库

如果本地配置的ssh 是其它域名下的，比如说公司内网gitlab，但是你此时想要创建的仓库是github上的，那代码推到远程仓库的时候需要使用https协议，而不是ssh，不然的话身份认证会失败的。

```
git@github.com: Permission denied (publickey).
```

##### 远程：
创建一个仓库， 假如链接为
```
https://github.com/sharemeans/vue.git/
```

##### 本地：
```
cd my-vue
git init
git add .
git commit -m "项目初始化"
git remote add origin https://github.com/sharemeans/vue.git/
// 根据提示输入账号密码，成功之后代码自动上传
// 代码推送到远程 -u表示推送成功后自动建立本地分支与远程分支的追踪
git push -u origin master
```

## 查看远程仓库地址
```
git remote -v
```

## 查看本地分支跟踪的远程分支
```
git branch -vv
```
## 基于远程分支A创建本地跟踪分支A
```
git checkout -b A origin/A
```

## 基于本地分支B创建本地分支A
```
git checkout -b A B
```

## 基于远程分支创建本地同名跟踪分支
```
git checkout --track origin/A
```

## 合并A 分支到当前分支，且所有冲突都采用A分支
```
git pull -s recursive -X theirs A
```

## 恢复被删除的stash
1. 输出最近删除的stash
```
git fsck --lost-found
/**
* dangling commit 42c7122df57f326a6e1e0498fb89af06ab521192
*/
```
2. 从输出列表中找出需要恢复的id：
```
git stash apply 42c7122df57f326a6e1e0498fb89af06ab521192
```

## 重命名当前分支

```
// 重命名为main
git branch -m main
git branch -M Main
```

## 打tag
1. 切换到要打tag的分支
2. 基于某个commit打tag
```
git tag -a [tagname] -m [tagmessage]
```
3. 提交tag到远程
```
git push origin [tagname]
```
4. 提交所有tag到远程
```
git push origin --tags
// 或者
git push --tags
```
5. 删除tag
```
// 第一步：删除本地tag
git tag -d [tagname]
// 第二步：推到远程
git push origin :refs/tags/[tagname]
```

## vscode git密码重置
vscode会帮我们记住git账号密码。当git账号变更导致vscode提示身份校验失败时，需要重置密码。
```
// 清空密码
git config --system --unset credential.helper
```

重置后，再次提交代码，会弹窗提示输入新的账号密码

## 参考资料

http://blog.codingplayboy.com/2017/04/06/git_branch/
http://blog.codingplayboy.com/2017/04/14/git_rebase/#rebase