---
title: docker consul 集群部署验证
categories:
  - Docker
tags:
  - consul
toc: true # 是否启用内容索引
---

## docker consul 集群部署验证

主要记录 docker consul 集群部署完成之后验证的文档。



##### 验证

1.查看成员（consul_server_1服务器）

```shell
docker exec consul_server_1 consul members
```

![image-20240227180705420](/imgs/image-20240227180705420.png)

2.查看集群的选举情况

```shell
docker exec consul_server_1 consul operator raft list-peers
```

![image-20240227182112918](/imgs/image-20240227182112918.png)

可以看到，目前的`leader` 是节点 `consul_server_1`  

3.通过网页打开http://192.168.26.73:8500/ui看是否能看到consul看板



##### 验证集群选举机制

1.重启`leader（consul_server_1）`

```shell
docker restart consul_server_1
```

2.查看`consul_server_2` 和 `consul_server_3` 日志

consul_server_2:

![image-20240227182239556](/imgs/image-20240227182239556.png)

consul_server_3:

![image-20240227182306232](/imgs/image-20240227182306232.png)



3.查看集群的选举情况

```shell
docker exec consul_server_1 consul operator raft list-peers
```

![image-20240227182647339](/imgs/image-20240227182647339.png)

通过步骤2和3，可以看到节点`consul_server_2` 被选举为新的 `leader`

**注：**

有一定几率，选举的`leader` 是重启后的节点（测试出现过，猜测是从日志上看重启的时候，其余正常的节点再尝试连接，返回了 `connect:connection refused ` 报错，而且尝试连接不止一次，在尝试连接几次之后，发现连接不上，才开始选举，在选举的时候，重启完成了，可以参与选举，这里就不做深度研究，知道有几率出现这个情况即可）



##### 验证节点优雅退出

1.`consul_server_1` 优雅退出

```shell
docker exec consul_server_1 consul leave
```

![image-20240227185134358](/imgs/image-20240227185134358.png)

优雅退出之后，查看服务状态是`Exited` 状态

![image-20240227190237046](/imgs/image-20240227190237046.png)

2.查看节点状态

在其余节点服务器上执行，用的是`consul_server_3` 节点（根据实际情况查看）

```shell
docker exec consul_server_3 consul members
```

![image-20240227185202210](/imgs/image-20240227185202210.png)

3.查看`leader`

当前`leader` 节点是`consul_server_2` （根据实际情况查看）

```shell
docker exec consul_server_2 consul operator raft list-peers
```

![image-20240227185408889](/imgs/image-20240227185408889.png)

查看后台日志，没有报错，访问ui正常，说明配置文件的bootstrap_expect=3，只是在创建集群的时候期待的节点数量，如果达不到就不会初次创建集群，但节点数据量达到3后，集群初次创建成功，后面如果server通过优雅退出，不影响集群的健康情况，集群仍然会正常运行，而优雅退出的集群的状态会标志为“left”。



4.重启优雅退出的节点（consul_server_1）

```shell
docker restart consul_server_1
```

查看集群状态，发现`consul_server_1` 节点状态 还是`left`，尽管启动命令中加入了`join` 参数 （这个问题可以研究一下）

![image-20240227190000659](/imgs/image-20240227190000659.png)

手动加入节点

![image-20240227190131242](/imgs/image-20240227190131242.png)

再次查看集群状态

![image-20240227190507029](/imgs/image-20240227190507029.png)

发现`consul_server_1` 节点的状态为 `alive`



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