---
title: docker consul 集群部署
categories:
  - Docker
tags:
  - consul
toc: true # 是否启用内容索引
---

## docker consul 集群部署

本文档演示的在一台服务器上部署节点=3的consul集群

### 准备

1.在宿主机上分别建立目录 `server1` 、`server2` 、`server3` 文件夹，并对应创建`config` 、`data` 、`log` 文件夹

其中：

- `config` ：配置文件路径。也是docker启动时读取的配置文件路径
- `data` ：数据存储路径。docker启动时挂载到容器中的指定路径
- `log` ：日志输出路径。（可以不落盘日志，本文档演示的是落盘日志）

![image-20240226144216083](/imgs/1.png)

------

### 单点部署

##### server1

1.进入到 `config` 目录下

```shell
cd server1/config
```

2.创建 `config.json` 配置文件

```shel
vim config.json
```

![image-20240226151703284](/imgs/2.png)

```shell
{
    "datacenter": "dc1",
    "bootstrap_expect": 1,
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",	
    "node_name": "consul_server_1",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": true,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    }
}
```

参数说明：

- `datacenter`：指定consul的数据中心名称
- `bootstrap_expect`：指定启动时需要的最少节点数
- `data_dir`：指定consul在运行过程中存储数据的目录路径（容器内路径）
- `log_file`：指定consul输出的日志路径（容器内路径）
- `log_level`：指定consul日志输出的等级
- `node_name`：指定consul的节点名称
- `client_addr`：指定consul客户端访问地址
- `server`：指定是否是server节点
- `ui`：指定是否启用consul的web管理界面
- `enable_script_checks`：指定是否启用支持脚本检查（Script Checks）功能。脚本检查允许用户通过自定义的脚本来检查服务的健康状态。
- `addresses`：指定Consul agent监听的IP地址

![image-20240226172420932](/imgs/image-20240226172420932.png)

`log_file`： 应该是 "/consul/log/"

3.配置启动命令

在`server1` 目录下 创建启动脚本`run.sh`

```shell
mkdir run.sh
```

```shell
#!/bin/bash

docker run -d -p 8500:8500 --name=consul_server_1 -v $PWD/data:/consul/data -v $PWD/config:/consul/config -v $PWD/log:/consul/log -e CONSUL_BIND_INTERFACE='eth0' consul agent -config-dir=/consul/config/config.json
```

参数说明

- `-d`：在后台运行容器。
- `-p 8500:8500`：指定容器内端口和宿主机端口的映射关系，将容器内的8500端口映射到宿主机的8500端口上。

- `--name=consul_server_1` ：指定容器名称为consul_server_1。
- `-v $PWD/data:/consul/data`：挂载宿主机上的数据存储目录到容器中的`/consul/data` 目录
- `-v $PWD/config:/consul/config`：挂载宿主机上的配置文件目录到容器中的`/consul/config` 目录
- `-v $PWD/log:/consul/log`：挂载宿主机上的日志输出目录到容器中的`/consul/log` 目录

- `-e CONSUL_BIND_INTERFACE='eth0'`：通过环境变量`CONSUL_BIND_INTERFACE`指定Consul绑定的网卡接口为eth0
- `consul agent` ：启动consul agent
- `-config-dir=/consul/config/config.json`：指定配置文件路径（容器内），consul会自动读取配置文件中的参数

![image-20240226161814054](/imgs/image-20240226161814054.png)

4.启动服务

```shell
sh run.sh
```

![image-20240226172625032](/imgs/image-20240226172625032.png)

5.通过网页打开http://127.0.0.1:8500/ui看是否能看到consul看板



##### 加入ACL认证

1.使用linux的命令生成一个64位的UUID作为master token

```shell
uuidgen

output:
487d910a-a599-49b5-979e-57855e53b3b0
```

![image-20240226174005313](/imgs/image-20240226174005313.png)

2.编写acl.hcl文件

```shell
vim config/acl.hcl
```

```shell
acl {
  enabled = true
  default_policy = "deny"
  enable_token_persistence = true
  tokens {
    master = "487d910a-a599-49b5-979e-57855e53b3b0"
  }
}
```

![image-20240226174126814](/imgs/image-20240226174126814.png)

3.重启服务

```shell
docker restart consul_server_1
```

4.访问UI，提示需要输入token，输入上面配置的master token即可

![image-20240226174320743](/imgs/image-20240226174320743.png)



------

### 集群部署

#### 一台服务器部署

