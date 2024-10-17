---
title: go通过gorm使用达梦数据库
categories:
  - Go
tags:
  - gorm
  - 达梦数据库
toc: true # 是否启用内容索引
---


## go通过gorm使用达梦数据库

根据官方文档说明，go想要通过gorm使用达梦数据库，需要下载orm方言包。

![image-20240410111239029](/imgs/image-20240410111239029.png)

如图所示，达梦官方提供了2个版本的orm方言包

-  V1：github.com/jinzhu/gorm
- V2：gorm.io/gorm

本文档演示使用的是V2版本方言包

------

#### 准备

- `dm` 驱动包 + 依赖
- v2方言包

go官方驱动地址：https://download.dameng.com/eco/adapter/resource/go/dm-go-driver.zip

下载好解压目录如下：

- dm-go-driver.zip  // 驱动包
- gorm_v1_dialect.zip  // v1方言包
- gorm_v2_dialect.zip  // v2方言包

**步骤**

1. 将 `dm-go-driver.zip` 解压之后，把 `dm` 包拷入到 `GOPATH`的`src` 目录下 【这个是dm驱动包】

2. 在 `src` 目录下 终端下载依赖包

   ```shell
   go get github.com/golang/snappy
   
   go get golang.org/x/text/encoding
   ```

3. 创建项目（演示的demo项目为 `dm8-test`）

4. 将 `gorm_v2_dialect.zip` 解压之后，把`dm` 包拷入到`dm8-test` 项目 目录下（查看下面结构树）【这个是dm方言包】

   此时，目录树如下

   src
   ├── dm  // 驱动包
   ├── dm8-test  // 演示demo
   │   ├── dm  // 方言包
   │   │   ├── create.go
   │   │   ├── dm.go
   │   │   ├── dm_test.go
   │   │   └── migrator.go
   │   ├── go.mod
   │   ├── go.sum
   │   └── main.go
   ├── github.com // 依赖包
   └── golang.org  // 依赖包

5. 打开项目（演示的demo项目为 `dm8-test`）-> `go.mod` -> 新增`replace dm => ../dm` （指定dm包的来源路径）

![image-20240410114700871](/imgs/image-20240410114700871.png)

------

#### 连接

在 `main.go` 中测试连接

```go
package main

import (
	"dm8-test/dm"
	"fmt"
	"gorm.io/gorm"
)

func main() {
	_, err := gorm.Open(dm.Open("dm://SYSDBA:SYSDBA@dm数据库IP:5236"), &gorm.Config{})
	if err != nil {
		panic("failed to connect database")
	}

	fmt.Println("connect dm8 success!!")

}
```

![image-20240410115921736](/imgs/image-20240410115921736.png)

#### 其他

如果需要封装达梦数据库到组件中，可以把dm驱动包 和 dm方言包拷入对应组件中，记得需要删除dm驱动包中的 `go.mod` 和 `.idea` ，加到组件包的 `go.mod`中即可，同时还需要修改dm方言包中的引用dm驱动包的路径（除去GOPATH的实际路径即可）

![image-20240410184441301](/imgs/image-20240410184441301.png)