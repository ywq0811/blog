---
title: windows安装sqlite以及使用文档
categories:
  - DB
tags:
  - db
toc: true # 是否启用内容索引
---

## windows安装sqlite以及使用文档

#### 安装

1.进入官网安装包下载路径:https://www.sqlite.org/download.html 选择windows版本下载

![image-20221122110759002](/imgs/image-20221122110759002.png)

2.选择路径加入下载的文件，我的路径D:\java\sqlite（或者C:\sqlite下）

3.将下载解压出来的5个文件，拷贝到D:\java\sqlite

![image-20221122111114287](/imgs/image-20221122111114287.png)

4.将D:\java\sqlite加入到环境变量

![image-20221122111148921](/imgs/image-20221122111148921.png)

5.打开cmd，输入'sqlite3'，将显示SQLite版本即表示安装成功

![image-20221122111256545](/imgs/image-20221122111256545.png)

提示：红色字体表示目前未指定相关数据库文件，而是以内存作为数据表等的存储位置



#### 使用

sqlite没用管理界面，只能在命令行中进行crud。（或者自行安装可视化管理界面）

以下操作都需要在环境变量路径下执行cmd，再执行`sqlite3`后执行

1.创建数据库，创建的数据库存储在sqlite3.exe所在的文件夹

```go
.open test.db    //test.db 就是创建的数据库
```

![image-20221122112012587](/imgs/image-20221122112012587.png)

效果图：

![image-20221122112038745](/imgs/image-20221122112038745.png)

2.查看数据库

```go
.databases
```

效果图

![image-20221122112212621](/imgs/image-20221122112212621.png)

（我这里是删除了test.db 数据库，重新建了一个kfa-workstation.db 数据库）





[其他参考][https://www.likecs.com/show-205144414.html]

