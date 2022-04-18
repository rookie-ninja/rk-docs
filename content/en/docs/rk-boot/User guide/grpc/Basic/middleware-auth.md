---
title: "Middleware auth"
linkTitle: "Middleware auth"
weight: 11
description: >
  Enable auth middleware
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## Options
| options                      | description                        | type     | default |
|------------------------------|------------------------------------|----------|---------|
| grpc.middleware.auth.enabled | Enable auth middleware             | boolean  | false   |
| grpc.middleware.auth.ignore   | Ignore by path                     | []string | []      |
| grpc.middleware.auth.basic    | Basic Auth info，format：<user:pass> | []string | []      |
| grpc.middleware.auth.apiKey   | X-API-Key                          | []string | []      |

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
      auth:
        enabled: true
        basic: ["user:pass"]
#        ignore: [""]
#        apiKey:
#          - "keys"
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

func (server *GreeterServer) Hello(_ context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 4.Validate
> if enableRkGwOption was set, then bellow error will occur.

```shell script
$ curl localhost:8080/v1/hello
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"missing authorization, provide one of bellow auth header:[Basic Auth]",
        "details":[
            {
                "code":16,
                "status":"Unauthenticated",
                "message":"[from-grpc] missing authorization, provide one of bellow auth header:[Basic Auth]"
            }
        ]
    }
}
```

> if enableRkGwOption was NOT set, then bellow error will occur.

```shell
{
    "code":16,
    "message":"missing authorization, provide one of bellow auth header:[Basic Auth]",
    "details":[
        {
            "@type":"type.googleapis.com/rk.api.v1.ErrorDetail",
            "code":16,
            "status":"Unauthenticated",
            "message":"[from-grpc] missing authorization, provide one of bellow auth header:[Basic Auth]"
        }
    ]
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 5.X-API-Key 
```yaml
---
grpc:
  - name: greeter
    ...
    middleware:
      auth:
        enabled: true
        apiKey: ["token"]
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
