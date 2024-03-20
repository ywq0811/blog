---
title: windows10 安装和使用 ssh
categories:
  - Windows
tags:
  - node
toc: true # 是否启用内容索引
---

## windows10 安装和使用 ssh

#### 安装

1.下载文件

下载地址：https://github.com/PowerShell/Win32-OpenSSH/releases  本人电脑64位

![image-20240320161041186](/imgs/image-20240320161041186.png)

2.安装

将这个下载好的压缩包，解压到C:\Program Files目录下

![image-20240320161153808](/imgs/image-20240320161153808.png)

3.配置到系统环境变量中

![image-20240320161257662](/imgs/image-20240320161257662.png)

4.进入到 C盘 -> Users ->对应的用户中 执行cmd运行ssh测试

![image-20240320161630214](/imgs/image-20240320161630214.png)

------



#### 生成SSH key

1. cmd中执行命令生成密钥

```shell
ssh-keygen -t rsa -C "your_email@example.com"
```

![image-20240320162116918](/imgs/image-20240320162116918.png)

出现这个图说明生成成功

2.进入C盘 -> Users -> 对应用户中查看,会生成一个 `.ssh` 文件夹

![image-20240320162225264](/imgs/image-20240320162225264.png)

3.读取公钥并添加到github中

![image-20240320162410847](/imgs/image-20240320162410847.png)

4.登录到 `github.com` 中 添加ssh key

![image-20240320162918218](/imgs/image-20240320162918218.png)

![image-20240320163038687](/imgs/image-20240320163038687.png)

5.对此从指定github上拉取项目就不需要验证了