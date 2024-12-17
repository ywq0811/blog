---
title: go连接dm8数据库和操作示范文档
categories:
  - Docker
tags:
  - dm8
toc: true # 是否启用内容索引
---

## go连接dm8数据库和操作示范文档

#### 1.安装DM数据库

请参考 [DM 数据库快速上手指南 ](https://eco.dameng.com/document/dm/zh-cn/start/index)或者 `达梦数据库安装和使用文档.md`

数据库安装过程中，请勾选创建 `BOOKSHOP`，`DMHR` 示例库，作为数据库模拟环境，（勾选的目的是为了下面连接dm数据库进行增删改查操作作为示例，不然需要自行创建表和表数据）如下图所示：

![DMHR 示例库](/imgs/java-jdbc-dmhr.png)



#### 2.安装DM驱动包

将提供的 DM 驱动包放在 GOPATH 的 src 目录下。驱动包位于 DM数据库安装目录下：dmdbms/drivers/go/dm-go-driver.zip 解压到 GPPATH 安装路径的 src 下，如图所示：

![image-20221021154102867](/imgs/image-20221021154102867.png)



#### 3.安装依赖包

所需 Go 依赖包有两个，`text` 和 `snappy`，从 Git 上把依赖包 clone 到本地，进入到src目录下，右键【Git Bash Here】打开 Git 命令行窗口，依次下载 text 和 snappy 依赖。

```go
git clone https://github.com/golang/text.git  ./golang.org/x/text

git clone https://github.com/golang/snappy  ./github.com/golang/snappy
```

![image-20221021154442352](/imgs/image-20221021154442352.png)



#### 4.通过go实现连接和操作dm数据库

创建项目，项目结构树如下：

```go
.
├── bin
└── src
    ├── dm    		// dm驱动包
    ├── github.com  // 依赖包
    ├── golang.org	// 依赖包
	└── demo  		// 项目
		├── main.go // 主函数入口
		├── go.mod  
		└── go.sum
```

main.go文件如下：

```go
/*该例程实现插入数据，修改数据，删除数据，数据查询等基本操作。*/
package main
// 引入相关包
import (
"database/sql"
"dm"
"fmt"
"io/ioutil"
"time"
)
var db *sql.DB
var err error
func main() {
	driverName := "dm"
	dataSourceName := "dm://SYSDBA:SYSDBA@localhost:5236"
	if db, err = connect(driverName, dataSourceName); err != nil {
		fmt.Println(err)
		return
	}
	if err = insertTable(); err != nil {
		fmt.Println(err)
		return
	}
	if err = updateTable(); err != nil {
		fmt.Println(err)
		return
	}
	if err = queryTable(); err != nil {
		fmt.Println(err)
		return
	}
	if err = deleteTable(); err != nil {
		fmt.Println(err)
		return
	}
	if err = disconnect(); err != nil {
		fmt.Println(err)
		return
	}
}
/* 创建数据库连接 */
func connect(driverName string, dataSourceName string) (*sql.DB, error) {
	var db *sql.DB
	var err error
	if db, err = sql.Open(driverName, dataSourceName); err != nil {
		return nil, err
	}
	if err = db.Ping(); err != nil {
		return nil, err
	}
	fmt.Printf("connect to \"%s\" succeed.\n", dataSourceName)
	return db, nil
}
/* 往产品信息表插入数据 */
func insertTable() error {
	var inFileName = "D:\\java\\vagrant\\data\\gopath\\src\\demo\\2022\\20221020\\data\\1.png"
	var sql = `INSERT INTO production.product(name,author,publisher,publishtime,
product_subcategoryid,productno,satetystocklevel,originalprice,nowprice,discount,
description,photo,type,papertotal,wordtotal,sellstarttime,sellendtime)
VALUES(:1,:2,:3,:4,:5,:6,:7,:8,:9,:10,:11,:12,:13,:14,:15,:16,:17);`
	data, err := ioutil.ReadFile(inFileName)
	if err != nil {
		return err
	}
	t1, _ := time.Parse("2006-Jan-02", "2005-Apr-01")
	t2, _ := time.Parse("2006-Jan-02", "2006-Mar-20")
	t3, _ := time.Parse("2006-Jan-02", "1900-Jan-01")
	_, err = db.Exec(sql, "三国演义", "罗贯中", "中华书局", t1, 4, "9787101046126", 10, 19.0000, 15.2000,
		8.0,
		"《三国演义》是中国第一部长篇章回体小说，中国小说由短篇发展至长篇的原因与说书有关。",
		data, "25", 943, 93000, t2, t3)
	if err != nil {
		return err
	}
	fmt.Println("insertTable succeed")
	return nil
}
/* 修改产品信息表数据 */
func updateTable() error {
	var sql = "UPDATE production.product SET name = :name WHERE productid = 11;"
	if _, err := db.Exec(sql, "三国演义（上）"); err != nil {
		return err
	}
	fmt.Println("updateTable succeed")
	return nil
}
/* 查询产品信息表 */
func queryTable() error {
	var productid int
	var name string
	var author string
	var description dm.DmClob
	var photo dm.DmBlob
	var sql = "SELECT productid,name,author,description,photo FROM production.product WHERE productid=11"
	rows, err := db.Query(sql)
	if err != nil {
		return err
	}
	defer rows.Close()
	fmt.Println("queryTable results:")
	for rows.Next() {
		if err = rows.Scan(&productid, &name, &author, &description, &photo); err != nil {
			return err
		}
		blobLen, _ := photo.GetLength()
		fmt.Printf("%v %v %v %v %v\n", productid, name, author, description, blobLen)
	}
	return nil
}
/* 删除产品信息表数据 */
func deleteTable() error {
	var sql = "DELETE FROM production.product WHERE productid = 12;"
	if _, err := db.Exec(sql); err != nil {
		return err
	}
	fmt.Println("deleteTable succeed")
	return nil
}
/* 关闭数据库连接 */
func disconnect() error {
	if err := db.Close(); err != nil {
		fmt.Printf("db close failed: %s.\n", err)
		return err
	}
	fmt.Println("disconnect succeed")
	return nil
}

```

**注意**

`insertTable()`接口中`inFileName`字段是图片路径，需要自行再相应目录下插入图片，并修改路径。



然后需要引用项目需要的以来增加到`go.mod`文件中，执行：

```go
go mod tidy
```

执行完，会发现`import`引入相关包中`dm`包是爆红的，需要手动加入到`go.mod replace`中

```go
replace (
	dm => ../dm
)
```

然后就可以执行`main`函数了，执行结果如下：

![image-20221021155845102](/imgs/image-20221021155845102.png)



#### 5.通过DM管理工具查看数据

上图执行`main`函数成功后，可打开`DM管理工具`（需要管理员权限），连接DM数据库，找到表查看数据是否插入成功

![image-20221021160404240](/imgs/image-20221021160404240.png)

或者通过sql语句来查看表中数据：

![image-20221021160602890](/imgs/image-20221021160602890.png)







参考文档[Go 数据库接口](https://eco.dameng.com/document/dm/zh-cn/app-dev/go-go.html)

