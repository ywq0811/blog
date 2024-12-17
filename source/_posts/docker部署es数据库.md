## docker部署es数据库

1.获取镜像

```go
docker pull elasticsearch:7.14.0
```

2.创建相关文件夹

```go
mkdir -p /mnt/d/java/share/elasticsearch/config
mkdir -p /mnt/d/java/share/elasticsearch/data
mkdir -p /mnt/d/java/share/elasticsearch/plugins
```

![image-20230109160331272](C:\Users\yanwq\AppData\Roaming\Typora\typora-user-images\image-20230109160331272.png)

3.配置文件

```go
echo "http.host: 0.0.0.0" >> /mnt/d/java/share/elasticsearch/config/elasticsearch.yml
```

配置完成，可以执行命令查看

```go
vim /mnt/d/java/share/elasticsearch/config/elasticsearch.yml
```

4.创建容器

```go
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
> -e "discovery.type=single-node" \
> -e ES_JAVA_OPTS="-Xms84m -Xmx512m" \
> -v /mnt/d/java/share/elasticsearch/data:/usr/share/elasticsearch/data \
> -v /mnt/d/java/share/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
> -v /mnt/d/java/share/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
> -d elasticsearch:7.14.0
```

![image-20230109160537007](C:\Users\yanwq\AppData\Roaming\Typora\typora-user-images\image-20230109160537007.png)

5.测试正常移动页面

![image-20230109160747537](C:\Users\yanwq\AppData\Roaming\Typora\typora-user-images\image-20230109160747537.png)



[参考文档](https://blog.csdn.net/qq_44732146/article/details/120744829?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-5-120744829-blog-124889752.pc_relevant_recovery_v2&spm=1001.2101.3001.4242.4&utm_relevant_index=8)

