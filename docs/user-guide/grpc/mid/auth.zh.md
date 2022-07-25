---
title: "权限中间件"
linkTitle: "权限中间件"
weight: 11
description: >
  启动权限中间件。
---

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## 权限选项
| 名字                           | 描述                           | 类型       | 默认值   |
|------------------------------|------------------------------|----------|-------|
| grpc.middleware.auth.enabled | 启动权限中间件                      | boolean  | false |
| grpc.middleware.auth.ignore  | 局部选项，忽略 API 路径               | []string | []    |
| grpc.middleware.auth.basic   | Basic Auth 信息，格式：<user:pass> | []string | []    |
| grpc.middleware.auth.apiKey  | X-API-Key                    | []string | []    |
| grpc.middleware.auth.ignore  | 提供字符串前缀，中间件会忽略包含这些字符串的请求路径   | []string | []    |

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
      auth:
        enabled: true
        basic: ["user:pass"]
#        ignore: [""]
#        apiKey:
#          - "keys"
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

func (server *GreeterServer) Hello(_ context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 4.验证
> 如果开启了 enableRkGwOption，则会把 grpc 应设成 Resultful API Error。

```bash
$ curl localhost:8080/v1/hello
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"missing authorization, provide one of bellow auth header:[Basic Auth]",
        "details":[
            {
                "code":16,
                "status":"Unauthenticated",
                "message":"[from-grpc] missing authorization, provide one of bellow auth header:[Basic Auth]"
            }
        ]
    }
}
```

> 如果没有开启 enableRkGwOption，则会返回默认 grpc 错误

```bash
{
    "code":16,
    "message":"missing authorization, provide one of bellow auth header:[Basic Auth]",
    "details":[
        {
            "@type":"type.googleapis.com/rk.api.v1.ErrorDetail",
            "code":16,
            "status":"Unauthenticated",
            "message":"[from-grpc] missing authorization, provide one of bellow auth header:[Basic Auth]"
        }
    ]
}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 5.X-API-Key 类型授权
```yaml
---
grpc:
  - name: greeter
    ...
    middleware:
      auth:
        enabled: true
        apiKey: ["token"]
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
