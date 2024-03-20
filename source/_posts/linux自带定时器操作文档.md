---
title: linux自带定时器操作文档
categories:
  - Linux
tags:
  - cron
toc: true # 是否启用内容索引
---

## linux自带定时器操作文档

计划任务是需要在指定时间执行的任务或者是周期性执行的任务，比如凌晨3点重启设备，每周对日志文件备份等。Linux系统会内置at和cron服务，at服务用来在指定时间执行任务，cron用来周期性执行任务。

#### 一.at一次性任务

[参考连接](https://blog.csdn.net/weixin_40228200/article/details/120711676)

#### 二.cron周期性任务

1、cron周期性任务依赖于系统后台的crond进程，类似于at，我们也要首先确认cron服务是否开启，执行命令：

```go
systemctl status crond
```



2、启动crontab服务

一般启动服务用 /sbin/service crond start 若是根用户的cron服务可以用 sudo service crond start， 这里还是要注意 下 不同版本linux系统启动的服务的命令也不同 ，像我的虚拟机里只需用 sudo service cron restart 即可，若是在根用下直接键入service cron start就能启动服务



3、查看服务是否已经运行用 ps -ax | grep cron

cron.daily是每天执行一次的job

cron.weekly是每个星期执行一次的job

cron.monthly是每月执行一次的job

cron.hourly是每个小时执行一次的job

cron.d是系统自动定期需要做的任务

crontab是设定定时任务执行文件

cron.deny文件就是用于控制不让哪些用户使用Crontab的功能

4.用户配置文件：

每个用户都有自己的cron配置文件,通过crontab -e 就可以编辑,一般情况下我们编辑好用户的cron配置文件保存退出后,系统会自动就存放于/var/spool/cron/目录中,文件以用户名命名.linux的cron服务是每隔一分钟去读取一次/var/spool/cron,/etc/crontab,/etc/cron.d下面所有的内容.



5.crontab文件格式：

  *           *          *        *          *             command

minute   hour    day   month   week      command

分          时         天      月        星期       命令



minute： 表示分钟，可以是从0到59之间的任何整数。

hour：表示小时，可以是从0到23之间的任何整数。

day：表示日期，可以是从1到31之间的任何整数。

month：表示月份，可以是从1到12之间的任何整数。

week：表示星期几，可以是从0到7之间的任何整数，这里的0或7代表星期日。

command：要执行的命令，可以是系统命令，也可以是自己编写的脚本文件。


6、crontab命令

cron服务提供crontab命令来设定cron服务的，以下是这个命令的一些参数与说明:

crontab -u //设定某个用户的cron服务，一般root用户在执行这个命令的时候需要此参数

crontab -l //列出某个用户cron服务的详细内容

crontab -r //删除没个用户的cron服务

crontab -e //编辑某个用户的cron服务

比如说root查看自己的cron设置:crontab -u root -l

再例如，root想删除fred的cron设置:crontab -u fred -r



#### 示例

当前是root用户



1.创建一个root用户底下的定时任务

```go
crontab -u root -e
```

如下是编辑器内容

```go
0 2 * */6 * /home/yanwq/kvp-cmsb/cleanlog

注释：每隔6个月的2点执行cleanlog脚本
```

操作成功图:

![8B8FFDA8-1890-4eac-A6CF-787B68E1D4D1](/imgs/8B8FFDA8-1890-4eac-A6CF-787B68E1D4D1.png)



2.查看root用户底下的定时任务

```go
crontab -u root -l
```

操作成功图:

![F4C413D6-17FC-470e-A5AC-590E74F27694](/imgs/F4C413D6-17FC-470e-A5AC-590E74F27694.png)



如果刚创建完定时任务，执行`crontab -u root -l`没有结果，需要重新加载cron服务，每个linux系统启动的服务命令是不一样

```go
/sbin/service crond reload
```

