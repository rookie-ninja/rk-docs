如何在多个进程日志中，通过 traceId 来追踪 RPC 日志？

## 概述
当我们没有使用例如 jaeger 的调用链服务的时候，我们希望通过日志来追踪分布式系统里的 RPC 请求。

启动器会通过 openTelemetry 库来向日志写入 traceId 来追踪 RPC。

## 快速开始
### 1.创建并编译 protocol buffer
[使用 buf 编译 protocol buf](../buf)

### 2.创建 ServerA 监听 1949 端口
> **bootA.yaml**
```yaml
---
grpc:
  - name: greeter
    port: 1949
#   gwPort: 8081                  # 可选项，如果不指定，会使用与 port 一样的端口
    enabled: true
    middleware:
      logging:
        enabled: true
      meta:
        enabled: true
      trace:
        enabled: true
```

> **serverA.go**
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
  boot := rkboot.NewBoot(rkboot.WithBootConfigPath("serverA.yaml", nil))

  // register grpc
  entry := rkgrpc.GetGrpcEntry("greeter")
  entry.AddRegFuncGrpc(registerGreeterA)
  entry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Wait for shutdown sig
  boot.WaitForShutdownSig(context.TODO())
}

func registerGreeterA(server *grpc.Server) {
  greeter.RegisterGreeterServer(server, &GreeterServerA{})
}

type GreeterServerA struct{}

func (server *GreeterServerA) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  // Call serverB at 2008 with grpc client
  opts := []grpc.DialOption{
    grpc.WithBlock(),
    grpc.WithInsecure(),
  }
  conn, _ := grpc.Dial("localhost:2008", opts...)
  defer conn.Close()
  client := greeter.NewGreeterClient(conn)

  // Inject current trace information into context
  newCtx := rkgrpcctx.InjectSpanToNewContext(ctx)
  client.Hello(newCtx, &greeter.HelloRequest{})

  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 2.创建 ServerB 监听 2008 端口
> **bootB.yaml**
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 2008                      # Port of grpc entry
    enabled: true                   # Enable grpc entry
    interceptors:
      loggingZap:
        enabled: true
      meta:
        enabled: true
      tracingTelemetry:
        enabled: true
```

> **serverB.go**
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
  boot := rkboot.NewBoot(rkboot.WithBootConfigPath("serverB.yaml", nil))

  // register grpc
  entry := rkgrpc.GetGrpcEntry("greeter")
  entry.AddRegFuncGrpc(registerGreeterB)
  entry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Wait for shutdown sig
  boot.WaitForShutdownSig(context.TODO())
}

func registerGreeterB(server *grpc.Server) {
  greeter.RegisterGreeterServer(server, &GreeterServerB{})
}

type GreeterServerB struct{}

func (server *GreeterServerB) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 3.启动 ServerA 与 ServerB
```bash
$ go run serverA.go
$ go run serverB.go
```

### 4.往 ServerA 发送请求
```bash
# Call http, both grpc and http will have same effect
$ curl localhost:1949/v1/hello
{"message":"hello!"}
```

### 5.验证日志
两个服务的日志中，会有同样的 traceId，不同的 requestId 和 eventId。

我们可以通过 **grep** traceId 来追踪 RPC。

> ServerA
```bash
------------------------------------------------------------------------
...
ids={"eventId":"1e5d04db-7095-42d8-bf04-586120bec67f","requestId":"1e5d04db-7095-42d8-bf04-586120bec67f","traceId":"644769270c63218c82a7dfc725b1d194"}
...
```
> ServerB
```bash
------------------------------------------------------------------------
...
ids={"eventId":"01ec723a-bf74-443a-aae6-dc3cf9608be0","requestId":"01ec723a-bf74-443a-aae6-dc3cf9608be0","traceId":"644769270c63218c82a7dfc725b1d194"}
...
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

## 概念
当启动了日志拦截器，原数据拦截器，调用链拦截器的时候，拦截器会往日志里写入如下三种 ID。

### EventId
当启动了日志拦截器，EventId 会自动生成。
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      logging:
        enabled: true
```

```bash
------------------------------------------------------------------------
...
ids={"eventId":"cd617f0c-2d93-45e1-bef0-95c89972530d"}
...
```

### RequestId
当启动了日志拦截器和原数据拦截器，RequestId 和 EventId 会自动生成，并且这两个 ID 会一致。

```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      logging:
        enabled: true
      meta:
        enabled: true
```

```bash
------------------------------------------------------------------------
...
ids={"eventId":"8226ba9b-424e-4e19-ba63-d37ca69028b3","requestId":"8226ba9b-424e-4e19-ba63-d37ca69028b3"}
...
```

> 即使用户覆盖了 RequestId，EventId 也会保持一致。

```go
  rkgrpcctx.AddHeaderToClient(ctx, rkmid.HeaderRequestId, "overridden-request-id")
```

```bash
------------------------------------------------------------------------
...
ids={"eventId":"overridden-request-id","requestId":"overridden-request-id"}
...
```

### TraceId
当启动了调用链拦截器，traceId 会自动生成。

```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      loggingZ:
        enabled: true
      meta:
        enabled: true
      trace:
        enabled: true
```

```bash
------------------------------------------------------------------------
...
ids={"eventId":"dd19cf9a-c7be-486c-b29d-7af777a78ebe","requestId":"dd19cf9a-c7be-486c-b29d-7af777a78ebe","traceId":"316a7b475ff500a76bfcd6147036951c"}
...
```
