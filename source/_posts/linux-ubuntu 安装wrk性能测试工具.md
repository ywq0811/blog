---
title: linux-ubuntu 安装wrk性能测试工具
categories:
  - linux
tags:
  - wrk
toc: true # 是否启用内容索引
---

## linux-ubuntu 安装wrk性能测试工具

1.若没有安装git，则安装apt-get git

2.从git上拉取wrk

```go
git clone https://github.com/wg/wrk
```

3.可能需要的依赖

```go
# 安装 make 工具
sudo apt-get install make
 
# 安装 gcc编译环境
sudo apt-get install build-essential
 
# 安装openssl的库
sudo apt-get install libssl-dev 
 
# 安装 压缩解压的库
sudo apt-get install zip
```

4.运行wrk

```go
cd wrk

make

#可以使用多线程编译来加快速度
make -j8, 8 表示 8个线程一起编译
```

