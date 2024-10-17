---
title: uos-arm64 项目国产化部署
categories:
  - Linux
tags:
  - arm64
  - docker
  - sqlite
  - 达梦数据库
toc: true # 是否启用内容索引
---

## 国产化部署

说明：在 `Uos` 操作系统`arm64`架构服务器上实现项目国产化。

实现列表：

1. 应用层
2. Sqlite
3. 达梦数据库

------

#### 应用层

##### 可执行文件启动

说明：由于这个服务需要用到Sqlite，并且需要在linux/arm64服务器上执行。而我们的项目在windos环境下开发，就需要使用到交叉编译。由于安装了桌面版的ubuntu，可以通过linux环境访问到window路径。

- 安装 `arm64` 交叉编译工具链

```shell
sudo apt-get install gcc-aarch64-linux-gnu
```

- 设置环境变量（设置环境变量以指示编译器和链接器使用arm64交叉编译工具链）

```shell
export CC=aarch64-linux-gnu-gcc
export CXX=aarch64-linux-gnu-g++
```

- 通过linux环境进入到项目目录，交叉编译项目（以 `unix-test` 项目举例）

```go
cd unix-test

GOOS=linux GOARCH=arm64 CGO_ENABLED=1 CC=aarch64-linux-gnu-gcc go build -v --ldflags="-X 'google.golang.org/protobuf/reflect/protoregistry.conflictPolicy=warn'" -o unix-test cmd/main.go
```

- 成功编译后，将可执行文件拷入 `linx/arm64` 服务器上
- 升级权限 

```shell
chmod +x unix-test
```

- 执行可执行文件

```shell
./unix-test
```

![image-20240315163638187](/imgs/image-20240315163638187.png)

可以看到使用的 `Sqlite` 数据库，连接成功，并且调用接口成功。

------

##### docker启动

在上面可执行文件基础上，使用arm64系统的基础镜像，进行docker部署

- 进入镜像仓库地址选择满足arm64版本的基础镜像

```go
https://hub.docker.com
```

- 在terminal(终端)上执行复制的命令(我使用的devel版本，可自行选择版本下载)

```go
docker pull arm64v8/ubuntu:devel
```

注：这里试过无数个 `busybox` 基础镜像版本，都不行；`arm64v8/centos:latest` 测试过也是可以的，但是被弃用了，所以就使用ubuntu了

- 将 `Dockerfile` 文件拷贝到服务器可执行文件 `unix-test`目录下， 修改`FROM` 

```go
FROM arm64v8/ubuntu:devel
```

- 构建镜像（演示的项目为unix-test）

```shell
docker build -t unix-test:1.0.0 .
```

- 执行docker run 命令启动服务

![image-20240407155259272](/imgs/image-20240407155259272.png)

------

#### Sqlite

再windows环境下直接生成db文件即可。将db文件拷入 `linux/arm64`服务对应文件，读取即可。

- 再windows下进入Sqlite目录下

```go
Sqlite3
```

- 生成db文件（以 `unix-test.db` 为例）

```shell
.open unix-test.db
```

- 再 `sqlite` 目录下会生成一个 `unix-test.db` 文件，拷入 `linux/arm64` 服务器对应位置

------

 

#### 达梦数据库

需要下载使用arm64系统的镜像

- 在镜像仓库中找到满足的镜像

```go
https://hub.docker.com/r/qinchz/dm8-arm64

我选择的是dm8
```

- 在terminal(终端)上执行复制的命令（镜像很大，下载很慢）

```go
docker pull qinchz/dm8-arm64
```

- 将基础镜像保存到本地

```go
docker save -o dm8-arm64.img qinchz/dm8-arm64:latest
```

- 将镜像拷入到arm64服务器
- 创建docker-compose.yml文件

vim docker-compose.yml

```go
version: '3'
services:
  DM8:
    image: qinchz/dm8-arm64:latest
    restart: always
    container_name: dm8
    ports:
      - "5236:5236"
    volumes:
      - /home/test/kst/yanwq/md8/data:/home/dmdba/data
```

参数说明：

- `version: '3'`：指定使用的Docker Compose文件版本
- `services`：定义服务的部分开始
- `DM8`：定义服务的名称为DM8
- `image`：指定使用的镜像为qinchz/dm8-arm64:latest
- `restart: always`：指定容器在退出时自动重启
- `container_name: dm8`：指定创建的容器名称为dm8
- `ports`：指定容器端口映射关系
- `5236:5236`：将主机的5236端口映射到容器的5236端口
- `mem_limit: 1g`：限制容器使用的内存为1GB
- `memswap_limit: 1g`：限制容器可以使用的swap交换空间为1GB
- `volumes`：指定挂载卷的配置
- `/home/test/kst/yanwq/md8/data:/home/dmdba/data`：将主机的/home/test/kst/yanwq/md8/data目录挂载到容器的/home/dmdba/data目录，实现主机和容器之间的文件共享

------

- 启动服务

```go
docker-compose up -d
```

















[使用DBeaver运行达梦数据库](https://blog.csdn.net/qq_57484285/article/details/128191061?spm=1001.2101.3001.6650.7&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-7-128191061-blog-123725042.235%5Ev43%5Epc_blog_bottom_relevance_base4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-7-128191061-blog-123725042.235%5Ev43%5Epc_blog_bottom_relevance_base4)