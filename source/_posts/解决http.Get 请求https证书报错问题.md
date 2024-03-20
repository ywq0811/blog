---
title: 解决http.Get()方法请求https 证书报错问题
categories:
  - Go
tags:
  - https
toc: true # 是否启用内容索引
---


## 解决http.Get()方法请求https 证书报错问题

```go
resp,err := http.Get("https://XXX")

报错：x509: certificate signed by unknown authority
```

原因：http.Get()会对传过来的数字证书进行校验，但是这个证书是由不知名CA签发的

解决方法：

1. 修改client.go代码。让client端忽略对证书的校验：

   ```go
   //通过设置tls.Config的InsecureSkipVerify为true，client将不再对服务端的证书进行校验。
   tr := &http.Transport{
           TLSClientConfig:    &tls.Config{InsecureSkipVerify: true},
       }
       client := &http.Client{Transport: tr}
   
       resp, err := client.Get("https://localhost:8081")
   ```

   这种做法可以解决问题，但是可能在生产环境下不进行校验，可能存在风险

2. 将CA证书放在项目代码下，加入到dockerfile里

   每台ubunt上都有CA证书，在目录`/etc/ssl/certs`下，都有`ca-certificates.crt`这个就是证书，将他拷贝到项目代码下，以asr-svc为例

![image-20221115172937769](/imgs/image-20221115172937769.png)

然后再dockerfile加入证书

```go
WORKDIR /etc/ssl/certs
ADD ca-certificates.crt .
```

即可，打包镜像，运行代码。



[参考文档](https://www.cnblogs.com/ficow/p/13945920.html)

