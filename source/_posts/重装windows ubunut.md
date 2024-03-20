---
title: 重装 windows ubuntu
categories:
  - Windows
tags:
  - ubuntu
toc: true # 是否启用内容索引
---

## 重装 windows ubuntu

1.https://www.yuque.com/shaycormac/blog/tn88q0 根据这个文档安装wls2和docker destop



2.windows商店下载terminal

3.配置/etc/profile

4.保存配置 source profile

5.下载go

6.更新源

https://www.cnblogs.com/ssxblog/p/11357126.html



7.安装mingw

````
apt install gcc
````

8.开始编译二进制文件（刚从gitlub clone下来需要go mod tidy）



9使用`make image`命令需要先安装apt install make 和 apt install make-guile ,可以按要求直接安装.



10.需要加载基础镜像`kvp-centos-ffmpeg-minial.img`

(在/d/java/share里面),然后执行`docker load -i kvp-centos-ffmpeg-minial.img`

