如何在多个进程日志中，通过 traceId 来追踪 RPC 日志？

## 概述
当我们没有使用例如 jaeger 的调用链服务的时候，我们希望通过日志来追踪分布式系统里的 RPC 请求。

启动器会通过 openTelemetry 库来向日志写入 traceId 来追踪 RPC。

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-echo
```

### 1.创建 ServerA 监听 8080 端口
> **bootA.yaml**
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      meta:
        enabled: true
      logging:
        enabled: true
      trace:
        enabled: true
```

> **serverA.go**
```go
package main

import (
  "context"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-echo/boot"
  "github.com/rookie-ninja/rk-echo/middleware/context"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot(rkboot.WithBootConfigPath("serverA.yaml", nil))

  // Register handler
  echoEntry := rkecho.GetEchoEntry("greeter")
  echoEntry.Echo.GET("/v1/greeter", func(ctx echo.Context) error {
    client := http.DefaultClient

    // Construct request point to Server B
    req, _ := http.NewRequestWithContext(context.Background(), http.MethodGet, "http://localhost:8081/v1/greeter", nil)

    // Inject parent trace info into request header
    rkechoctx.InjectSpanToHttpRequest(ctx, req)

    // Send request to Server B
    client.Do(req)

    return ctx.JSON(http.StatusOK, "Hello!")
  })

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
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
    middleware:
      meta:
        enabled: true
      logging:
        enabled: true
      trace:
        enabled: true
```

> **serverB.go**
```go
package main

import (
  "context"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-echo/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot(rkboot.WithBootConfigPath("serverB.yaml", nil))

  // Register handler
  echoEntry := rkecho.GetEchoEntry("greeter")
  echoEntry.Echo.GET("/v1/greeter", func(ctx echo.Context) error {
    return ctx.JSON(http.StatusOK, "Hello!")
  })

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}
```

### 3.启动 ServerA 与 ServerB
```bash
$ go run serverA.go
$ go run serverB.go
```

### 4.往 ServerA 发送请求
```bash
$ curl localhost:8080/v1/greeter
```

### 5.验证日志
两个服务的日志中，会有同样的 traceId，不同的 requestId 和 eventId。

我们可以通过 **grep** traceId 来追踪 RPC。

> ServerA
```bash
------------------------------------------------------------------------
...
ids={"eventId":"0928ec71-01ea-4cc3-9c33-61711eff1689","requestId":"0928ec71-01ea-4cc3-9c33-61711eff1689","traceId":"0dd657c8522cfadc3cfc98e37db9b527"}
...
```

> ServerB
```bash
------------------------------------------------------------------------
...
ids={"eventId":"3adcaabc-14d7-40b0-a8d2-8f8cdb58b311","requestId":"3adcaabc-14d7-40b0-a8d2-8f8cdb58b311","traceId":"0dd657c8522cfadc3cfc98e37db9b527"}
...
```

## 概念
当启动了日志中间件，原数据中间件，调用链中间件的时候，中间件会往日志里写入如下三种 ID。

### EventId
当启动了日志中间件，EventId 会自动生成。

```yaml
---
echo:
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
当启动了日志中间件和原数据中间件，RequestId 和 EventId 会自动生成，并且这两个 ID 会一致。

```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      meta:
        enabled: true
      logging:
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
  rkechoctx.SetHeaderToClient(ctx, rkmid.HeaderRequestId, "overridden-request-id")
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
echo:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      meta:
        enabled: true
      logging:
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