---
title: docker部署达梦数据库文档
categories:
  - Docker
tags:
  - dm8
toc: true # 是否启用内容索引
---

## docker部署达梦数据库文档

#### 1.下载镜像

请在达梦数据库官网下载 [Docker 安装包](https://eco.dameng.com/download/)。



#### 2.导入镜像

拷贝安装包到你中意的目录下，执行以下命令导入安装包：

```go
docker load -i dm8_20220822_rev166351_x86_rh6_64_ctm.tar
```

结果显示如下：

![image-20221019113036523](/imgs/image-20221019113036523.png)

导入完成后，可以使用 `docker images` 查看导入的镜像。结果显示如下：

![image-20221019113146223](/imgs/image-20221019113146223.png)



#### 3.启动容器

镜像导入后，使用 `docker run` 启动容器，启动命令如下：

```go
docker run -d -p 5236:5236 --restart=always --name dm8_01 --privileged=true -e PAGE_SIZE=16 -e LD_LIBRARY_PATH=/opt/dmdbms/bin -e INSTANCE_NAME=dm8_01 -v /home/dm8:/opt/dmdbms/data dm8_single:v8.1.2.128_ent_x86_64_ctm_pack4
```

结果显示如下：

![image-20221019113511872](/imgs/image-20221019113511872.png)

容器启动完成后，使用 `docker ps`  查看镜像的启动情况，结果显示如下：

![image-20221019113550760](/imgs/image-20221019113550760.png)

启动完成后，可通过日志检查启动情况，命令如下：

```go
docker logs -f dm8_01
```

结果显示如下：

![image-20221019113703773](/imgs/image-20221019113703773.png)



#### 4.进入容器

容器启动成功后，执行以下命令进入容器:

```go
docker exec -it dm8_01 /bin/bash
```

结果显示如下：

![image-20221019114115438](/imgs/image-20221019114115438.png)



#### 5.进入disql验证

进入容器后，先执行以下命令防止中文乱码：

```go
source /etc/profile
```

结果显示如下：

![image-20221019114345261](/imgs/image-20221019114345261.png)

进入disql目录，执行以下命令：

```go
cd /opt/dmdbms/bin
```

结果显示如下：

![image-20221019114815752](/imgs/image-20221019114815752.png)

进入disql，执行以下命令

```go
./disql
```

结果显示如下：

![image-20221019115006081](/imgs/image-20221019115006081.png)

输入用户名和密码进行验证：

```go
username:SYSDBA
password:SYSDBA001

提示：新版本 Docker 镜像中数据库默认用户名/密码为SYSDBA/SYSDBA001
```

结果显示如下：

![image-20221019115225323](/imgs/image-20221019115225323.png)



**参考**

docker安装DM数据库可参考[达梦技术文档]https://eco.dameng.com/document/dm/zh-cn/start/dm-install-docker.html