在一台服务器上部署consul集群，docker run命令可以使用-p 和 -e CONSUL_BIND_INTERFACE='eth0' 的方式。因为在同一台服务器上，容器网卡是eth0。

##### server1

1.进入到 `config` 目录下

```shell
cd server1/config
```

2.创建 `config.json` 配置文件

```shel
vim config.json
```

![image-20240226151703284](/imgs/2.png)

```shell
{
    "datacenter": "dc1",
    "bootstrap_expect": 3,
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",	
    "node_name": "consul_server_1",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": true,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    }
}
```

参数说明：

- `datacenter`：指定consul的数据中心名称
- `bootstrap_expect`：指定启动时需要的最少节点数
- `data_dir`：指定consul在运行过程中存储数据的目录路径（容器内路径）
- `log_file`：指定consul输出的日志路径（容器内路径）
- `log_level`：指定consul日志输出的等级
- `node_name`：指定consul的节点名称
- `client_addr`：指定consul客户端访问地址
- `server`：指定是否是server节点
- `ui`：指定是否启用consul的web管理界面
- `enable_script_checks`：指定是否启用支持脚本检查（Script Checks）功能。脚本检查允许用户通过自定义的脚本来检查服务的健康状态。
- `addresses`：指定Consul agent监听的IP地址

![image-20240226161334661](/imgs/image-20240226161334661.png)

`log_file`： 应该是 "/consul/log/"

3.配置启动命令

在`server1` 目录下 创建启动脚本`run.sh`

```shell
mkdir run.sh
```

```shell
#!/bin/bash

docker run -d -p 8500:8500 --name=consul_server_1 -v $PWD/data:/consul/data -v $PWD/config:/consul/config -v $PWD/log:/consul/log -e CONSUL_BIND_INTERFACE='eth0' consul agent -config-dir=/consul/config/config.json
```

参数说明

- `-d`：在后台运行容器。
- `-p 8500:8500`：指定容器内端口和宿主机端口的映射关系，将容器内的8500端口映射到宿主机的8500端口上。

- `--name=consul_server_1` ：指定容器名称为consul_server_1。
- `-v $PWD/data:/consul/data`：挂载宿主机上的数据存储目录到容器中的`/consul/data` 目录
- `-v $PWD/config:/consul/config`：挂载宿主机上的配置文件目录到容器中的`/consul/config` 目录
- `-v $PWD/log:/consul/log`：挂载宿主机上的日志输出目录到容器中的`/consul/log` 目录

- `-e CONSUL_BIND_INTERFACE='eth0'`：通过环境变量`CONSUL_BIND_INTERFACE`指定Consul绑定的网卡接口为eth0
- `consul agent` ：启动consul agent
- `-config-dir=/consul/config/config.json`：指定配置文件路径（容器内），consul会自动读取配置文件中的参数

![image-20240226161814054](/imgs/image-20240226161814054.png)

4.启动服务

```shell
sh run.sh
```

启动后，因为配置了bootstrap_expect=3，但只启动了一个server，所以会报错：没有集群领导者

![4](/imgs/4.png)

需要把另外2个服务也启动起来！



##### server2

1.进入到 `config` 目录下

```shell
cd server2/config
```

2.创建 `config.json` 配置文件

```shel
vim config.json
```

```shell
{
    "datacenter": "dc1",
    "bootstrap_expect": 3,
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",	
    "node_name": "consul_server_2",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": true,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    }
}
```

3.配置启动令

在`server2` 目录下 创建启动脚本`run.sh`

```shell
mkdir run.sh
```

```shell
#!/bin/bash

docker run -d -p 8510:8500 --name=consul_server_2 -v $PWD/data:/consul/data -v $PWD/config:/consul/config -v $PWD/log:/consul/log -e CONSUL_BIND_INTERFACE='eth0' consul agent -config-dir=/consul/config/config.json
```

4.启动服务

```shell
sh run.sh
```

5.加入集群

```shell
docker exec -it consul_server_3 consul join {consul_server_1.IP}
```



##### server3

1.进入到 `config` 目录下

```shell
cd server3/config
```

2.创建 `config.json` 配置文件

```shel
vim config.json
```

```shell
{
    "datacenter": "dc1",
    "bootstrap_expect": 3,
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",	
    "node_name": "consul_server_3",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": true,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    }
}
```

3.配置启动命令

在`server3` 目录下 创建启动脚本`run.sh`

```shell
mkdir run.sh
```

