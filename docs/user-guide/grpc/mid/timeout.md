Enable timeout middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## Options
| options                                   | description               | type     | default |
|-------------------------------------------|---------------------------|----------|---------|
| grpc.middleware.timeout.enabled           | Enable timeout middleware | boolean  | false   |
| grpc.middleware.timeout.ignore            | Ignore by path            | []string | []      |
| grpc.middleware.timeout.timeoutMs         | Timeouts in milliseconds  | int      | 5000    |
| grpc.middleware.timeout.paths.path        | API path                  | string   | ""      |
| grpc.interceptors.timeout.paths.timeoutMs | Timeouts in milliseconds  | int      | 5000    |

## Quick start
### 1.Create and compile protocol buffer
[Compile protobuf](../buf)

### 2.Create boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      timeout:
        enabled: true
#        ignore: [""]
        timeoutMs: 5000
        paths:
          - path: "/api.v1.Greeter/Hello"
            timeoutMs: 1
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
  "time"
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
  time.Sleep(10 * time.Millisecond)

  return &greeter.HelloResponse{
    Message: "hello!",
  }, nil
}
```

### 4.Validate
> Send request

```bash
$ curl localhost:8080/v1/hello
{
    "error":{
        "code":408,
        "status":"Request Timeout",
        "message":"Request timed out!",
        "details":[
            {
                "code":1,
                "status":"Canceled",
                "message":"[from-grpc] Request timed out!"
            }
        ]
    }
}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)