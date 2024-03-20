---
title: docker部署loki+promtail
categories:
  - Docker
tags:
  - grafana
toc: true # 是否启用内容索引
---

## docker部署loki+promtail

#### loki

1.拉取镜像

```go
docker pull grafana/loki:2.1.0
```

注：本文档适用2.1.0，不适合2.7.3版本，启动服务的时候会报wal错误

2.新建配置文件

```shell
mkdir -p /home/loki     #创建loki文件夹
mkdir -p /home/loki/index  #创建index文件夹 
mkdir -p /home/loki/chunks #创建chunks文件夹
chmod -R 777 /home/loki/index  #提权
chmod -R 777 /home/loki/chunks  #提权
cd /home/loki
touch loki-config.yaml //创建loki-config配置文件

```

3.打开配置文件

```shell
vim loki-config.yaml

-----
auth_enabled: false   #是否启用身份验证，如果设置true。需要提供有效的用户名和密码才能访问loki

server:
  http_listen_port: 3100  #定义loki实例监听的地址和端口
  grpc_listen_port: 3110  #定义loki grpc监听的地址和端口
  grpc_server_max_recv_msg_size: 1073741824  #grpc最大接收消息值，默认4m
  grpc_server_max_send_msg_size: 1073741824  #grpc最大发送消息值，默认4m

ingester:
  lifecycler:
    address: 127.0.0.1  
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 5m
  chunk_retain_period: 30s
  max_transfer_retries: 0
  max_chunk_age: 20m  #一个timeseries块在内存中的最大持续时间。如果timeseries运行的时间超过此时间，则当前块将刷新到存储并创建一个新块

schema_config:
  configs:
    - from: 2021-01-01
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 168h

storage_config:
  boltdb:
    directory: /home/loki/index #存储索引地址
  filesystem:
    directory: /home/loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h
  ingestion_rate_mb: 30  #修改每用户摄入速率限制，即每秒样本量，默认值为4M
  ingestion_burst_size_mb: 15  #修改每用户摄入速率限制，即每秒样本量，默认值为6M

chunk_store_config:
        #max_look_back_period: 168h   #回看日志行的最大时间，只适用于即时日志
        max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false #日志保留周期开关，默认为false
  retention_period: 0s  #日志保留周期

```

4.启动容器

```shell
docker run -d --restart always -p 3100:3100  --privileged=true --name loki -v /home/loki:/mnt/config -v /home/loki/index:/opt/loki/index -v /home/loki/chunks:/opt/loki/chunks grafana/loki:2.0.1 -config.file=/mnt/config/loki-config.yaml

复用修改宿主机挂载的路径即可
```

5.验证

```shell
curl http://127.0.0.1:3100/api/prom/label
curl localhost:3100/loki/api/v1/labels

注：第一个curl如果返回{}或者空，则失败，查看配置文件或者查看日志排查原因
```

![image-20230707155443566](/imgs/image-20230707155443566.png)

#### promtail

```go
promtail 是日志收集器，用于将日志发送到loki进行存储和分析。promtail 可以以代理的方式运行在应用程序所在的主机上，通过监控日志文件或者通过日志文件的标准输出来收集日志信息。

Promtail将收集到的日志数据结构化为Loki所需的格式，并将其发送到Loki实例中。Loki则负责对日志进行存储，并提供查询和浏览日志的功能。
```

根据上面的定义，promtail一般部署在应用程序所在服务器上。此文档拿tts-svc-dev应用程序来当例子，收集tts-svc-dev的日志发送给loki。所以promtail部署在tts-svc-dev服务器上（192.168.26.74:3200）



1.拉取镜像

```shell
docker pull grafana/promtail:2.1.0
```

2.创建文件夹

```shell
mkdir -p /home/kst/aihc/promtail
cd /home/kst/aihc/promtail
touch promtail-config.yaml
```

3.打开配置文件

```shell
vim promtail-config.yaml

------
server:
  http_listen_port: 9080 #云服务器需开放9080端口
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

#把loki当客户端连接
clients:
  - url: http://192.168.2.50:3100/loki/api/v1/push #这里修改实际loki的ip：port

scrape_configs:
  - job_name: tts-svc-dev  #标签，用于查询
    #pipeline_stages:
    static_configs:
    - targets:
       - localhost
      labels:
       #标签，用于后面的查询 
       job: tts-svc-dev
       __path__: /var/log/tts-svc-dev/*.log #注意，这里的路径是映射主机的/data/aihc/tts-svc/dev/log目录的容器里的目录，指的是容器里面映射的路径
```

4.启动容器

```shell
docker run -d --name promtail --privileged=true -p 9080:9080 -m 1024m -v /home/kst/aihc/promtail:/mnt/config -v /data/aihc/tts-svc/dev/log:/var/log/tts-svc-dev/ -v /etc/localtime:/etc/localtime:ro grafana/promtail:2.1.0 -config.file=/mnt/config/promtail-config.yaml

注： 这里需要把宿主机tts-svc日志存储路径挂载进容器里面， 对应配置文件的_path_参数
```

5.验证

```shell
curl "http://192.168.2.50:3100/api/prom/label"  #实际loki的ip:port
```

![image-20230707154011354](/imgs/image-20230707154011354.png)

6.接下来就可以访问grafana查看了



#### grafana界面配置

1.打开grafana访问路径,此文档部署grafana路径是192.168.2.50:3100

```shell
网页打开 http://192.168.2.50:3100
初始化账户和密码为:admin admin
登录之后会跳出修改密码界面，点击【spik】可以跳过
```

2.添加loki

![image-20230707155839036](/imgs/image-20230707155839036.png)

![image-20230707160032533](/imgs/image-20230707160032533.png)

![image-20230707160046360](/imgs/image-20230707160046360.png)

![image-20230707160144473](/imgs/image-20230707160144473.png)

然后拉到下面，点击【Save & Test】，即添加成功

3.使用loki

![image-20230707160249769](/imgs/image-20230707160249769.png)

![image-20230707160316228](/imgs/image-20230707160316228.png)

可以看到有我们刚才配置的Job_name，点击【tts-svc-dev】

![image-20230707160353065](/imgs/image-20230707160353065.png)

成功查看日志

4.操作日志

可以点击【Add query】，输入需要查询的，查出指定日志。查询方式可百度