```shell
#!/bin/bash

docker run -d -p 8520:8500 --name=consul_server_3 -v $PWD/data:/consul/data -v $PWD/config:/consul/config -v $PWD/log:/consul/log -e CONSUL_BIND_INTERFACE='eth0' consul agent -config-dir=/consul/config/config.json
```

4.启动服务

```shell
sh run.sh
```

5.加入集群

```shell
docker exec -it consul_server_3 consul join {consul_server_1.IP}
```



##### 加入ACL认证

1.生成UUID

```shell
uuidgen

output:

```

2.分别在`consul_server_1` 、`consul_server_2` 、`consul_server_3` 的`config` 文件夹中新增`acl.hcl` 配置文件

```shell
vim config/acl.hcl
```

```shell
primary_datacenter = "dc1"
acl {
  enabled = true
  default_policy = "deny"
  enable_token_persistence = true
  tokens {
    master = "487d910a-a599-49b5-979e-57855e53b3b0"
  }
}
```

参数说明：

- `enabled = true`：代表开启ACL
- `default_policy=“deny”`：默认为allow，如果需要自定义权限，需要将其设置为deny
- `enable_token_persistence =true`： 开启token持久化，将token持久化到磁盘上

3.重启 `consul_server_1` 、 `consul_server_2` 、`consul_server_3` 服务

```shell
docker restart consul_server_1

docker restart consul_server_2

docker restart consul_server_3
```

5.启动UI界面查看，登录需要`secreatID`验证，即输入`acl.hcl` 配置文件中的`master` token







------

#### 多台服务器部署

目前查找到能部署成功的方式是通过network的方式。



##### server1

服务器IP：192.168.26.73

1.进入到 `config` 目录下

```shell
cd server1/config
```

2.创建 `config.json` 配置文件

```shel
vim config.json
```

