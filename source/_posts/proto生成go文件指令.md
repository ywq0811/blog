---
title: proto生成go文件指令
categories:
  - Proto
tags:
  - proto
toc: true # 是否启用内容索引
---

## proto生成go文件指令

在当前目录下生成xxx.proto的pb文件

```go
protoc --go_out ./ ./xxx.proto
或
protoc --go_out=. --go-grpc_out=. kvps_forward.proto
```



或者

```go
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative  wenet.20230301.proto
```



