---
title: docker 部署grafana
categories:
  - Docker
tags:
  - grafana
toc: true # 是否启用内容索引
---

## docker 部署grafana

1.拉取镜像

```go
docker pull grafana/grafana:7.4.3
```

2.启动容器

```go
docker run -d --restart always -p 3000:3000 --name grafana grafana/grafana:7.4.3
```

3.网页访问，进入可视化界面

```go
部署服务器ip:3000

输入用户名密码 admin/admin，进入会改密码
```