![image-20240226151703284](/imgs/2.png

```shell
{
    "datacenter": "dc1",
    "bootstrap_expect": 3,
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",	
    "node_name": "consul_server_1",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": true,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    }
}
```

![image-20240226161334661](/imgs/image-20240226161334661.png)

`log_file`： 应该是 "/consul/log/"

3.配置启动命令

在`server1` 目录下 创建启动脚本`run.sh`

```shell
mkdir run.sh
```

```shell
#!/bin/bash

docker run -d --net=host --name=consul_server_1 -v $PWD/data:/consul/data -v $PWD/config:/consul/config -v $PWD/log:/consul/log -e CONSUL_BIND_INTERFACE='ens192' consul agent -bind=192.168.26.73 -config-dir=/consul/config/config.json
```

参数说明

- `-d`：在后台运行容器。
- `--network=host`：将容器连接到名为host的网络
- `--name=consul_server_1` ：指定容器名称为consul_server_1。
- `-v $PWD/data:/consul/data`：挂载宿主机上的数据存储目录到容器中的`/consul/data` 目录
- `-v $PWD/config:/consul/config`：挂载宿主机上的配置文件目录到容器中的`/consul/config` 目录
- `-v $PWD/log:/consul/log`：挂载宿主机上的日志输出目录到容器中的`/consul/log` 目录
- `-e CONSUL_BIND_INTERFACE='ens192'`：通过环境变量`CONSUL_BIND_INTERFACE`指定Consul绑定的网卡接口为`ens192`（这个根据实际的网卡名称，通过`ifconfig` 命令查看）
- `consul agent` ：启动consul agent
- `-bind=192.168.26.73`：指定Consul节点在Docker容器内监听的IP地址（服务器IP）
- `-config-dir=/consul/config/config.json`：指定配置文件路径（容器内），consul会自动读取配置文件中的参数

![image-20240227174226118](/imgs/image-20240227174226118.png)

4.启动服务

```shell
sh run.sh
```

启动后，因为配置了bootstrap_expect=3，但只启动了一个server，所以会报错：没有集群领导者

![4](/imgs/4.png)

需要把另外2个服务也启动起来！



##### server2

服务器IP：192.168.26.74

1.进入到 `config` 目录下

```shell
cd server2/config
```

2.创建 `config.json` 配置文件

```shel
vim config.json
```

```shell
{
    "datacenter": "dc1",
    "bootstrap_expect": 3,
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",	
    "node_name": "consul_server_2",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": true,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    }
}
```

3.配置启动命令

在`server2` 目录下 创建启动脚本`run.sh`

```shell
mkdir run.sh
```

```shell
#!/bin/bash

docker run -d --network=host --name=consul_server_2 -v $PWD/data:/consul/data -v $PWD/config:/consul/config -v $PWD/log:/consul/log -e CONSUL_BIND_INTERFACE='ens192' consul agent -bind=192.168.26.74 -join=192.168.26.73
```

参数说明

- `-join=192.168.26.73`：将节点加入到`consul_server_1` IP上

![image-20240227174405828](/imgs/image-20240227174405828.png)

4.启动服务

```shell
sh run.sh
```



##### server3

服务器IP：192.168.26.75

1.进入到 `config` 目录下

```shell
cd server3/config
```

2.创建 `config.json` 配置文件

```shel
vim config.json
```

```shell
{
    "datacenter": "dc1",
    "bootstrap_expect": 3,
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",	
    "node_name": "consul_server_3",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": true,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    }
}
```

3.配置启动命令

在`server3` 目录下 创建启动脚本`run.sh`

```shell
mkdir run.sh
```

```shell
#!/bin/bash

docker run -d --network=host --name=consul_server_3 -v $PWD/data:/consul/data -v $PWD/config:/consul/config -v $PWD/log:/consul/log -e CONSUL_BIND_INTERFACE='ens192' consul agent -bind=192.168.26.75 -join=192.168.26.73 -config-dir=/consul/config/config.json
```

参数说明

- `-join=192.168.26.73`：将节点加入到`consul_server_1` IP上

![image-20240227174802437](/imgs/image-20240227174802437.png)

4.启动服务

```shell
sh run.sh
```

![image-20240227174946704](/imgs/image-20240227174946704.png)



##### 节点自动加入集群

1.分别编辑`sverer1`、`sverer2`、`sverer3` 的配置文件，加入`start_join` 和 `retry_join` 字段

```shell
vim config/config.json
```

![image-20240229141458532](/imgs/image-20240229141458532.png)

`server2` 和 `server3` 同样配置。

2.重新加载配置文件，验证配置是否正确

```shell
docker exec consul_server_1 consul reload
```

![image-20240229143001728](/imgs/image-20240229143001728.png)

我测试的`consul_server_2` 是 `leader`，加入了配置，但是没有重启`leader`。我重启的是 `consul_server_1`，测试优雅退出 `consul_server_1`，然后再重启，发现自动加入到了节点；也测试 `consul_server_3` 不加配置，然后优雅退出 `consul_server_3` 再重启，发现 `consul_server_3` 也自动加入到了节点。（这是为什么呢，从测试的结果来看，只要一个节点加入配置即可）



##### 加入ACL认证

方法一：配置`acl.hcl `，通过consul acl bootstrap 生成token，然后把生成的token当做 `master` 的token。

本文档不采用该方法，有兴趣的可以自行去了解。[参考文档](https://blog.csdn.net/li450126014/article/details/105951195?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-5-105951195-blog-111150583.235%5Ev43%5Epc_blog_bottom_relevance_base4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-5-105951195-blog-111150583.235%5Ev43%5Epc_blog_bottom_relevance_base4&utm_relevant_index=10) 中【增加ACL token权限配置】目录。



方法二：使用linux的 `uuidgen` 命令生成一个64位UUID作为 `master` token，写入`acl.hcl` 配置文件中

1.生成UUID

```shell
uuidgen
```

output:

```shell
17b0d7f6-cd24-4989-ad84-9e1e5c938ce8
```

![image-20240229144313793](/imgs/image-20240229144313793.png)

2.分别在`consul_server_1` 、 `consul_server_2` 、`consul_server_3` 的 `config`文件夹中 新增`acl.hcl` 配置文件，并将生成的token 加入文件中

```shell
vim config/acl.hcl
```

```shell
primary_datacenter = "dc1"
acl {
  enabled = true
  default_policy = "deny"
  enable_token_persistence = true
  tokens {
    master = "17b0d7f6-cd24-4989-ad84-9e1e5c938ce8"
  }
}
```

参数说明：

- `enabled = true`：代表开启ACL
- `default_policy=“deny”`：默认为allow，如果需要自定义权限，需要将其设置为deny
- `enable_token_persistence =true`： 开启token持久化，将token持久化到磁盘上

3.重启 `consul_server_1` 、 `consul_server_2` 、`consul_server_3` 服务

```shell
docker restart consul_server_1

docker restart consul_server_2

docker restart consul_server_3
```

4.启动UI界面查看

![image-20240229161256697](/imgs/image-20240229161256697.png)

输入 `acl.hcl` 配置文件中的 `master` 的 token

![image-20240229161313672](/imgs/image-20240229161313672.png)



![image-20240229161433696](/imgs/image-20240229161433696.png)

