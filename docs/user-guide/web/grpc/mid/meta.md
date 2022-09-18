Enable meta middleware

## Overview
Meta middleware will inject bellow information into response header.

| Header key               | Description                     |
|--------------------------|---------------------------------|
| X-Request-Id             | Auto generated request ID       |
| X-[Prefix]-App-Domain    | ENV value of DOMAIN             |
| X-[Prefix]-App-Unix-Time | Current Unix time               |
| X-[Prefix]-Received-Time | Unix time when request received |

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## Options
| options                     | description                        | type     | default |
|-----------------------------|------------------------|----------|-------|
| grpc.middleware.meta.enabled | Enable meta middleware | boolean  | false |
| grpc.middleware.meta.prefix  | X-<Prefix>-XXX         | string   | RK    |
| grpc.middleware.meta.ignore  | Ignore by path         | []string | []    |

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
      meta:
        enabled: true                                      # Optional, default: false
#        ignore: [""]                                      # Optional, default: []
#        prefix: ""                                        # Optional, default: "RK"
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

```bash
$ curl -vs localhost:8080/v1/hello
...
< HTTP/1.1 200 OK
< Content-Type: application/json
< X-Request-Id: 81a32625-e605-4b42-8444-6f59c0d59fd1
< X-Rk-App-Domain: *
< X-Rk-App-Name: 
< X-Rk-App-Unix-Time: 2022-04-17T21:41:43.662239+08:00
< X-Rk-App-Version: 
< X-Rk-Received-Time: 2022-04-17T21:41:43.662239+08:00
...
{"message":"hello!"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 4.Override requestId
```go
func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
    // Override request id
    rkgrpcctx.AddHeaderToClient(ctx, rkmid.HeaderRequestId, "request-id-override")
    // We expect new request id attached to logger
    rkgrpcctx.GetLogger(ctx).Info("Received request")

    return &greeter.HelloResponse{
        Message: "hello!",
    }, nil
}
```

```bash
$ curl -vs localhost:8080/v1/hello
...
< X-Request-Id: 82ec81e4-7e57-463a-8c4f-f5b3abba7b7b
< X-Request-Id: request-id-override
< X-Rk-App-Domain: *
< X-Rk-App-Name: 
< X-Rk-App-Unix-Time: 2022-04-17T21:43:01.123026+08:00
< X-Rk-App-Version: 
< X-Rk-Received-Time: 2022-04-17T21:43:01.123026+08:00
...
{"Message":"Hello rk-dev!"}
```

> RequestId will be injected into logger if logger middleware was enabled.

```bash
2022-04-17T21:45:14.725+0800    INFO    grpc/main.go:38 Received request        {"requestId": "request-id-override"}
------------------------------------------------------------------------
endTime=2022-04-17T21:45:14.725231+08:00
startTime=2022-04-17T21:45:14.725157+08:00
elapsedNano=74648
timezone=CST
ids={"eventId":"request-id-override","requestId":"request-id-override"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"apiMethod":"","apiPath":"/api.v1.Greeter/Hello","apiProtocol":"","apiQuery":"","grpcMethod":"Hello","grpcService":"api.v1.Greeter","grpcType":"UnaryServer","gwMethod":"GET","gwPath":"/v1/hello","gwScheme":"http","gwUserAgent":"curl/7.64.1","userAgent":""}
counters={}
pairs={}
timing={}
remoteAddr=127.0.0.1:50128
operation=/api.v1.Greeter/Hello
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5.Override header prefix
```yaml
---
grpc:
  - name: greeter
    ...
    middleware:
      meta:
        enabled: true
        prefix: "Override"
```

```bash
$ curl -X GET -vs localhost:8080/v1/hello
...
< X-Override-App-Domain: *
< X-Override-App-Name: 
< X-Override-App-Unix-Time: 2022-04-17T21:46:34.246483+08:00
< X-Override-App-Version: 
< X-Override-Received-Time: 2022-04-17T21:46:34.246483+08:00
< X-Request-Id: 993eeb82-ed52-47a5-8c0f-05912784eb18
< X-Request-Id: request-id-override
...
{"message":"hello!"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)