---
title: windows下通过nginx配置项目
categories:
  - windows
tags:
  - go
  - vue
  - mysql
  - nginx
toc: true # 是否启用内容索引
---

## windows下通过nginx配置项目

**说明：**

本文档演示windows下通过nginx配置纯vue项目，在浏览器上访问页面。

**准备：**

1. 后端：go语言打包`main.exe `服务，通过配置项连接 `mysql` 数据库
2. 前端：`vue` 项目

**操作步骤：**

1. 通过 `npm run build` 打包生成`dist` 前端静态文件 (本文档不展示)
2. 在 `windows` 下载 `nginx`

```text
https://nginx.org/en/download.html
```

![image-20241017172300423](/imgs/image-20241017172300423.png)

2. 下载后解压（路径可以根据你自己来定，我这边解压到 `D:\java`目录下，并重命名为nginx）

   ![image-20241017172526623](/imgs/image-20241017172526623.png)

3. 配置 `nginx`

   打开 Nginx 的配置文件 `nginx.conf`，通常位于 `\nginx\conf\nginx.conf`。

   在 `http` 块中添加以下配置：

   ```text
   http {
       server {
           listen       80;  # 监听80端口
           server_name  localhost;  # 服务器名称
   
           # 前端静态文件目录
           location / {
               root   C:/path/to/your/dist;  # 替换为你的dist文件夹路径
               index  index.html index.htm;
               try_files $uri $uri/ /index.html;  # 处理单页应用的刷新问题
           }
   
           # 反向代理后端服务
           location /api/ {
               proxy_pass http://127.0.0.1:8080;  # 替换为你的main.exe监听的端口
               proxy_set_header Host $host;
               proxy_set_header X-Real-IP $remote_addr;
               proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
               proxy_set_header X-Forwarded-Proto $scheme;
           }
       }
   }
   ```

   **解释：**

   - `location /`：处理前端静态文件的请求。`root` 指定了静态文件的目录，`try_files` 用于处理单页应用的刷新问题。
   - `location /api/`：处理后端 API 请求。`proxy_pass` 指定了后端服务的地址，通常是 `main.exe` 监听的地址和端口。

   根据实际的内容进行修改

   ![image-20241017173604496](/imgs/image-20241017173604496.png)

4. 启动后端服务`main.exe`

   ![image-20241017173330248](/imgs/image-20241017173330248.png)

   

5. 进到 `	nginx` 目录下启动

   ```text
   start nginx
   ```

   ![image-20241017173917101](/imgs/image-20241017173917101.png)

6. 访问管理页面

   在浏览器上输入 `127.0.0.1:80` (nginx配置里面定义的80端口)

   ![image-20241017174029898](/imgs/image-20241017174029898.png)

7. 验证配置

   - 确保 `main.exe` 正常运行，并且没有报错。
   - 检查 Nginx 的日志文件（通常在 `\nginx\logs` 目录下），查看是否有任何错误信息。

8. 关闭 `nginx`

   在 `nginx`目录下执行 `nginx -s stop`，再次访问页面无法打开。

