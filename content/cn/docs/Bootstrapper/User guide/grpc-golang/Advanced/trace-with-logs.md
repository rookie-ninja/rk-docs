---
title: "分布式日志追踪"
linkTitle: "分布式日志追踪"
weight: 10
description: >
  如何在多个进程日志中，通过 traceId 来追踪 RPC 日志？
---

## 概述
当我们没有使用例如 jaeger 的调用链服务的时候，我们希望通过日志来追踪分布式系统里的 RPC 请求。

启动器会通过 openTelemetry 库来向日志写入 traceId 来追踪 RPC。

![](/bootstrapper/user-guide/gin-golang/advanced/trace-arch.png)

## 概念
当启动了日志拦截器，原数据拦截器，调用链拦截器的时候，拦截器会往日志里写入如下三种 ID。

### EventId
当启动了日志拦截器，EventId 会自动生成。
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 1949                      # Port of grpc entry
    enableReflection: true
    commonService:
      enabled: true                 # Enable common service for testing
    interceptors:
      loggingZap:
        enabled: true
```
```shell script
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
  - name: greeter                   # Name of grpc entry
    port: 1949                      # Port of grpc entry
    enableReflection: true
    commonService:
      enabled: true                 # Enable common service for testing
    interceptors:
      loggingZap:
        enabled: true
      meta:
        enabled: true
```
```shell script
------------------------------------------------------------------------
...
ids={"eventId":"8226ba9b-424e-4e19-ba63-d37ca69028b3","requestId":"8226ba9b-424e-4e19-ba63-d37ca69028b3"}
...
```

> 即使用户覆盖了 RequestId，EventId 也会保持一致。

```go
  rkgrpcctx.AddHeaderToClient(ctx, rkgrpcctx.RequestIdKey, "overridden-request-id")
```
```shell script
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
  - name: greeter                   # Name of grpc entry
    port: 1949                      # Port of grpc entry
    enableReflection: true
    commonService:
      enabled: true                 # Enable common service for testing
    interceptors:
      loggingZap:
        enabled: true
      meta:
        enabled: true
      tracingTelemetry:
        enabled: true
```
```shell script
------------------------------------------------------------------------
...
ids={"eventId":"dd19cf9a-c7be-486c-b29d-7af777a78ebe","requestId":"dd19cf9a-c7be-486c-b29d-7af777a78ebe","traceId":"316a7b475ff500a76bfcd6147036951c"}
...
```

## 快速开始
### 1.创建 ServerA 监听 1949 端口
> **bootA.yaml**
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 1949                      # Port of grpc entry
    enableReflection: true
    interceptors:
      loggingZap:
        enabled: true
      meta:
        enabled: true
      tracingTelemetry:
        enabled: true
```
> **serverA.go**
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-demo/api/gen/v1"
	"github.com/rookie-ninja/rk-grpc/interceptor/context"
	"google.golang.org/grpc"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot(rkboot.WithBootConfigPath("bootA.yaml"))

	// Get grpc entry with name
	grpcEntry := boot.GetGrpcEntry("greeter")
	grpcEntry.AddRegFuncGrpc(registerGreeter)
	grpcEntry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

func registerGreeter(server *grpc.Server) {
	greeter.RegisterGreeterServer(server, &GreeterServer{})
}

type GreeterServer struct{}

func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
    // Call serverB at 2008 with grpc client
	opts := []grpc.DialOption {
		grpc.WithBlock(),
		grpc.WithInsecure(),
	}
	conn, _ := grpc.Dial("localhost:2008", opts...)
	defer conn.Close()
	client := greeter.NewGreeterClient(conn)
	
    // Inject current trace information into context
	newCtx := rkgrpcctx.InjectSpanToNewContext(ctx)
	client.Greeter(newCtx, &greeter.GreeterRequest{Name: "A"})

	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
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
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-demo/api/gen/v1"
	"github.com/rookie-ninja/rk-grpc/interceptor/context"
	"google.golang.org/grpc"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot(rkboot.WithBootConfigPath("bootB.yaml"))

	// Get grpc entry with name
	grpcEntry := boot.GetGrpcEntry("greeter")
	grpcEntry.AddRegFuncGrpc(registerGreeterB)
	grpcEntry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

func registerGreeterB(server *grpc.Server) {
	greeter.RegisterGreeterServer(server, &GreeterServerB{})
}

type GreeterServerB struct{}

func (server *GreeterServerB) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```

### 3.启动 ServerA 与 ServerB
```shell script
$ go run serverA.go
$ go run serverB.go
```

### 4.往 ServerA 发送请求
```shell script
# Call grpc, both grpc and http will have same effect
$ grpcurl -plaintext localhost:1949 api.v1.Greeter.Greeter
# Call http, both grpc and http will have same effect
$ curl "localhost:8080/v1/greeter?name=rk-dev"
```

### 5.验证日志
两个服务的日志中，会有同样的 traceId，不同的 requestId 和 eventId。

我们可以通过 **grep** traceId 来追踪 RPC。

> ServerA
```shell script
------------------------------------------------------------------------
...
ids={"eventId":"05928652-642c-4c4a-829e-b27b81c979c7","requestId":"05928652-642c-4c4a-829e-b27b81c979c7","traceId":"3614ffe216458f445a611a20e41be948"}
...
```
> ServerB
```shell script
------------------------------------------------------------------------
...
ids={"eventId":"9b12380e-293d-4501-bc55-80a5b1295748","requestId":"9b12380e-293d-4501-bc55-80a5b1295748","traceId":"3614ffe216458f445a611a20e41be948"}
...
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
