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
### 1.Create and compile protocol buffer
[Compile protobuf](../buf)

### 2.Create ServerA with 8080
> **bootA.yaml**
```yaml
---
grpc:
  - name: greeter
    port: 1949
#   gwPort: 8081                  # Optional, default: gateway port will be the same as grpc port if not provided
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

### 3.Create ServerB with 8081
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

### 4.Start ServerA & ServerB
```bash
$ go run serverA.go
$ go run serverB.go
```

### 5.Send request to ServerA
```bash
$ curl localhost:8080/v1/greeter
```

### 6.Validate logs
There will be uniq traceId and different requestId & eventId.

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

## Concept
There will be three types of ID generated in logs.

### EventId
Generated while logging middleware was enabled.
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
Generated while logging and meta middleware enabled. RequestId and eventId will the same.

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
Generate while trace middleware was enabled.

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
