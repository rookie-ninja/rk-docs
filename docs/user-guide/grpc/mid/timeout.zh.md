---
title: "超时中间件"
linkTitle: "超时中间件"
weight: 11
description: >
  启动超时中间件。
---

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## 超时选项
| 名字                                       | 描述             | 类型       | 默认值   |
|------------------------------------------|----------------|----------|-------|
| grpc.middleware.timeout.enabled           | 启动超时中间件        | boolean  | false |
| grpc.middleware.timeout.ignore            | 局部选项，忽略 API 路径 | []string | []    |
| grpc.middleware.timeout.timeoutMs         | 超时时间，毫秒        | int      | 5000  |
| grpc.middleware.timeout.paths.path        | 路径             | string   | ""    |
| grpc.interceptors.timeout.paths.timeoutMs | 基于访问路径的超时时间，毫秒 | int      | 5000  |

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
      timeout:
        enabled: true
#        ignore: [""]
        timeoutMs: 5000
        paths:
          - path: "/api.v1.Greeter/Hello"
            timeoutMs: 1
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
  "time"
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
  time.Sleep(10 * time.Millisecond)

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
        "code":408,
        "status":"Request Timeout",
        "message":"Request timed out!",
        "details":[
            {
                "code":1,
                "status":"Canceled",
                "message":"[from-grpc] Request timed out!"
            }
        ]
    }
}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)