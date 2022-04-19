---
title: "Trace with logs"
linkTitle: "Trace with logs"
weight: 10
description: >
  How to trace logs in distributed system?
---

## Overview
We will introduce a way to trace logs in distributed system without tracing service like jaeger.

## Quick start
### 1.Install

```shell
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-echo
```

### 1.Create ServerA with 8080
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

### 2.Create ServerB with 8081
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

### 3.Start ServerA & ServerB
```shell script
$ go run serverA.go
$ go run serverB.go
```

### 4.Send request to ServerA
```shell
$ curl localhost:8080/v1/greeter
```

### 5.Validate logs
There will be uniq traceId and different requestId & eventId.

> ServerA
```shell script
------------------------------------------------------------------------
...
ids={"eventId":"0928ec71-01ea-4cc3-9c33-61711eff1689","requestId":"0928ec71-01ea-4cc3-9c33-61711eff1689","traceId":"0dd657c8522cfadc3cfc98e37db9b527"}
...
```

> ServerB
```shell script
------------------------------------------------------------------------
...
ids={"eventId":"3adcaabc-14d7-40b0-a8d2-8f8cdb58b311","requestId":"3adcaabc-14d7-40b0-a8d2-8f8cdb58b311","traceId":"0dd657c8522cfadc3cfc98e37db9b527"}
...
```

## Concept
There will be three types of ID generated in logs.

### EventId
Generated while logging middleware was enabled.

```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      logging:
        enabled: true
```

```shell script
------------------------------------------------------------------------
...
ids={"eventId":"cd617f0c-2d93-45e1-bef0-95c89972530d"}
...
```

### RequestId
Generated while logging and meta middleware enabled. RequestId and eventId will the same.

```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      meta:
        enabled: true
      logging:
        enabled: true
```

```shell script
------------------------------------------------------------------------
...
ids={"eventId":"8226ba9b-424e-4e19-ba63-d37ca69028b3","requestId":"8226ba9b-424e-4e19-ba63-d37ca69028b3"}
...
```

```go
  rkginctx.SetHeaderToClient(ctx, rkmid.HeaderRequestId, "overridden-request-id")
```

```shell script
------------------------------------------------------------------------
...
ids={"eventId":"overridden-request-id","requestId":"overridden-request-id"}
...
```

### TraceId
Generate while trace middleware was enabled.

```yaml
---
gin:
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

```shell script
------------------------------------------------------------------------
...
ids={"eventId":"dd19cf9a-c7be-486c-b29d-7af777a78ebe","requestId":"dd19cf9a-c7be-486c-b29d-7af777a78ebe","traceId":"316a7b475ff500a76bfcd6147036951c"}
...
```