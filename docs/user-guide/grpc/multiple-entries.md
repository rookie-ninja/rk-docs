---
title: "Multiple entries"
linkTitle: "Multiple entries"
weight: 7
description: >
  Start multiple entries in one process.
---

## Overview
User can start multiple Entry in one single process.

Furthermore, use can even start Gin and gRPC in one single process.

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
grpc:
  - name: alice
    port: 8080
    enabled: true
  - name: bob
    port: 8081
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
  alice := rkgrpc.GetGrpcEntry("alice")
  alice.AddRegFuncGrpc(registerGreeter)
  alice.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

  bob := rkgrpc.GetGrpcEntry("bob")
  bob.AddRegFuncGrpc(registerGreeter)
  bob.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

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

### 5.Validate
```bash
$ curl localhost:8080/v1/hello
{"message":"hello!"}

$ curl localhost:8081/v1/hello
{"message":"hello!"}
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)