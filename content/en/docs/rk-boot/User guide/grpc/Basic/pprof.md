---
title: "PPROF"
linkTitle: "PPROF"
weight: 4
description: >
  Enable pprof UI。
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## 选项
| options           | description          | type    | default |
|-------------------|----------------------|---------|---------|
| grpc.pprof.enabled | Eanble pprof web UI  | boolean | false   |
| grpc.pprof.path    | Path of pprof web UI | string  | pprof   |

## Quick start
### 1.Create and compile protocol buffer
[Compile protobuf](/en/docs/rk-boot/user-guide/grpc/basic/buf/)

### 2.Create boot.yaml

```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    pprof:
      enabled: true
#      path: ""
```

### 3.Create main.go
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

### 4.Validate
> **PPROF:** [http://localhost:8080/pprof](http://localhost:8080/pprof)

![](/rk-boot/user-guide/gin/basic/gin-pprof.png)

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
