---
title: "限流中间件"
linkTitle: "限流中间件"
weight: 10
description: >
  启动限流中间件。
---

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## 选项
| 名字                                       | 描述                   | 类型      | 默认值         |
|------------------------------------------|----------------------|---------|-------------|
| grpc.middleware.rateLimit.enabled         | 启动限流中间件              | boolean | false       |
| grpc.middleware.rateLimit.ignore            | 局部选项，忽略 API 路径       | []string | []      |
| grpc.middleware.rateLimit.algorithm       | 限流算法， 支持 leakyBucket | string  | leakyBucket |
| grpc.middleware.rateLimit.reqPerSec       | 全局限流值                | int     | 1000000     |
| grpc.middleware.rateLimit.paths.path      | 访问路径                 | string  | ""          |
| grpc.middleware.rateLimit.paths.reqPerSec | 基于访问路径的限流值           | int     | 1000000     |

## 快速开始
### 1.创建并编译 protocol buffer
[使用 buf 编译 protocol buf](../buf)

### 2.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      rateLimit:
        enabled: true
        paths:
          - path: "/v1/hello"
            reqPerSec: 0
#        algorithm: "leakyBucket"
#        reqPerSec: 100
```

### 3.创建 main.go
```go
package main

import (
  "context"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-demo/api/gen/v1"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "google.golang.org/grpc"
)

func main() {
  boot := rkboot.NewBoot()

  // register grpc
  entry := rkgrpc.GetGrpcEntry("greeter")
  entry.AddRegFuncGrpc(registerGreeter)
  entry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Wait for shutdown sig
  boot.WaitForShutdownSig(context.TODO())
}

func registerGreeter(server *grpc.Server) {
  greeter.RegisterGreeterServer(server, &GreeterServer{})
}

type GreeterServer struct{}

func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 4.验证
> 发送请求

```bash
$ curl localhost:8080/v1/hello
{
    "error":{
        "code":429,
        "status":"Too Many Requests",
        "message":"slow down your request",
        "details":[
            {
                "code":8,
                "status":"ResourceExhausted",
                "message":"[from-grpc] slow down your request"
            }
        ]
    }
}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)