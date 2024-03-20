---
title: wsl2-Ubuntu 安装 go
categories:
  - Ubuntu
tags:
  - wsl2
  - go
toc: true # 是否启用内容索引
---

## windows安装sqlite以及使用文档

1.重装ubuntu18.04

2.sudo su 进入root权限

3.  cd /mnt/d/share  进入到d盘share文件夹下

4. cp go1.16.6.linux-amd64.tar.gz /usr/local/  讲share文件夹下的go.tar拷贝到/usr/local 目录下   

5.cd /usr/local

6.tar -xvf go.tar  解压go.tar文件

7.rm -rf go.tar

8.vim /etc/profile
添加：
export GOROOT=/usr/local/go
export GOPATH=/mnt/d/vagrant/data/gopath/src
export GOPROXY=http://goproxy.cn
export GOSUMDB=goproxy.cn
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
export SHARE=/mnt/d/share

9.source /etc/profile

10.go version