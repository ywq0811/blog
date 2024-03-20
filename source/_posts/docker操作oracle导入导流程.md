---
title: docker 操作oracle数据库文档
categories:
  - Docker
tags:
  - oracle
toc: true # 是否启用内容索引
---

## docker 操作oracle数据库文档

说明：本文档主要是记录docker部署oracle，操作数据库文档，docker 部署oracle可以参考[docker部署oracle](https://blog.csdn.net/junle_carpediem/article/details/123430144)



docker部署完oracle数据库后

1.通过指令来查看oarcle进程名

```go
docker ps
```

2.找到oracle进程服务，通过指令进入oracle容器

```go
docker exec -it 进程名 bash
```

3.切换root用户，输入密码

```go
su root

再跳出Password：时，输入密码：helowin
```

4.切换oracle用户，无需输入密码 ,

```go
su - oracle
```

或者3.不执行， 直接执行4，输入密码进入

```go
su - oracle
Password: oracle
```



5.登录sqlplus软连接

```go
sqlplus /nolog
```

6.显示当前用户

```go
show user;

结果为：
USER is ""  //说明当前没有登录用户
```

7.切换用户 【如果没有创建用户，可以参考下面三.创建用户流程,以用户 yanwq为例。   部署oracle完，oracle内部有2个建好的用户：system和sys, 用户可直接登录到system用户以创建其他用户，因为system具有创建别 的用户的 权限。 在安装oracle时，用户或系统管理员首先可以为自己建立一个用户。】

```go
conn yanwq/123456

结果为：
Connected. //用户连接成功
```

8.输入sql语句，查看表数据，这边以voice表为例

```go
select * from voice;
```

9.directories是oracle类似虚拟目录的一个概念，它对应服务器上的一个具体路径。可以使用命令查看现有的directory也可以直接创建新的，建议创建新的，比较简单

```go
/查询现有的directory
select * from dba_directories;


/创建一个新的directories
create or replace directory yanwq_bak as '/home/oracle/app'


结果为：
Directory created.

说明：创建一个directories =yanwq_bak的在/home/oracle/app下 ，可以修改路径和yanwq_bak
```

9.退出

```go
exit
```

10.执行命令导出数据和日志

```go
expdp yanwq/123456 directory=yanwq_bak dumpfile=yanwq.dmp logfile=yanwq.log

说明:使用yanwq用户连接数据库，并导出yanwq用户的数据，这样就会将dmp和log文件存放在服务器上的/home/oracle/app目录下
```

11.执行一下命令进入app目录下，验证是否有生成yanwq.dmp和yanwq.log文件

```go
cd /home/oracle/app
```

12.导入数据,  导入之前建议先将表删除，因为oracle数据是归属于用户的，所以我们一般是先将用户删除掉之后，重建，再执行impdp命令进行导入数据。  

【进过测试只清理掉表数据，无法导入，会提示tableName exists.会跳过导入这个表的操作，删除表是可以导入的】

```go
impdp yanwq/123456 directory=yanwq_bak dumpfile=yanwq.dmp logfile=yanwq.log
```



#### 其他：

一、创建用户

oracle内部有两个建好的用户：system和sys。用户可直接登录到system用户以创建其他用户，因为system具有创建别 的用户的 权限。 在安装oracle时，用户或系统管理员首先可以为自己建立一个用户。

语法[创建用户]： create user 用户名 identified by 口令[即密码]；

例子： create user test identified by test;

语法[更改用户]: alter user 用户名 identified by 口令[改变的口令];

例子： alter user test identified by 123456;



二、删除用户

语法：drop user 用户名;

例子：drop user test;

若用户拥有对象，则不能直接删除，否则将返回一个错误值。指定关键字cascade,可删除用户所有的对象，然后再删除用户。

语法： drop user 用户名 cascade;

例子： drop user test cascade;



三.创建用户流程,以用户 yanwq为例

进入容器之后，进入Oracle：sqlplus /nolog
使用sysdba登录oracle：conn sys/oracle as sysdba
创建用户：create user yanwq identified by 123456
赋予用户权限：grant dba to yanwq 
登录：grant create session to yanwq 
conn yanwq /123456





四.删除表数据

TRUNCATE TABLE voice;

