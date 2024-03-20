---
title: go-zero 生成和打包api和rpc
categories:
  - Go
tags:
  - go-zero
toc: true # 是否启用内容索引
---

### api

##### 1.goctl 生成order.api

````
goctl api go -api order.api -dir .
````

##### 2.构建 order-api 二进制 文件

````
cd api

GOOS=linux GOARCH=amd64 go build -o order-api order.go
````

##### 3.构建order-api镜像

````
docker build -t order-api:1.4.0 .
````

##### 4.保存镜像

````
docker save -o order-api.img order-api:1.4.0
````



#### rpc

##### 1.goctl生成order.porto

````
goctl rpc protoc order.proto --go_out=./types --go-grpc_out=./types --zrpc_out=.
````

##### 2.构建 order-rpc 二进制 文件

````
cd rpc

GOOS=linux GOARCH=amd64 go build -o order-rpc order.go
````

##### 3.构建announcement-api镜像

````
docker build -t order-rpc:1.4.0 .
````

##### 4.保存镜像

````
docker save -o order-rpc.img order-rpc:1.4.0
````

