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

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-zero
```

### 1.Create ServerA with 8080
> **bootA.yaml**
```yaml
---
zero:
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
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-zero/boot"
  "github.com/rookie-ninja/rk-zero/middleware/context"
  "github.com/zeromicro/go-zero/rest"
  "net/http"
)

// @title Swagger Example API
// @version 1.0
// @description This is a sample rk-demo server.
// @termsOfService http://swagger.io/terms/

// @securityDefinitions.basic BasicAuth

// @contact.name API Support
// @contact.url http://www.swagger.io/support
// @contact.email support@swagger.io

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot(rkboot.WithBootConfigPath("serverA.yaml", nil))

  // Register handler
  zeroEntry := rkzero.GetZeroEntry("greeter")
  zeroEntry.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: func(writer http.ResponseWriter, request *http.Request) {
      client := http.DefaultClient

      // Construct request point to Server B
      req, _ := http.NewRequestWithContext(context.Background(), http.MethodGet, "http://localhost:8081/v1/greeter", nil)

      // Inject parent trace info into request header
      rkzeroctx.InjectSpanToHttpRequest(request, req)

      // Send request to Server B
      client.Do(req)

      writer.WriteHeader(http.StatusOK)
      writer.Write([]byte("Hello!"))
    },
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
zero:
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
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-zero/boot"
  "github.com/zeromicro/go-zero/rest"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot(rkboot.WithBootConfigPath("serverB.yaml", nil))

  // Register handler
  zeroEntry := rkzero.GetZeroEntry("greeter")
  zeroEntry.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: func(writer http.ResponseWriter, request *http.Request) {
      writer.WriteHeader(http.StatusOK)
      writer.Write([]byte("Hello!"))
    },
  })

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

```

### 3.Start ServerA & ServerB
```bash
$ go run serverA.go
$ go run serverB.go
```

### 4.Send request to ServerA
```bash
$ curl localhost:8080/v1/greeter
```

### 5.Validate logs
There will be uniq traceId and different requestId & eventId.

> ServerA
```bash
------------------------------------------------------------------------
...
ids={"eventId":"6055f89d-04d2-44fa-afd6-75d8585bee8e","requestId":"6055f89d-04d2-44fa-afd6-75d8585bee8e","traceId":"d2ad1e15362761dfb36087aacdfededb"}
...
```

> ServerB
```bash
------------------------------------------------------------------------
...
ids={"eventId":"328d3b58-1df7-4257-9446-d8850aa2ecda","requestId":"328d3b58-1df7-4257-9446-d8850aa2ecda","traceId":"d2ad1e15362761dfb36087aacdfededb"}
...
```

## Concept
There will be three types of ID generated in logs.

### EventId
Generated while logging middleware was enabled.

```yaml
---
zero:
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
Generated while logging and meta middleware enabled. RequestId and eventId will the same.

```yaml
---
zero:
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

```go
  rkzeroctx.SetHeaderToClient(writer, rkmid.HeaderRequestId, "overridden-request-id")
```

```bash
------------------------------------------------------------------------
...
ids={"eventId":"overridden-request-id","requestId":"overridden-request-id"}
...
```

### TraceId
Generate while trace middleware was enabled.

```yaml
---
zero:
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