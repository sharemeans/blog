---
title: linux常见命令
categories: linux
date: 2019-12-6
--- 

## 查看端口占用情况

> sudo netstat -apn | grep 80

![](/images/2019060301.png)

最后一列可以看到进程号

## 查看ngin端口启用状态：
> ps -ef | grep nginx

## 查看端口进程

上一步的进程号可以查出是哪个应用占用了端口

> ps -ef | grep 3345

## 杀掉进程

> sudo kill -9 3345

-9只是一个信号，表示强制终止，不管正在终止的进程在做什么

## 查看或编辑nginx配置
```
 vi /usr/local/nginx/conf/nginx.conf
 vi nginx.conf
```

## 重启服务器：
> /usr/local/nginx/sbin/nginx -s reload

## 连接远程服务器:
ssh root@192.168.1.1

## 上传文件夹到远程目录：
scp -r /usr/xxx root@192.168.1.1:/opt/project
这个命令执行的结果是，xxx目录会存在于服务器project目录下

## 删除文件夹：
> rm -rf dist
## 移动所有文件到上一个目录：
> mv * ../
## 查看当前目录：
> pwd

## 查看用户：
> cat /etc/passwd

## 查看用户组：
> cat /etc/group

## 查看用户属于哪个组：
> id root