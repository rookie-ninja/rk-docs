---
title: "grpc-gateway"
linkTitle: "grpc-gateway"
weight: 2
description: >
  Enable grpc-gateway.
---

## Overview
By default, gRPC and grpc-gateway wil use the same port if service was started by rk-boot.

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## Options
| Name                    | Description                                                                 | Type    | Default |
|-------------------------|-----------------------------------------------------------------------------|---------|---------|
| grpc.name               | gRPC name                                                                   | string  | ""      |
| grpc.port               | gRPC port                                                                   | integer | 0       |
| grpc.enabled            | Enable gRPC                                                                 | bool    | false   |
| grpc.description        | gRPC description                                                            | string  | ""      |
| grpc.enableReflection   | Enable gRPC reflection                                                      | boolean | false   |
| grpc.enableRkGwOption   | Enable RK sytle grpc-gateway option which pass all headers to gRPC metadata | boolean | false   |
| grpc.noRecvMsgSizeLimit | gRPC server side message size limit                                         | int     | 4000000 |

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
    enableReflection: true
    enableRkGwOption: true
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

### 4.Directory hierarchy
```shell script
$ tree
.
├── Makefile
├── README.md
├── api
│   ├── gen
│   │   ├── google
│   │   │   ├── api
│   │   │   │   ├── annotations.pb.go
│   │   │   │   ├── http.pb.go
│   │   │   │   └── httpbody.pb.go
│   │   │   └── rpc
│   │   │       ├── code.pb.go
│   │   │       ├── error_details.pb.go
│   │   │       └── status.pb.go
│   │   └── v1
│   │       ├── greeter.pb.go
│   │       ├── greeter.pb.gw.go
│   │       ├── greeter.swagger.json
│   │       └── greeter_grpc.pb.go
│   └── v1
│       ├── greeter.proto
│       └── gw_mapping.yaml
├── boot.yaml
├── buf.gen.yaml
├── buf.yaml
├── go.mod
├── go.sum
├── main.go
└── third-party
    └── googleapis
        ├── LICENSE
        ├── README.grpc-gateway
        └── google
            ├── api
            │   ├── annotations.proto
            │   ├── http.proto
            │   └── httpbody.proto
            └── rpc
                ├── code.proto
                ├── error_details.proto
                └── status.proto
```

