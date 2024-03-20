---
title: Linux执行mysql命令
categories:
  - Linux
tags:
  - mysql
toc: true # 是否启用内容索引
---

如果是dockers容器，需要先进入容器

````
docker exec -it mysql sh
````

然后登陆mysql

````
mysql -u root -p
输入密码
````

显示数据库

````
show databases;
````

打开数据库

```
use 数据库名;

例如:use kvpl;
```

显示表

````
show tables;
````

显示表中记录

````
select * from 表名;

例如：select * from node;
````

建库

````
create database 库名;
````

建表

````
create table 表名;
````

删库

````
drop database 库名;
````

删表

````
drop table 表名;
````

删除表中所有记录

````
delete from 表名;
````

