---
title: "日志中间件"
linkTitle: "日志中间件"
weight: 6
description: >
  启动 API 请求日志中间件。
---

## 安装
```shell
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## 选项
| 名字                                        | 描述                   | 类型       | 默认值     |
|-------------------------------------------|----------------------|----------|---------|
| grpc.middleware.logging.enabled           | 启动日志拦截器              | boolean  | false   |
| grpc.middleware.logging.ignore            | 局部选项，忽略 API 路径       | []string | []      |
| grpc.middleware.logging.loggerEncoding    | 日志格式：json 或者 console | string   | console |
| grpc.middleware.logging.loggerOutputPaths | 日志文件路径               | []string | stdout  |
| grpc.middleware.logging.eventEncoding     | 日志格式：json 或者 console | string   | console |
| grpc.middleware.logging.eventOutputPaths  | 日志文件路径               | []string | stdout  |

## 概念
我们需要提前了解两个概念。

1. [Event](https://github.com/rookie-ninja/rk-query)
2. [Logger](https://github.com/uber-go/zap)

### Logger
rk-boot 使用 [zap](https://github.com/uber-go/zap) 为默认日志，[lumberjack](https://github.com/natefinch/lumberjack) 为日志滚动库。

> 例子
>
> ```shell script
2022-04-17T21:34:06.029+0800    INFO    boot/grpc_entry.go:960  Bootstrap grpcEntry     {"eventId": "a84186c3-0e05-4714-9276-80b3b68d2733", "entryName": "greeter", "entryType": "gRPCEntry"}
> ```

### Event
rk-boot 把每一个 RPC 请求视作 **Event**，并且使用 [rk-query](https://github.com/rookie-ninja/rk-query) 中的 Event 类型来记录日志。

| 字段 | 详情 |
| ---- | ---- |
| endTime | 结束时间 |
| startTime | 开始时间 |
| elapsedNano | Event 时间开销（Nanoseconds） |
| timezone | 时区 |
| ids | 包含 eventId, requestId 和 traceId。如果元数据拦截器被启动，或者 event.SetRequest() 被用户调用，新的 RequestId 将会被使用，同时 eventId 与 requestId 会一模一样。 如果调用链拦截器被启动，traceId 将会被记录。|
| app | 包含 [appName, appVersion](https://github.com/rookie-ninja/rk-entry#appinfoentry), entryName, entryType。 |
| env | 包含 arch, domain, hostname, localIP, os 字段。这些字段来自系统环境变量（DOMAIN）。 "*" 代表环境变量为空。|
| payloads | 包含 RPC 相关信息。 |
| error | 包含错误。|
| counters | 通过 event.SetCounter() 来操作。|
| pairs | 通过 event.AddPair() 来操作。 |
| timing | 通过 event.StartTimer() 和 event.EndTimer() 来操作。 |
| remoteAddr | RPC 远程地址。 |
| operation | RPC 名字。 |
| resCode | RPC 返回码。 |
| eventStatus | Ended 或者 InProgress |

> 例子
>
> ```shell script
> ------------------------------------------------------------------------
> endTime=2021-06-25T01:30:45.144023+08:00
> startTime=2021-06-25T01:30:45.143767+08:00
> elapsedNano=255948
> timezone=CST
> ids={"eventId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","requestId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","traceId":"65b9aa7a9705268bba492fdf4a0e5652"}
> app={"appName":"rk-grpc","appVersion":"master-xxx","entryName":"greeter","entryType":"GrpcEntry"}
> env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
> payloads={"apiMethod":"GET","apiPath":"/rk/v1/healthy","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
> error={}
> counters={}
> pairs={}
> timing={}
> remoteAddr=localhost:60718
> operation=/rk/v1/healthy
> resCode=200
> eventStatus=Ended
> EOE
> ```

## 快速开始
### 1.创建并编译 protocol buffer
[使用 buf 编译 protocol buf](/cn/docs/rk-boot/user-guide/grpc/basic/buf/)

### 2. 创建 boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      logging:
        enabled: true
#        ignore: [""]
#        loggerEncoding: "console"
#        loggerOutputPaths: ["logs/app.log"]
#        eventEncoding: "console"
#        eventOutputPaths: ["logs/event.log"]
```

### 3. 创建 main.go
```go
package main

import (
  "context"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-demo/api/gen/v1"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "github.com/rookie-ninja/rk-grpc/v2/middleware/context"
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

### 4. 验证
> 发送请求

```shell script
$ curl localhost:8080/v1/hello                                             
{"message":"hello!"}
```

> 日志

```shell script
------------------------------------------------------------------------
endTime=2022-04-17T21:34:40.049531+08:00
startTime=2022-04-17T21:34:40.049501+08:00
elapsedNano=30062
timezone=CST
ids={"eventId":"6b6b26c1-e493-4539-956d-c69c4da99a9f"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"apiMethod":"","apiPath":"/api.v1.Greeter/Hello","apiProtocol":"","apiQuery":"","grpcMethod":"Hello","grpcService":"api.v1.Greeter","grpcType":"UnaryServer","gwMethod":"GET","gwPath":"/v1/hello","gwScheme":"http","gwUserAgent":"curl/7.64.1","userAgent":""}
counters={}
pairs={}
timing={}
remoteAddr=127.0.0.1:49849
operation=/api.v1.Greeter/Hello
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 5. JSON 格式
```yaml
---
grpc:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerEncoding: "json"
        eventEncoding: "json"
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 6. 修改日志文件路径
> 修改日志文件路径之后，默认在 1GB 大小之后，进行切割，并压缩。

```yaml
---
grpc:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerOutputPaths: ["logs/app.log"]
        eventOutputPaths: ["logs/event.log"]
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 7. 获取日志实例
当每一次 RPC 请求进来的时候，中间件会把 RequestId（当启动了原数据中间件）注入到日志实例中。换句话说，每一个 RPC 请求，都会有一个新的 Logger 实例。
```go
func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
    rkgrpcctx.GetLogger(ctx).Info("Received request")

    return &greeter.HelloResponse{
        Message: "hello!",
    }, nil
}
```

```shell script
2022-04-15T19:17:01.380+0800    INFO    grpc/main.go:34  Received request        {"requestId": "446c1138-1b20-48f7-a479-7b3e8a26de40"}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 8. 修改 Event
日志中间件会为每一个 RPC 请求创建一个 Event 实例。

用户可以添加 pairs，counters，errors。
```go
func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
    event := rkgrpcctx.GetEvent(ctx)
    event.AddPair("key", "value")

    return &greeter.HelloResponse{
        Message: "hello!",
    }, nil
}
```

```shell script
------------------------------------------------------------------------
endTime=2022-04-17T21:38:29.673039+08:00
startTime=2022-04-17T21:38:29.673028+08:00
elapsedNano=10656
timezone=CST
ids={"eventId":"5ae475a4-a529-42cb-b80b-073a75995d17"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"apiMethod":"","apiPath":"/api.v1.Greeter/Hello","apiProtocol":"","apiQuery":"","grpcMethod":"Hello","grpcService":"api.v1.Greeter","grpcType":"UnaryServer","gwMethod":"GET","gwPath":"/v1/hello","gwScheme":"http","gwUserAgent":"curl/7.64.1","userAgent":""}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=127.0.0.1:49930
operation=/api.v1.Greeter/Hello
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
