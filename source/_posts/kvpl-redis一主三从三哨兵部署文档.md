---
title: redis哨兵部署
categories:
  - Docker
tags:
  - redis
toc: true # 是否启用内容索引
---

### 各redis的ip和端口如下：

主(redis-mater)：192.168.26.75:16379

从(redis-slaveof)：192.168.26.73:6380

三哨兵：(sentinel-26379)：192.168.73:26379  

​				(sentinel-26380)：192.168.74:26380   

​				(sentinel-26381)：192.168.75:26381

### docker部署流程：

#### 下载redis镜像

```
docker pull redis:5.0.9
```

#### 创建主data文件夹

```
mkdir /home/yanwq/docker_data/redis/redis_data/data
```

#### docker运行主redis

```
docker run --name redis-master --net=host -v /home/yanwq/docker_data/redis/redis_data1/data:/data -d redis:5.0.9 redis-server --port 16379
```

#### 创建从data1文件夹

```
mkdir /home/yanwq/docker_data/redis/redis_data1/data
```

#### docker运行从redis

```
docker run --name redis-slaveof --net=host -v /home/yanwq/docker_data/redis/redis_data1/data:/data -d redis:5.0.9 redis-server --slaveof 192.168.26.75 16379 --port 6380
```

#### 查看集群是否成功

```
docker exec –it redis-master bash
redis-cli -p 16379
info replication

//范例结果
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.26.73,port=6380,state=online,offset=43329,lag=1
master_replid:62e00c7324e572561d197106981439b3741ab908
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:43613
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:43613
```

![image-20241217140514682](/imgs/image-20241217140514682.png)

#### 创建哨兵配置

##### sentinel-26379

```
mkdir /home/yanwq/docker_data/redis/sentinel/sentinel1
mkdir sentinel-26379.conf
```

##### cd sentinel-263.79.conf 配置内容如下：

```
protected-mode no
bind 0.0.0.0
port 26379
daemonize no
sentinel monitor mymaster 192.168.26.75 16379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 180000
```

![image-20241217140713740](/imgs/image-20241217140713740.png)

##### sentinel-26380

```
mkdir /home/yanwq/docker_data/redis/sentinel/sentinel2
mkdir sentinel-26380.conf
```

##### cd sentinel-263.80.conf 配置内容如下：

```
protected-mode no
bind 0.0.0.0
port 26380
daemonize no
sentinel monitor mymaster 192.168.26.75 16379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 180000
```

![image-20241217140741040](/imgs/image-20241217140741040.png)

##### sentinel-26381

```
mkdir /home/yanwq/docker_data/redis/sentinel/sentinel3
mkdir sentinel-26381.conf
```

##### cd sentinel-263.81.conf 配置内容如下：

```
protected-mode no
bind 0.0.0.0
port 26379
daemonize no
sentinel monitor mymaster 192.168.26.75 16379 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 180000
```

![image-20241217140801380](/imgs/image-20241217140801380.png)

#### docker运行哨兵

##### 哨兵sentinel-26379

```
docker run -it --name sentinel-26379 --net=host -v /home/yanwq/docker_data/redis/sentinel/sentinel1/sentinel-26379.conf:/usr/local/etc/redis/sentinel-26379.conf -d redis:5.0.9 redis-sentinel /usr/local/etc/redis/sentinel-26379.conf 
```

##### 查看日志

```
docker logs -f sentinel-26379
```

##### 哨兵sentinel-26380

```
docker run -it --name sentinel-26380 --net=host -v /home/yanwq/docker_data/redis/sentinel/sentinel2/sentinel-26380.conf:/usr/local/etc/redis/sentinel-26380.conf -d redis:5.0.9 redis-sentinel /usr/local/etc/redis/sentinel-26380.conf 
```

##### 查看日志

```
docker logs -f sentinel-26380
```

##### 哨兵sentinel-26381

```
docker run -it --name sentinel-26381 --net=host -v /home/yanwq/docker_data/redis/sentinel/sentinel3/sentinel-26381.conf:/usr/local/etc/redis/sentinel-26381.conf -d redis:5.0.9 redis-sentinel /usr/local/etc/redis/sentinel-26381.conf
```

##### 查看日志

```
docker logs -f sentinel-26381
```

#### 查看集群+哨兵是否成功

##### 进入主redis中执行

```
docker exec -it redis-master bash
redis-cli -p 16379
keys *					// (empty list or set)
set user1 user1   		//OK
get user1      			//"usre1"
```

![image-20241217140829467](/imgs/image-20241217140829467.png)

##### 进入从redis中查看

```
docker exec -it redis-slaveof bash
redis-cli -p 6380
get user1			//"user1"
```

![image-20241217140839081](/imgs/image-20241217140839081.png)

##### 停掉主redis服务,打开3个会话查看哨兵日志

```
docker stop redis-master
docker logs -f sentinel-26379
docker logs -f sentinel-26380
docker logs -f sentinel-26381
```

![image-20241217140846971](/imgs/image-20241217140846971.png)