---
title: "Trace RPC with logs"
linkTitle: "Trace RPC with logs"
weight: 10
description: >
  How to trace RPC with logs?
---

## Overview
Sometimes, we don't want to install any of tracing service like jaeger because of cost. In that case, we need a way to trace RPC with logs only.

Bootstrapper introduce a way to trace RPC with openTelemetry library but without tracing service.

![](/bootstrapper/user-guide/go/gin/advanced/trace-arch.png)

## Concept
RK bootstrapper will use traceId to track each RPC in distributed system wich will be logged into RPC log while tracing and log interceptor enabled in bootstrapper.

### EventId
EventId would be generated if logging interceptor/middleware was enabled bellow:
```yaml
---
gin:
  - name: greeter
    port: 8080
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
Generated by meta interceptor/middleware automatically if user enabled bellow:
```yaml
---
gin:
  - name: greeter
    port: 8080
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

> If meta interceptor/middleware was enabled or requestId was enabled by user, then eventId will be override by requestId.
>
> Simply, eventId will always the same as requestId
```go
  rkginctx.SetHeaderToClient(ctx, rkginctx.RequestIdKey, "overridden-request-id")
```
```shell script
------------------------------------------------------------------------
...
ids={"eventId":"overridden-request-id","requestId":"overridden-request-id"}
...
```

### TraceId
Generated if user enable tracing interceptor/middleware by user as bellow:
```yaml
---
gin:
  - name: greeter
    port: 8080
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

## Quick start
### 1.Create ServerA at 8080
> **bootA.yaml**
```yaml
---
gin:
  - name: greeter
    port: 8080
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
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-gin/interceptor/context"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot(rkboot.WithBootConfigPath("bootA.yaml"))

	// Register handler
	boot.GetGinEntry("greeter").Router.GET("/v1/hello", func(ctx *gin.Context) {
		client := http.DefaultClient

        // Construct request point to Server B
		req, _ := http.NewRequestWithContext(context.Background(), http.MethodGet, "http://localhost:8081/v1/hello", nil)

		// Inject parent trace info into request header
		rkginctx.InjectSpanToHttpRequest(ctx, req)

        // Send request to Server B
		client.Do(req)

		ctx.JSON(http.StatusOK, "Hello!")
	})

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 2.Create ServerB at 8081
> **bootB.yaml**
```yaml
---
gin:
  - name: greeter
    port: 8081
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
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// @title RK Swagger for Gin
// @version 1.0
// @description This is a greeter service with rk-boot.

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot(rkboot.WithBootConfigPath("bootB.yaml"))

	// Register handler
	boot.GetGinEntry("greeter").Router.GET("/v1/hello", func(ctx *gin.Context) {
		ctx.JSON(http.StatusOK, "Hello!")
	})

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 3.Start ServerA and ServerB
```shell script
$ go run serverA.go
$ go run serverB.go
```

### 4.Send request to ServerA
```shell script
$ curl localhost:8080/v1/hello
```

### 5.Validate logs
Two server will have same traceId in event log but different requestId and eventId.

What we need to do is **grep** same traceId.

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