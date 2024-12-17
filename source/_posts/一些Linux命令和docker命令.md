---
title: 一些linux命令和docker命令
categories:
  - Docker 
tags:
  - docker
  - linux
toc: true # 是否启用内容索引
---


指令 | 效果
--|--
lsof -i:端口号 | 查看指定端口进程
kill 进程号 | 消灭进程
nohup 【运行指令，如npm run serve】 & | 在后台运行
tail -f nohup.out | 查看后台运行的日志【实时
tail -100 nohup.out | 查看后台运行的日志【最后100条
unzip 文件名 | 解压zip文件到当前目录
mv 被修改文件名 修改后文件名 | 修改文件/文件夹名
rz | 从本地上传文件到Xshall中的linux当前文件夹
sz 文件名 | 在Xshall中从linux下载文件到本地
yum install -y lrzsz | 安装上传下载工具
whereis 已安装的包 | 查找包安装的路径
go test -v -run="TestGenerateAdminToken" | 乌班图进入项目下jwt文件夹执行test文件
chmod 777 文件名 | 将文件权限提到777
ln -sf 文件1 文件2 | 对文件2建立软连接导向到文件1
ctrl+R | 搜索历史命令行--选好后回车直接执行
source /etc/profile | 更新当前用户下的环境变量，非root权限切到root权限后执行
apt-get update | 升级系统中的包
apt-get upgrade | 升级系统中的包
ifconfig | 查看网络相关配置
cat /proc/cpuinfo | 查看机器cpu详情
&& | 先执行&&前的指令，如果执行成功，则执行&&后的指令
scp 文件名 目标服务器账号@ip地址:保存的文件夹路径 | 发送文件到目标服务器的指定路径
`ps -aux | grep 服务名` | 查看某服务有没有在跑
du -h --max-depth=1 /home | 查看home目录下磁盘空间使用情况 
修改centOS6系统时区
```shall
cp /etc/localtime /root/old.timezone
rm /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Chongqing /etc/localtime
```
修改docker中的mysql时区
先修正系统时区，然后进入docker bash
```shall
/etc/init.d/mysql stop
```
关闭数据库，守护进程会自动拉起mysql，mysql关闭后会将使用者弹出docker bash，没有关系，重新进去就好了
***
### nginx指令
指令 | 效果
-- | --
nginx | 执行/跑
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf | 用指定的配置文件启动nginx
ps -ef\|grep nginx | 查看nginx是否启动
kill nginx:master进程号 | 停止nginx
kill -hup nginx:master进程号 | 重启nginx
nginx -s reload | 重启nginx
nginx -t | 测试nginx的配置文件可用性
***
### 分屏幕使用相关指令
指令 | 效果
--|--
sudo screen -S 屏幕名 | 建立一个指定名称的屏幕并进入
sudo screen -list | 查看当前窗口列表【screen列表】【会显示窗口对应的进程号】
sudo screen -r -D 进程号 | 进入指定进程的窗口
【在某个窗口中】ctrl+a+d | 回到主窗口
***
### git操作相关指令
指令 | 效果
--|--
git clone + gilab上的http路径 | 克隆master到此处
git pull origin develop | 在文件夹中拉取develop的代码
supervisorctl reload | 重启supervisor的守护进程【会关闭supervisor当前正在跑的程序】
supervisorctl restart 项目名 | supervisor重启指定项目
***
### windows指令
指令 | 效果
-- | --
powercfg -h off | 关闭计算机休眠并删除休眠文件，释放C盘空间【管理员身份打开cmd】

#### ubuntu命令

查看系统版本:sudo lsb_release -a

linux内核版本:cat /proc/version

##### 或者

 kernel 版本和操作系统架构，运行命令：uname -a



## Docker命令

构建`projectName`镜像：

```bash
docker build . -t projectName:0.0.1  //:0.0.1 是版本号
或者
docker build -t projectName:0.0.1 .
```

执行下面的命令来运行镜像：

```bash
docker run -p 8888:8888 goweb_app
```

列出docker本地镜像

```bash
docker images
```

删除本地镜像

```bash
docker rmi projectName:0.0.1
```

通过镜像的id来删除指定镜像

```
docker rmi <image id>
```

查看我们已经启动的容器

```
docker ps
```

查看我们已经启动的容器(包括已经退出的)

```
docker ps -a
```

停止已经启动的容器

```
docker stop CONTAINER ID
```

进入mysql容器命令行

```
docker exec -it mysql /bin/bash
```

执行结果如下：

```
root@428753b73869:/# 
```



**保存Docker镜像为文件**

如果要讲镜像保存为本地文件，可以使用Docker save命令。

命令格式：

```
docker save -o 保存后的文件名   要保存的镜像

如：Docker save实例

docker save -o java8.tar lwieske/java-8
```

从文件载入镜像可以使用Docker load命令。

命令格式：

```
docker load --input 文件
```

或者

```
docker load < 文件名
```

如：Docker load实例

```
docker load < java8.tar
```

查看端口是否被占用

```
lsof -i:端口号      如：lsof -i:3306
```

kill占用端口

```
kill PID
```

查看ubuntu版本

```
lsb_release -a
```





### 用kvpl项目作为例子来打包镜像

##### 1.载入基础镜像（kvp-centos-ffmpeg-minial.img）

```
docker load -i kvp-centos-ffmpeg-minial.img
```

成功后可以用 `docker iamge`查看镜像

##### 2.进入到kvpl目录下，用dockerfile构建`kvpl`镜像

```
1. cd $GOPATH/src/navi/kvpl

2. docker build -t kvp:0.0.1 .   // 最后的 . 就是在指定路径下找Dockerfile文件构建
```

成功后可以用 `docker iamge`查看镜像

##### 3.保存镜像

```
docker save -o kvpl.tar kvpl         //kvpl.tar是保存后的文件名,kvpl是镜像名
```

保存后可以在当前路径下输入命令`ll`，查看保存的镜像 `kvpl.tar`文件





在windows环境下打包二进制文件,然后到ubuntu里打包成docker镜像

````
GOOS=linux GOARCH=amd64 go build -o order-rpc order.go
````



