---
title: nginx在windows实现开机自启
categories:
  - Windows
tags:
  - nginx
  - WinSW
toc: true # 是否启用内容索引
---


## nginx在windows实现开机自启

1.Windows Service Wrapper工具下载

下载地址：**https://github.com/winsw/winsw/releases**

![image-20241018103608753](/imgs/image-20241018103608753.png)

2.将 `WinSw-x64.exe` 放到 `nginx` 目录下，重命名为 `nginx-service.exe`

![image-20241018103807682](/imgs/image-20241018103807682.png)

3.在 `nginx` 目录下新建服务日志文件夹 `server-logs` ,用于存放 `nginx` 服务相关日志

![image-20241018104055834](/imgs/image-20241018104055834.png)

4.在 `nginx` 目录下新建配置文件 `nginx-service.xml`,写入编辑信息，配置好后就可以将Nginx注册为Windows服务

![image-20241018104258153](/imgs/image-20241018104258153.png)

配置信息如下：

```reStructuredText
<!-- nginx-service.xml -->
<service>
    <id>nginx</id>
    <name>Nginx Service</name>
    <description>Nginx服务</description>
    <logpath>D:\java\nginx\server-logs</logpath>
    <log mode="roll-by-size">
		<sizeThreshold>10240</sizeThreshold>     
		<keepFiles>8</keepFiles>  
	</log>
    <executable>D:\java\nginx\nginx.exe</executable>
    <stopexecutable>D:\java\nginx\nginx.exe -s stop</stopexecutable>
</service>
```

![image-20241018104728560](/imgs/image-20241018104728560.png)

5.在 `nginx` 目录下以管理员运行命令 `nginx-service.exe install` 完成注册

![image-20241018104836143](/imgs/image-20241018104836143.png)

6.启动服务 `nginx-service.exe start`,完成开机自启

![image-20241018105053202](/imgs/image-20241018105053202.png)

7.查看 `服务` ,检查是否自启成功

![image-20241018105126293](/imgs/image-20241018105126293.png)

----

**WinSW命令说明**

**install**：注册服务

**uninstall**：卸载服务

**start**：启动服务，启动服务之前，该服务必须已经安装

**stop**：停止服务

**stopwait**：停止服务，直到服务退出，此命令才返回

**restart**：重启服务

**status**：查看服务状态

---

windows 自行开发的项目开机自启 参考地址：https://www.jianshu.com/p/759bc5ae01d1