### 5.Validate
```shell script
$ go run main.go
2022-04-17T18:15:55.603+0800    INFO    boot/grpc_entry.go:960  Bootstrap grpcEntry     {"eventId": "46a79118-2966-4b55-a062-8714e6ac54ac", "entryName": "greeter", "entryType": "gRPCEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T18:15:55.603368+08:00
startTime=2022-04-17T18:15:55.603117+08:00
elapsedNano=251193
timezone=CST
ids={"eventId":"46a79118-2966-4b55-a062-8714e6ac54ac"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"grpcPort":8080,"gwPort":8080}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

> Restful API

```shell script
$ curl localhost:8080/v1/hello             
{"message":"hello!"}
```

> gRPC

```shell
$ grpcurl -plaintext localhost:8080 api.v1.Greeter.Hello 
{
  "message": "hello!"
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

## Gateway server option
### 1.Enable RkServerOption
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      logging:
        enabled: true
```

### 2.Validate logs
```shell script
$ curl localhost:8080/v1/hello
```

> gwMethod, gwPath, gwScheme, gwUserAgent will be in Event
```shell script
------------------------------------------------------------------------
endTime=2021-07-09T21:03:43.518106+08:00
...
payloads={"grpcMethod":"Hello","grpcService":"api.v1.Greeter","grpcType":"unaryServer","gwMethod":"GET","gwPath":"/v1/hello","gwScheme":"http","gwUserAgent":"curl/7.64.1"}
...
```

### 3.Validate gRPC metadata
```go
func (server *GreeterServer) Hello(ctx context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
    fmt.Println(rkgrpcctx.GetIncomingHeaders(ctx))

    return &greeter.HelloResponse{
        Message: "hello!",
    }, nil
}
```

```shell script
map[:authority:[0.0.0.0:8080] accept:[*/*] content-type:[application/grpc] user-agent:[grpc-go/1.44.1-dev] x-forwarded-for:[127.0.0.1] x-forwarded-host:[localhost:8080] x-forwarded-method:[GET] x-forwarded-path:[/v1/hello] x-forwarded-remote-addr:[127.0.0.1:49273] x-forwarded-scheme:[http] x-forwarded-user-agent:[curl/7.64.1]]
```

### 4.Override gateway server option for marshaller
Please refer bellow codes to override marshaller.
- [protobuf-go/encoding/protojson/encode.go](https://github.com/protocolbuffers/protobuf-go/blob/master/encoding/protojson/encode.go#L43)
- [protobuf-go/encoding/protojson/decode.go ](https://github.com/protocolbuffers/protobuf-go/blob/master/encoding/protojson/decode.go#L33)

```yaml
grpc:
  - name: greeter                                     # Required
    port: 8080                                        # Required
    enabled: true                                     # Required
    enableRkGwOption: true                            # Optional, default: false
    gwOption:                                         # Optional, default: nil
      marshal:                                        # Optional, default: nil
        multiline: false                              # Optional, default: false
        emitUnpopulated: false                        # Optional, default: false
        indent: ""                                    # Optional, default: false
        allowPartial: false                           # Optional, default: false
        useProtoNames: false                          # Optional, default: false
        useEnumNumbers: false                         # Optional, default: false
      unmarshal:                                      # Optional, default: nil
        allowPartial: false                           # Optional, default: false
        discardUnknown: false                         # Optional, default: false
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

## Error mapping

| gRPC Code | gRPC Status         | Gateway(Http) Code | Gateway(Http) Status  |
|-----------|---------------------|--------------------|-----------------------|
| 0         | OK                  | 200                | OK                    |
| 1         | CANCELLED           | 408                | Request Timeout       |
| 2         | UNKNOWN             | 500                | Internal Server Error |
| 3         | INVALID_ARGUMENT    | 400                | Bad Request           |
| 4         | DEADLINE_EXCEEDED   | 504                | Gateway Timeout       |
| 5         | NOT_FOUND           | 404                | Not Found             |
| 6         | ALREADY_EXISTS      | 409                | Conflict              |
| 7         | PERMISSION_DENIED   | 403                | Forbidden             |
| 8         | RESOURCE_EXHAUSTED  | 429                | Too Many Requests     |
| 9         | FAILED_PRECONDITION | 400                | Bad Request           |
| 10        | ABORTED             | 409                | Conflict              |
| 11        | OUT_OF_RANGE        | 400                | Bad Request           |
| 12        | UNIMPLEMENTED       | 501                | Not Implemented       |
| 13        | INTERNAL            | 500                | Internal Server Error |
| 14        | UNAVAILABLE         | 503                | Service Unavailable   |
| 15        | DATA_LOSS           | 500                | Internal Server Error |
| 16        | UNAUTHENTICATED     | 401                | Unauthorized          |

### 1.Validate error
> Expect 500
```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	return nil, errors.New("error triggered manually")
}
```

```shell script
$ curl localhost:8080/v1/hello
{
    "error":{
        "code":500,
        "status":"Internal Server Error",
        "message":"error triggered manually",
        "details":[]
    }
}
```

### 2.Validate error（grpc error）
> We will use status.New() to create gRPC error.

```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	return nil, rkgrpcerr.PermissionDenied("permission denied manually").Err()
}
```

```shell script
$ curl localhost:8080/v1/hello
{
    "error":{
        "code":403,
        "status":"Forbidden",
        "message":"permission denied manually",
        "details":[
            {
                "code":7,
                "status":"PermissionDenied",
                "message":"[from-grpc] permission denied manually"
            }
        ]
    }
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)