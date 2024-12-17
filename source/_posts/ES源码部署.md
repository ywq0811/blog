---
title: ES源码部署
categories:
  - linux
tags:
  - es数据库
toc: true # 是否启用内容索引
---

# ES源码部署

## JAVA部署

将jdk-8u161-linux-x64.tar.gz拷入合适位置，或着自己去java官网下你想要的版本，要求是8uxxx序列的，版本需比161高

[JAVA官方下载网址](https://www.oracle.com/cn/java/technologies/javase/javase-jdk8-downloads.html)

tar -xvf解压

将解压后的内容拷到合适位置，本例中使用/usr/lib/jdk

```shell
cp -r jdk1.8.0_161/.  /usr/lib/jdk
```

写入环境变量

```shell
sudo vim /etc/profile
```

```
#set java env
export JAVA_HOME=/usr/lib/jdk
export JRE_HOME=${JAVA_HOME}/jre    
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib    
export PATH=${JAVA_HOME}/bin:$PATH
```

使环境变量生效

```shell
source /etc/profile
```

配置软连接，部分软件可能会从/usr/bin目录下查找Java

```shell
sudo update-alternatives --install /usr/bin/java  java  /usr/lib/jdk/bin/java 300
sudo update-alternatives --install /usr/bin/javac  javac  /usr/lib/jdk/bin/javac 300
```

查看java版本号，判断部署是否成功

```shell
java -version
```

## ES部署

[官方部署文档](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/getting-started-install.html)

[官方源码安装部署文档](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/zip-targz.html)

#### 下载源码包

```shell
curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.2.tar.gz
```

直接curl下速度很慢，这里建议直接自己翻墙下载后拷进服务器

将源码包放置到合适位置，本例中放置于/home

```shell
tar -xvf elasticsearch-6.6.2.tar.gz
cd elasticsearch-6.6.2/config
vim elasticsearch.yml
```

#### 修改配置文件

初始的配置文件中全部是注释状态的，一定要进行设置

```
cluster.name: my-application
 
path.data: /home/esdata/data
 
path.logs: /home/esdata/log
 
network.host: 0.0.0.0
 
http.port: 9200
 
# 是否支持跨域
http.cors.enabled: true
 
# *表示支持所有域名
http.cors.allow-origin: "*"
```

path.data和path.logs是es数据落盘的位置，可以自行调整，本例中使用上述路径

```shell
vim jvm.options
```

调整java虚拟机占用的内存大小，官方推荐设置为机器内存的一半，可以根据实际情况进行调整，比如服务压力不大就可以不用设置那么大，另外如果设置的过小的话，CPU的压力就会高起来

如果服务压力不大且机器配置允许的话，可以写1g，本例中写500m。如果服务压力较大，请根据实际使用时的CPU压力自行调整

此配置项应该原本就有，不需要加在后面，如果没看到就找找看是不是被注释状态



```
-Xms500m
-Xmx500m
```



#### 修改文件标识符大小上限

```shell
vim /etc/sysctl.conf
```

在最后加上

```
vm.max_map_count=655360
```

使配置生效

```shell
sysctl -p
```

一样是修改文件标识符

```shell
vim /etc/security/limits.conf
```

如果有的话就修改，没有的话就加上【这个应该是原本就有【默认状态下可能是注释着的，你加的这四行得是非注释状态

```
root soft nofile 655370
root hard nofile 655370
* soft nofile 655370
* hard nofile 655370
```

这个配置文件修改后就会生效，但是在当前连接中不会生效，需要重新开一个连接，如果是xshell连服务器的，需要重开一个页面这样

如果是直接在服务器上操作，可以reboot重启服务器

```shell
ulimit -n
```

查看文件标识符的配置是否生效



#### 创建elasticsearch用户

```shell
adduser elasticsearch
passwd elasticsearch
```

输入密码，自己记住

给予elasticsearch账号ES相关文件夹的权限，包括数据存储的文件夹，和elasticsearch程序所在文件夹

```shell
chown -R elasticsearch /home/esdata/
chown -R elasticsearch /home/elasticsearch-6.6.2
```



#### 切换到elasticsearch账号，启动ES

```shell
su elasticsearch
cd /home/elasticsearch-6.6.2
./bin/elasticsearch -d -p pid
```

启动后会自动在后台运行，并在当前目录下生成一个名为pid的文件，里面记录着ES进程的pid



使用此指令查看ES是否正常启动

```shell
curl localhost:9200
```

```json
{
  "name" : "-5ESl62",
  "cluster_name" : "my-application",
  "cluster_uuid" : "yxlA4dNJQPeei9KiUlVc9w",
  "version" : {
    "number" : "6.6.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "3bd3e59",
    "build_date" : "2019-03-06T15:16:26.864148Z",
    "build_snapshot" : false,
    "lucene_version" : "7.6.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

看到此信息，则表示ES服务已经正常启动



#### 关闭

可以使用此指令直接读取pid文件中的进程pid将ES关闭

```shell
pkill -F pid
```

查看ES服务的运行状态

```
ps aux | grep elasticsearch
```

```shell
root     10700  0.0  0.0  63048   460 pts/1    S    Aug11   0:00 su elasticsearch
root     11939  0.0  0.1  63048  2484 pts/4    S    Aug11   0:00 su elasticsearch
elastic+ 15632  0.0  0.0  14428   996 pts/4    S+   14:52   0:00 grep --color=auto elasticsearch
```

如果显示这些，表示ES服务已经被关掉

如果没有被关掉可以手动kill -9杀掉



#### 开机自启

使用root权限

在/etc/systemd/system目录下新建一个elasticsearch.service文件

```
cd /etc/systemd/system
vim elasticsearch.service
```

写入以下内容，其中

User 根据你创建的给ES的账号自行调整

ExecStart 根据你的ES文件目录自行调整

```
[Unit]
Description=elasticsearch
[Service]
User=elasticsearch
LimitNOFILE=100000
LimitNPROC=100000
ExecStart=/home/elasticsearch-6.6.2/bin/elasticsearch
[Install]
WantedBy=multi-user.target
```

开启开机自启动

```
systemctl enable elasticsearch
```

启动ES【现在可以用root账号启动了】

```
systemctl start elasticsearch
```



## 参考文档

[Ubuntu 16.04下Java环境安装与配置](https://www.cnblogs.com/lfri/p/10437266.html)

[ES安装部署](https://blog.csdn.net/xiaoxiongaa0/article/details/90815541)

[CentOS 7 elasticsearch service 开机自启](https://blog.csdn.net/jiankunking/article/details/82770832)



## 扩展阅读

[总结—elasticsearch启动失败的几种情况及解决](https://blog.51cto.com/10950710/2124131)