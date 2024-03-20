---
title: docker 部署mysql
categories:
  - Docker
tags:
  - mysql
toc: true # 是否启用内容索引
---

## docker 部署mysql

##### 第1步：从Docker Hub拉取官方mysql镜像

docker pull mysql:5.7

##### 第2步：使用docker images命令查看镜像

##### 第3步：启动我们的mysql的镜像，创建一个MySQL容器

使用命令：

```
docker run -d --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7
```

解释一下这里的参数：


-d表示在后台运行，不随当前命令行窗口的退出而退出


--name给容器起一个别名，以后可以通过这个别名管理此容器


-p 3307：3307把宿主机的3307端口映射到Mysql容器的3306端口


-e MySQL容器的环境配置

##### 第4步：查看我们已经启动的mysql容器

使用命令：docker ps

##### 第5步：进入MySQl容器：使用的docker exec命令，-it是参数，bash表示创建一个交互界面

使用命令：docker exec -it mysql:5.7 /bin/bash



##### 第6步：登录MySQL的服务器：使用的root用户登录的MySQL，在输入密码之后，我们可以看到已经进去了的MySQL

mysql -u root -p
输入密码