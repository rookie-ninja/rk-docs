---
title: "原数据中间件"
linkTitle: "原数据中间件"
weight: 8
description: >
  启动原数据中间件。
---

## 概述
原数据中间件将会把下面的信息，以 HTTP 头部的形式，返回给客户。

| Header 键                 | 详情             |
|--------------------------|----------------|
| X-Request-Id             | 自动生成请求 ID。     |
| X-[Prefix]-App-Domain    | 当前环境。          |
| X-[Prefix]-App-Unix-Time | 当前服务的 Unix 时间。 |
| X-[Prefix]-Received-Time | 接收到请求的时间戳。     |

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## 原数据选项
| 名字                          | 描述             | 类型       | 默认值   |
|-----------------------------|----------------|----------|-------|
| grpc.middleware.meta.enabled | 启动原数据中间件       | boolean  | false |
| grpc.middleware.meta.prefix  | X-<Prefix>-XXX | string   | RK    |
| grpc.middleware.meta.ignore  | 局部选项，忽略 API 路径 | []string | []    |

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
      meta:
        enabled: true                                      # Optional, default: false
#        ignore: [""]                                      # Optional, default: []
#        prefix: ""                                        # Optional, default: "RK"
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
$ curl -vs localhost:8080/v1/hello
...
< HTTP/1.1 200 OK
< Content-Type: application/json
< X-Request-Id: 81a32625-e605-4b42-8444-6f59c0d59fd1
< X-Rk-App-Domain: *
< X-Rk-App-Name: 
< X-Rk-App-Unix-Time: 2022-04-17T21:41:43.662239+08:00
< X-Rk-App-Version: 
< X-Rk-Received-Time: 2022-04-17T21:41:43.662239+08:00
...
{"message":"hello!"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 4.覆盖 requestId
```go
func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
    // Override request id
    rkgrpcctx.AddHeaderToClient(ctx, rkmid.HeaderRequestId, "request-id-override")
    // We expect new request id attached to logger
    rkgrpcctx.GetLogger(ctx).Info("Received request")

    return &greeter.HelloResponse{
        Message: "hello!",
    }, nil
}
```

```bash
$ curl -vs localhost:8080/v1/hello
...
< X-Request-Id: 82ec81e4-7e57-463a-8c4f-f5b3abba7b7b
< X-Request-Id: request-id-override
< X-Rk-App-Domain: *
< X-Rk-App-Name: 
< X-Rk-App-Unix-Time: 2022-04-17T21:43:01.123026+08:00
< X-Rk-App-Version: 
< X-Rk-Received-Time: 2022-04-17T21:43:01.123026+08:00
...
{"Message":"Hello rk-dev!"}
```

> 如果我们启动了日志中间件，那我们会看到如下的日志。

```bash
2022-04-17T21:45:14.725+0800    INFO    grpc/main.go:38 Received request        {"requestId": "request-id-override"}
------------------------------------------------------------------------
endTime=2022-04-17T21:45:14.725231+08:00
startTime=2022-04-17T21:45:14.725157+08:00
elapsedNano=74648
timezone=CST
ids={"eventId":"request-id-override","requestId":"request-id-override"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"apiMethod":"","apiPath":"/api.v1.Greeter/Hello","apiProtocol":"","apiQuery":"","grpcMethod":"Hello","grpcService":"api.v1.Greeter","grpcType":"UnaryServer","gwMethod":"GET","gwPath":"/v1/hello","gwScheme":"http","gwUserAgent":"curl/7.64.1","userAgent":""}
counters={}
pairs={}
timing={}
remoteAddr=127.0.0.1:50128
operation=/api.v1.Greeter/Hello
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5.覆盖 header prefix
```yaml
---
grpc:
  - name: greeter
    ...
    middleware:
      meta:
        enabled: true
        prefix: "Override"
```

```bash
$ curl -X GET -vs localhost:8080/v1/hello
...
< X-Override-App-Domain: *
< X-Override-App-Name: 
< X-Override-App-Unix-Time: 2022-04-17T21:46:34.246483+08:00
< X-Override-App-Version: 
< X-Override-Received-Time: 2022-04-17T21:46:34.246483+08:00
< X-Request-Id: 993eeb82-ed52-47a5-8c0f-05912784eb18
< X-Request-Id: request-id-override
...
{"message":"hello!"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)