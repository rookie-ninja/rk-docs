---
title: "Override boot.yaml"
linkTitle: "Override boot.yaml"
weight: 9
description: >
  Override values in boot.yaml with flags and environment variable
---

## Overview
User can override values in boot.yaml with flags and environment variable

- Override value in boot.yaml with (\-\-rkset)

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### 2.Create and compile protocol buffer
[Compile protobuf](../buf)

### 3.Create boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.Create main.go
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

func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 5.Change Port with flag
Follows format of【grpc[0].port】if value was in List.

```bash
$ go run main.go --rkset grpc[0].port=8081
2022-04-17T23:04:53.960+0800    INFO    entry/util.go:332       Found flag to override, applying...     {"flags": ["grpc[0].port=8081"]}
2022-04-17T23:04:53.960+0800    INFO    boot/grpc_entry.go:965  Bootstrap grpcEntry     {"eventId": "df4a9ff3-6537-433e-8718-bb07880070c4", "entryName": "greeter", "entryType": "gRPCEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T23:04:53.961165+08:00
startTime=2022-04-17T23:04:53.960979+08:00
elapsedNano=185688
timezone=CST
ids={"eventId":"df4a9ff3-6537-433e-8718-bb07880070c4"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"grpcPort":8081,"gwPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

> Send request to 8081
```bash
$ curl localhost:8081/v1/hello
{"message":"hello!"}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 6.Override Port with environment variable
【RK_】should be included as prefix while using environment variables.

Example：RK_GRPC_0_PORT

```go
package main

import (
  "context"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-demo/api/gen/v1"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "google.golang.org/grpc"
  "os"
)

func main() {
  os.Setenv("RK_GRPC_0_PORT", "8081")

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

func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

```bash
$ go run main.go
2022-04-17T23:05:49.493+0800    INFO    entry/util.go:299       Found ENV to override, applying...      {"env": ["RK_GRPC_0_PORT=8081 => grpc[0].port=8081"]}
2022-04-17T23:05:49.493+0800    INFO    boot/grpc_entry.go:965  Bootstrap grpcEntry     {"eventId": "fa9dba2d-ec47-4720-ae89-537b61063013", "entryName": "greeter", "entryType": "gRPCEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T23:05:49.493936+08:00
startTime=2022-04-17T23:05:49.49368+08:00
elapsedNano=256504
timezone=CST
ids={"eventId":"fa9dba2d-ec47-4720-ae89-537b61063013"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"grpcPort":8081,"gwPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```