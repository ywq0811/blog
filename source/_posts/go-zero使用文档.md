---
title: go-zero的基本使用
categories:
  - Go
tags:
  - go-zero
toc: true # 是否启用内容索引
---

# go-zero的基本使用

> 安装就不赘述了，照着[官方文档](https://go-zero.dev/cn/prepare.html)即可，装好Goctl和protoc

### 1. 创建API工程

> goctl 工具创建工程分为两种，一种是api工程，一种是rpc工程，如下：
>
> api：goctl api go -api user.api -dir .
>
> rpc：goctl rpc proto -src user.proto -dir .
>
> 其中，rpc工程的创建依赖.proto文件，而api工程的创建依赖.api文件

- 创建目录：

```shell
mkdir go-zero-test
```

- 创建user接口工程的目录

```shell
mkdir -p user/api && cd user/api
```

- 用goland打开这个文件，创建`go.mod`

- 添加api文件

```shell
vim user.api

type (
	HelloReq {
		Name string `form:"name"`
	}
	HelloRes {
		Code int    `json:"code"`
		Msg  string `json:"msg"`
	}
)

service user-api {
	@handler hello
	get /user/hello (HelloReq) returns (HelloRes)
}
```

- 创建api工程

```shell
goctl api go -api user.api -dir .
```

可以看到生成了很多文件，[文件结构看这里](https://go-zero.dev/cn/api-dir.html)

### 2. 编写业务逻辑

- user/api/internal/logic/hellologic.go

```go
func (l *HelloLogic) Hello(req *types.HelloReq) (*types.HelloRes, error) {
	msg := fmt.Sprintf("Hello %s", req.Name)
	return &types.HelloRes{Code: 0, Msg: msg}, nil
}
```

- 进入`api`

````
cd api
````

- 启动

```shell
go run user.go
```

- 访问 http://localhost:8888/user/hello?name=hh

### 3. model层使用

> 可以归纳为以下步骤：
>
> 1. 执行命令生成model文件 goctl model  xxx
> 2. config.go和yaml添加数据库和缓存的配置项
> 3. 上下文中注入依赖 servicecontext.go
> 4. 使用，从上下文中取出来用，l.svcCtx.XxxModel.Xxx()
>
> 另外，model层可以简便的切换为Gorm，但是api层就很难切换为Gin了。

#### 3.1 model生成

- 在数据库中创建一张user表
- 在user目录下创建model目录
- 执行命令生成

```shell
cd user/model
goctl model mysql datasource -url="root:123456@tcp(127.0.0.1:3306)/go-zero-test" -table="user" -c -dir .
# 也可以用sql文件
# goctl model mysql ddl -src user.sql -dir . -c
```

#### 3.2 配置

- 添加mysql配置项 `vim api/internal/config/config.go`

```shell
package config

import "github.com/tal-tech/go-zero/rest"

type Config struct {
    rest.RestConf
    
    Mysql struct{
        DataSource string
    }

    CacheRedis cache.CacheConf
}
```

- 配置文件添加 ` vim api/etc/user-api.yaml`

```shell
Mysql:
  DataSource: $user:$password@tcp($url)/$db?charset=utf8mb4&parseTime=true&loc=Asia%2FShanghai
CacheRedis:
  - Host: $host
    Pass: $pass
    Type: node
```

#### 3.3 上下文注入model

`api/internal/svc/servicecontext.go`

```go
type ServiceContext struct {
    Config    config.Config
    UserModel model.UserModel
}

func NewServiceContext(c config.Config) *ServiceContext {
    conn:=sqlx.NewMysql(c.Mysql.DataSource)
    return &ServiceContext{
        Config: c,
        UserModel: model.NewUserModel(conn,c.CacheRedis),
    }
}
```

#### 3.4 修改接口文件

- user.api，添加登录接口

```go
syntax = "v1"

type (
	HelloReq {
		Name string `form:"name"`
	}
	HelloRes {
		Code int    `json:"code"`
		Msg  string `json:"msg"`
	}

	LoginReq {
		Username string `json:"username"`
		Password string `json:"password"`
	}
	LoginReply {
		Id           int64  `json:"id"`
		Name         string `json:"name"`
		Gender       string `json:"gender"`
		AccessToken  string `json:"accessToken"`
		AccessExpire int64  `json:"accessExpire"`
		RefreshAfter int64  `json:"refreshAfter"`
	}
)

service user-api {
	@handler login
	post /api/user/login (LoginReq) returns (LoginReply)
	
	@handler hello
	get /api/user/hello (HelloReq) returns (HelloRes)
}
```

- 再次执行goctl更新api

```shell
goctl api go -api user.api -dir .
```

#### 3.5 逻辑层使用model

- `api/internal/logic/loginlogic.go`

```go
func (l *LoginLogic) Login(req types.LoginReq) (*types.LoginReply, error) {
	if len(strings.TrimSpace(req.Username)) == 0 || len(strings.TrimSpace(req.Password)) == 0 {
		return nil, errors.New("参数错误")
	}

	userInfo, err := l.svcCtx.UserModel.FindOneByName(req.Username)
	switch err {
	case nil:
	case model.ErrNotFound:
		return nil, errors.New("用户名不存在")
	default:
		return nil, err
	}

	if userInfo.Password != req.Password {
		return nil, errors.New("用户密码不正确")
	}

	return &types.LoginReply{
		Id:          userInfo.Id,
		Name:        userInfo.Username,
		Gender:      strconv.FormatInt(userInfo.Gender, 10),
	}, nil
}
```

- model自动生成的只有简单的增删改查，没有根据字段名的查询，这里可以自己实现`FindOneByName`函数

```go
func (m *defaultUserModel) FindOneByName(username string) (*User, error) {
	var resp User
	query := "select * from user where username = ? limit 1;"
	err := m.QueryRowNoCache(&resp, query, username)
	switch err {
	case nil:
		return &resp, nil
	case sqlc.ErrNotFound:
		return nil, ErrNotFound
	default:
		return nil, err
	}
}
```

### 4. jwt的使用

>  步骤归纳：
>
> 1. config.go和yaml添加Auth配置项
> 2. 编写token生成函数
> 3. 需要鉴权的接口在api文件中添加jwt:Auth声明
> 4. 重新执行命令生成api代码

### 5. 中间件

> 步骤归纳：
>
> 1. api文件中添加middleware声明
> 2. 重新执行命令生成api代码
> 3. 上下文中注入依赖
> 4. 编写中间件的Handle 处理逻辑

### 6. rpc服务

#### 6.1 创建服务

- 编写proto文件

```protobuf
vim user/rpc/user.proto

syntax = "proto3";

package user;

option go_package = "user";

message IdReq{
  int64 id = 1;
}

message UserInfoReply{
  int64 id = 1;
  string name = 2;
  string gender = 3;
}

service user {
  rpc getUser(IdReq) returns(UserInfoReply);
}
```

- 生成rpc代码

```shell
cd service/user/rpc
goctl rpc proto -src user.proto -dir .
```

- 如果使用了数据库和缓存记得修改config.go和yaml

- yaml添加etcd配置

```yaml
Etcd:
  Hosts:
  - 127.0.0.1:2379
    Key: user.rpc
```

- 如果调用了model层，记得在上下文中注入依赖（svc下servicecontext.go）

- 编写逻辑层

```go
func (l *GetUserLogic) GetUser(in *user.IdReq) (*user.UserInfoReply, error) {
    one, err := l.svcCtx.UserModel.FindOne(in.Id)
    if err != nil {
        return nil, err
    }

    return &user.UserInfoReply{
        Id:     one.Id,
        Name:   one.Name,
        Number: one.Number,
        Gender: one.Gender,
    }, nil
}
```

- 调用rpc

#### 6.2 调用服务

> 可以归纳为以下步骤：
>
> 1. config.go和yaml添加rpc服务端的配置项
> 2. 上下文中注入依赖
> 3. 逻辑层可以从上下文中取出来调用

- 在调用端的config.go中加入rpc服务端的配置项

```go
 type Config struct {
      rest.RestConf
      Auth struct {
          AccessSecret string
          AccessExpire int64
      }
      UserRpc zrpc.RpcClientConf
  }
```

- yaml中添加rpc服务端的配置

```yaml
UserRpc:
    Etcd:
      Hosts:
        - 127.0.0.1:2379
      Key: user.rpc
```

- 上下文注入依赖(servicecontext.go)

```go
type ServiceContext struct {
	Config    config.Config
	...
	UserRpc   userclient.User
}

func NewServiceContext(c config.Config) *ServiceContext {
	return &ServiceContext{
		Config:    c,
		...
		UserRpc:   userclient.NewUser(zrpc.MustNewClient(c.UserRpc)),
	}
}
```

- 逻辑层调用

```go
func (l *PingLogic) Ping() error {
	fmt.Println("ping...")
    
	user, err := l.svcCtx.UserRpc.GetUser(l.ctx, &userclient.IdReq{Id: 3})
	fmt.Println(user, err)
    
    fmt.Println("api调用rpc咯")
	return nil
}
```

---

### 7. `// 自适应降载保护`

> 讲白了就是根据CPU压力来保护服务，压力过高时拒绝新的请求，直到当前积攒的请求处理完、CPU压力降下来后再次开放
>
> 但是目前我还测不出来，不知道是不是因为在windows上的原因，读取不到cpu使用率，从stat日志打出来的内容可以看到每次读取的cpu使用率都是0

- 在rest和zrpc框架里有可选激活配置
- CpuThreshold，0-1000，默认值900，如果把值设置为大于0的值，则激活该服务的自动降载机制
- 如果请求被drop，那么错误日志里会有`dropped`关键字
- [官方文档](https://go-zero.dev/cn/loadshedding.html)

---

### 8. 熔断

- 在gprc调用中已经内置了，无需额外编码
- [熔断的算法看官方文档](https://go-zero.dev/cn/breaker-algorithms.html)
- 测试起来比较简单
  - 一个api接口、一个rpc接口，api接口调用rpc
  - rpc接口内延时1秒，返回错误
  - 连续请求api接口，会发现刚开始会等待1秒后才接收到响应，到后面已经是瞬间失败了，说明api接口没有再去调用rpc

---

### 9. 并发限制

- RestConf中有有一个MaxConns配置用来限制并发数量
- 当程序中未处理完毕、未返回响应的请求数量超过此配置，后续请求将被直接拒绝

---

### 10. 引擎并发控制方案 - PeriodLimit

> 通过对zo-zero框架的了解，认为可以将其限流工具-PeriodLimit 作为引擎的并发控制方案之一

- 举个栗子，在3秒钟内，允许他10个并发

```go
var (
		seconds = 3
		quota   = 10
	)
l := limit.NewPeriodLimit(seconds, quota, redis.NewRedis("127.0.0.1:6379", redis.NodeType), "periodlimit")
```

- do函数假设是调用引擎的处理函数，而这个函数执行完毕需要耗时3秒

```go
func do(l *limit.PeriodLimit, userID string, i int) {
    // 通过 l.Take 传入的 key 来区分用户
	code, err := l.Take(userID)
	if err != nil {
		logx.Error(err)
	}

    // 耗时3秒
	time.Sleep(time.Second*3)

	switch code {
	case limit.OverQuota:  // 超出限制
		logx.Errorf("OverQuota key: %v", i)
	case limit.Allowed:    // 在限制范围内
		logx.Infof("AllowedQuota key: %v", i)
	case limit.HitQuota:   // 达到限制的临界点
		logx.Errorf("HitQuota key: %v", i)
	default:
		logx.Errorf("DefaultQuota key: %v", i)
	}
}
```

- 调用

```go
for i := 0; i < 100; i++ {
	go do(l, "user1", i)
}
```

- 优点：

  - 使用简单、开箱即用；
  - 基于 `redis` 计数器，通过调用 `redis lua script`，保证计数过程的原子性，同时也支持分布式情况下正常计数；

- 缺点：

  - 初始化时传入的时间限制与实际引擎调用的时间需要尽可能接近，否则会有误差；

  - 要记录时间窗口内的所有行为记录，如果这个量特别大的时候，内存消耗会变得非常严重；

---

