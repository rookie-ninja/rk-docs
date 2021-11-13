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

![](/bootstrapper/user-guide/echo-golang/advanced/trace-arch.png)

## 概念
当启动了日志拦截器，元数据拦截器，调用链拦截器的时候，拦截器会往日志里写入如下三种 ID。

### EventId
当启动了日志拦截器，EventId 会自动生成。

```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
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
当启动了日志拦截器和元数据拦截器，RequestId 和 EventId 会自动生成，并且这两个 ID 会一致。

```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    interceptors:
      meta:
        enabled: true
      loggingZap:
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
  rkechoctx.SetHeaderToClient(ctx, rkechoctx.RequestIdKey, "overridden-request-id")
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
echo:
  - name: greeter
    port: 8080
    enabled: true
    interceptors:
      meta:
        enabled: true
      loggingZap:
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
### 1.创建 ServerA 监听 8080 端口
> **bootA.yaml**
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    interceptors:
      meta:
        enabled: true
      loggingZap:
        enabled: true
      tracingTelemetry:
        enabled: true
```
> **serverA.go**
```go
package main

import (
	"context"
	"github.com/labstack/echo/v4"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-echo/interceptor/context"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot(rkboot.WithBootConfigPath("bootA.yaml"))

	// Register handler
	boot.GetEchoEntry("greeter").Echo.GET("/v1/hello", func(ctx echo.Context) error {
		client := http.DefaultClient

        // Construct request point to Server B
		req, _ := http.NewRequestWithContext(context.Background(), http.MethodGet, "http://localhost:8081/v1/hello", nil)

		// Inject parent trace info into request header
		rkechoctx.InjectSpanToHttpRequest(ctx, req)

        // Send request to Server B
		client.Do(req)

		return ctx.JSON(http.StatusOK, "Hello!")
	})

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 2.创建 ServerB 监听 8081 端口
> **bootB.yaml**
```yaml
---
echo:
  - name: greeter
    port: 8081
    enabled: true
    interceptors:
      meta:
        enabled: true
      loggingZap:
        enabled: true
      tracingTelemetry:
        enabled: true
```
> **serverB.go**
```go
package main

import (
	"context"
	"github.com/labstack/echo/v4"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot(rkboot.WithBootConfigPath("bootB.yaml"))

	// Register handler
	boot.GetEchoEntry("greeter").Echo.GET("/v1/hello", func(ctx echo.Context) error {
		return ctx.JSON(http.StatusOK, "Hello!")
	})

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 3.启动 ServerA 与 ServerB
```shell script
$ go run serverA.go
$ go run serverB.go
```

### 4.往 ServerA 发送请求
```shell script
$ curl localhost:8080/v1/hello
```

### 5.验证日志
两个服务的日志中，会有同样的 traceId，不同的 requestId 和 eventId。

我们可以通过 **grep** traceId 来追踪 RPC。

> ServerA
```shell script
------------------------------------------------------------------------
...
ids={"eventId":"9b2e9278-0e19-4936-9afe-966ff7fce1c9","requestId":"9b2e9278-0e19-4936-9afe-966ff7fce1c9","traceId":"fc3801be6de88116a7c47fafca6a7328"}
...
```
> ServerB
```shell script
------------------------------------------------------------------------
...
ids={"eventId":"7b9585de-eaa3-4ad5-ae7a-e3be7fdb312e","requestId":"7b9585de-eaa3-4ad5-ae7a-e3be7fdb312e","traceId":"fc3801be6de88116a7c47fafca6a7328"}
...
```