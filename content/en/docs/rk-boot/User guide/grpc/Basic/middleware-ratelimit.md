---
title: "Middleware ratelimit"
linkTitle: "Middleware ratelimit"
weight: 10
description: >
  Enable ratelimit middleware
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## Options
| options                     | description                        | type     | default |
|------------------------------------------|-------------------------------|----------|-------------|
| grpc.middleware.rateLimit.enabled         | Enable ratelimit middleware   | boolean  | false       |
| grpc.middleware.rateLimit.ignore  | Ignore by path                | []string | []    |
| grpc.middleware.rateLimit.algorithm       | leakyBucket is supported only | string   | leakyBucket |
| grpc.middleware.rateLimit.reqPerSec       | global limit                  | int      | 1000000     |
| grpc.middleware.rateLimit.paths.path      | API path                      | string   | ""          |
| grpc.middleware.rateLimit.paths.reqPerSec | limit value of path           | int      | 1000000     |

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
    middleware:
      rateLimit:
        enabled: true
        paths:
          - path: "/v1/hello"
            reqPerSec: 0
#        algorithm: "leakyBucket"
#        reqPerSec: 100
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
> Send request

```shell script
$ curl localhost:8080/v1/hello
{
    "error":{
        "code":429,
        "status":"Too Many Requests",
        "message":"slow down your request",
        "details":[
            {
                "code":8,
                "status":"ResourceExhausted",
                "message":"[from-grpc] slow down your request"
            }
        ]
    }
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)