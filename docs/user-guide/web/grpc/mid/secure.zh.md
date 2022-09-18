---
title: "安全中间件"
linkTitle: "安全中间件"
weight: 16
description: >
  启动安全中间件。
---

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## Secure 选项
| 名字                                          | 描述                                  | 类型       | 默认值             |
|---------------------------------------------|-------------------------------------|----------|-----------------|
| grpc.middleware.secure.enabled               | 启动安全中间件                             | boolean  | false           |
| grpc.middleware.secure.ignore                | 局部选项，忽略 API 路径                      | []string | []              |
| grpc.middleware.secure.xssProtection         | X-XSS-Protection                    | string   | "1; mode=block" |
| grpc.middleware.secure.contentTypeNosniff    | X-Content-Type-Options              | string   | nosniff         |
| grpc.middleware.secure.xFrameOptions         | X-Frame-Options                     | string   | SAMEORIGIN      |
| grpc.middleware.secure.hstsMaxAge            | Strict-Transport-Security           | int      | 0               |
| grpc.middleware.secure.hstsExcludeSubdomains | HSTS SubDomains                     | bool     | false           |
| grpc.middleware.secure.hstsPreloadEnabled    | 开启 HSTS 预加载。                        | bool     | false           |
| grpc.middleware.secure.contentSecurityPolicy | Content-Security-Policy             | string   | ""              |
| grpc.middleware.secure.cspReportOnly         | Content-Security-Policy-Report-Only | bool     | false           |
| grpc.middleware.secure.referrerPolicy        | Referrer-Policy                     | string   | ""              |

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
      secure:
        enabled: true
#        ignore: [""]
#        xssProtection: ""
#        contentTypeNosniff: ""
#        xFrameOptions: ""
#        hstsMaxAge: 0
#        hstsExcludeSubdomains: false
#        hstsPreloadEnabled: false
#        contentSecurityPolicy: ""
#        cspReportOnly: false
#        referrerPolicy: ""

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
```bash
$ curl -vs localhost:8080/v1/hello
...
< Content-Type: application/json; charset=utf-8
< X-Content-Type-Options: nosniff
< X-Frame-Options: SAMEORIGIN
< X-Xss-Protection: 1; mode=block
< Date: Sat, 16 Apr 2022 12:35:05 GMT
< Content-Length: 20
...
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)