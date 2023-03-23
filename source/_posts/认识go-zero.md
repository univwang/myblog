---
title: 认识go-zero
date: 2023-03-19 14:42:18
tags: [Go]
excerpt: 认识微服务框架go-zero
categories: Go
index_img: /img/index_img/7.png
banner_img: /img/banner_img/background3.jpg
---

![](https://raw.githubusercontent.com/univwang/img/master/202303191237766.png)

<a class="btn" target="_blank" rel="noopener" style="font-size:20px; color: green" href="https://go-zero.dev/cn/" title="github">官方文档</a>


## 1. 环境搭建

### goctl

```bash
go install github.com/zeromicro/go-zero/tools/goctl@latest
```

### protoc && protoc-gen-go

```bash
goctl env check -i -f --verbose                
```

## 2. 单体服务

### 2.1 创建项目

```bash
goctl api new hello
cd /hello
go mod tidy
```

```shell
D:\11516\LEARN\GO-ZERO1\HELLO
│  go.mod
│  go.sum
│  hello.api //接口文件
│  hello.go //入口文件
│
├─etc
│      hello-api.yaml //配置文件
│
└─internal
    ├─config
    │      config.go //配置文件
    │
    ├─handler //处理函数
    │      hellohandler.go
    │      routes.go
    │
    ├─logic //业务逻辑
    │      hellologic.go
    │
    ├─svc //服务
    │      servicecontext.go
    │
    └─types //类型
            types.go
```


### 2.2 启动项目
    
```bash
go run hello.go -f etc/hello-api.yaml
```


## 3. 微服务

### 3.1 创建项目

```bash
goctl api new order
goctl rpc new user
go work init # 工作区初始化
go work use order
go work use user 
```

### 3.2 项目结构

```shell
├─order
│  │  go.mod
│  │  order.api
│  │  order.go
│  │
│  ├─etc
│  │      order-api.yaml
│  │
│  └─internal
│      ├─config
│      │      config.go
│      │
│      ├─handler
│      │      orderhandler.go
│      │      routes.go
│      │
│      ├─logic
│      │      orderlogic.go
│      │
│      ├─svc
│      │      servicecontext.go
│      │
│      └─types
│              types.go
│
└─user
    │  go.mod
    │  user.go
    │  user.proto
    │
    ├─etc
    │      user.yaml
    │
    ├─internal
    │  ├─config
    │  │      config.go
    │  │
    │  ├─logic
    │  │      pinglogic.go
    │  │
    │  ├─server
    │  │      userserver.go
    │  │
    │  └─svc
    │          servicecontext.go
    │
    ├─user
    │      user.pb.go
    │      user_grpc.pb.go
    │
    └─userclient
            user.go
```

### 3.3 编写proto


```go
syntax = "proto3";

package user;

// protoc-gen-go 版本大于1.4.0, proto文件需要加上go_package,否则无法生成
option go_package = "./user";

message IdRequest {
  string id = 1;
}

message UserResponse {
  // 用户id
  string id = 1;
  // 用户名称
  string name = 2;
  // 用户性别
  string gender = 3;
}

service User {
  rpc getUser(IdRequest) returns(UserResponse);
}
```


生成代码

```bash
goctl rpc protoc user.proto --go_out=./types --go-grpc_out=./types --zrpc_out=.
```

### 3.4 编写rpc逻辑

在 internal的logic目录下编写rpc逻辑

### 3.5 编写order.api

```go
type(
	OrderReq {
		Id string `path:"id"`
	}

	OrderReply {
		Id string `json:"id"`
		Name string `json:"name"`
	}
)
service order {
	@handler getOrder
	get /api/order/get/:id (OrderReq) returns (OrderReply)
}

```

生成代码

```bash
goctl api go -api order.api -dir .
```

### 3.6 配置grpc服务

1. order 配置

```yaml
Name: order
Host: 0.0.0.0
Port: 8888

UserRpc:
  Etcd:
    Hosts:
      - 127.0.0.1:2379
    Key: user.rpcName: order
Host: 0.0.0.0
Port: 8888

UserRpc:
  Etcd:
    Hosts:
      - 127.0.0.1:2379
    Key: user.rpc
```

2. config / svc 配置

复制userclient文件夹，在config配置`UserRpc zrpc.RpcClientConf`， 在servicecontext中设置实例

```go
package svc

import (
	"github.com/zeromicro/go-zero/zrpc"
	"order/internal/config"
	"order/userclient"
)

type ServiceContext struct {
	Config  config.Config
	UserRpc userclient.User
}

func NewServiceContext(c config.Config) *ServiceContext {
	return &ServiceContext{
		Config:  c,
		UserRpc: userclient.NewUser(zrpc.MustNewClient(c.UserRpc)),
	}
}

```

3. 在logic中调用

```go

func (l *GetOrderLogic) GetOrder(req *types.OrderReq) (resp *types.OrderReply, err error) {
	userId := l.getOrderById(req.Id)                                                                       //获取需要的用户id
	userResponse, err := l.svcCtx.UserRpc.GetUser(context.Background(), &userclient.IdRequest{Id: userId}) //调用user服务
	if err != nil {
		return nil, err
	}
	resp = &types.OrderReply{
		Id:       req.Id,
		Name:     "hello",
		UserName: userResponse.Name,
	}
	return
}
```