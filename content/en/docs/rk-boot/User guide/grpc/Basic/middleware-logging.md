---
title: "Middleware logging"
linkTitle: "Middleware logging"
weight: 6
description: >
  Enable logging middleware
---

## Install
```shell
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## Options
| options                                   | description                 | type     | default |
|-------------------------------------------|-----------------------------|----------|---------|
| grpc.middleware.logging.enabled           | Enable logging middleware   | boolean  | false   |
| grpc.middleware.logging.ignore            | Ignore by path              | []string | []      |
| grpc.middleware.logging.loggerEncoding    | Logger format：json, console | string   | console |
| grpc.middleware.logging.loggerOutputPaths | Logger output path          | []string | stdout  |
| grpc.middleware.logging.eventEncoding     | Event format：json, console  | string   | console |
| grpc.middleware.logging.eventOutputPaths  | Event output path           | []string | stdout  |

## Concept
1. [Event](https://github.com/rookie-ninja/rk-query)
2. [Logger](https://github.com/uber-go/zap)

### Logger
rk-boot use [zap](https://github.com/uber-go/zap) as underlying logger instance，[lumberjack](https://github.com/natefinch/lumberjack) as log rotation.

> Example
>
> ```shell script
> 2021-07-05T23:35:17.104+0800    INFO    boot/gin_entry.go:631   Bootstrapping GinEntry. {"eventId": "08d11a34-472a-4785-a98c-144a3417d7f0", "entryName": "greeter", "entryType": "GinEntry", "port": 8080, "interceptorsCount": 1, "swEnabled": false, "tlsEnabled": false, "commonServiceEnabled": false, "tvEnabled": false, "promPath": "/metrics", "promPort": 8080}
> ```

### Event
rk-boot will record RPC metadata into **Event**.

| Fields      | Description                                                                                                 |
|-------------|-------------------------------------------------------------------------------------------------------------|
| endTime     | End time                                                                                                    |
| startTime   | Start time                                                                                                  |
| elapsedNano | Event elapsed（Nanoseconds）                                                                                  |
| timezone    | Timezone                                                                                                    |
| ids         | Includes eventId, requestId and traceId                                                                     |
| app         | Includes [appName, appVersion](https://github.com/rookie-ninja/rk-entry#appinfoentry), entryName, entryType |
| env         | Includes arch, domain, hostname, localIP, os 字段。这些字段来自系统环境变量（DOMAIN）。 "*" 代表环境变量为空                          |
| payloads    | Includes RPC metadata                                                                                       |
| error       | Includes errors                                                                                             |
| counters    | Set through event.SetCounter()                                                                              |
| pairs       | Set through event.AddPair()                                                                                 |
| timing      | Set through event.StartTimer() and event.EndTimer()                                                         |
| remoteAddr  | RPC remote address                                                                                          |
| operation   | RPC name                                                                                                    |
| resCode     | RPC return code                                                                                             |
| eventStatus | Ended or InProgress                                                                                         |

> Example
>
> ```shell script
> ------------------------------------------------------------------------
> endTime=2021-06-25T01:30:45.144023+08:00
> startTime=2021-06-25T01:30:45.143767+08:00
> elapsedNano=255948
> timezone=CST
> ids={"eventId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","requestId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","traceId":"65b9aa7a9705268bba492fdf4a0e5652"}
> app={"appName":"rk-gin","appVersion":"master-xxx","entryName":"greeter","entryType":"GinEntry"}
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

## Quick start
### 1.Create and compile protocol buffer
[Compile protobuf](/en/docs/rk-boot/user-guide/grpc/basic/buf/)

### 2.Create boot.yaml
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

### 3.Create main.go
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

### 4.Validate
> Send request

```shell script
$ curl localhost:8080/v1/hello                                             
{"message":"hello!"}
```

> Logs

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

### 5. JSON format
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

### 6. Change output path
> Logs will be rotated every 1GB

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

### 7. Get log instance
rk-boot will assign RequestId into zap logger for every RPC
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

### 8. Change Event
rk-boot will create a new Event for every RPC.

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
