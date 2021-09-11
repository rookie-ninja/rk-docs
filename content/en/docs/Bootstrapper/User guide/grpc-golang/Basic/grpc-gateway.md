---
title: "GRPC Gateway"
linkTitle: "GRPC Gateway"
weight: 2
description: >
  Enable grpc-gateway.
---

## Overview
In traditional way, we need to add a bunch of codes in order to enable [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway), 

With bootstrapper, grpc-gateway will be automatically started and bind to one single port with grpc for convenience.

> **Steps to enable grpc-gateway:**
> 1. Write gw_mapping.yaml
> 2. Write buf.yaml & buf.gen.yaml
> 3. Generate gateway file based on gw_mapping.yaml
> 4. Register gateway function into GRPC server

## Prerequisite
Install required tools.
> We recommend use [rk](https://github.com/rookie-ninja/rk) install them easily.
```shell script
# Install RK CLI
$ go get -u github.com/rookie-ninja/rk/cmd/rk

# List available installation
$ rk install
COMMANDS:
    buf                      install buf on local machine
    cfssl                    install cfssl on local machine
    cfssljson                install cfssljson on local machine
    gocov                    install gocov on local machine
    golangci-lint            install golangci-lint on local machine
    mockgen                  install mockgen on local machine
    pkger                    install pkger on local machine
    protobuf                 install protobuf on local machine
    protoc-gen-doc           install protoc-gen-doc on local machine
    protoc-gen-go            install protoc-gen-go on local machine
    protoc-gen-go-grpc       install protoc-gen-go-grpc on local machne
    protoc-gen-grpc-gateway  install protoc-gen-grpc-gateway on local machine
    protoc-gen-openapiv2     install protoc-gen-openapiv2 on local machine
    swag                     install swag on local machine
    rk-std                   install rk standard environment on local machine
    help, h                  Shows a list of commands or help for one command

# Install buf, protoc-gen-go, protoc-gen-go-grpc, protoc-gen-grpc-gateway, protoc-gen-openapiv2
$ rk install protoc-gen-go
$ rk install protoc-gen-go-grpc
$ rk install protoc-gen-go-grpc-gateway
$ rk install protoc-gen-openapiv2
$ rk install buf
```

| Tool | Description | Installation |
| ---- | ---- | ---- |
| [buf](https://docs.buf.build) | Help you create consistent Protobuf APIs that preserve compatibility and comply with design best-practices. | [Install](https://docs.buf.build/installation) |
| [protoc-gen-go](https://github.com/golang/protobuf/tree/master/protoc-gen-go) | Plugin for the Google protocol buffer compiler to generate Go code.. | [Install](https://grpc.io/docs/languages/go/quickstart/) |
| [protoc-gen-go-grpc](https://github.com/grpc/grpc-go) | This project aims to provide that HTTP+JSON interface to your gRPC service. | [Install](https://grpc.io/docs/languages/go/quickstart/) |
| [protoc-gen-grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) | plugin for Google protocol buffer compiler to generate a reverse-proxy, which converts incoming RESTful HTTP/1 requests gRPC invocation. | [Install](https://github.com/grpc-ecosystem/grpc-gateway#installation) |
| [protoc-gen-openapiv2](https://github.com/grpc-ecosystem/grpc-gateway) | plugin for Google protocol buffer compiler to generate open API config file. | [Install](https://github.com/grpc-ecosystem/grpc-gateway#installation) |

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
```

## General options
> These are general options to start a grpc server with rk-boot

| name | description | type | default value | Required |
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | The name of grpc server | string | "", server won't start | Required |
| grpc.port | The port of grpc server | integer | 0, server won't start | Required |
| grpc.description | Description of grpc entry. | string | "" | Optional |
| grpc.enableReflection | Enable grpc server reflection | boolean | false |
| grpc.enableRkGwOption | Rk style server option | bool | false | Optional |
| grpc.gwMappingFilePaths | The path of gw_mapping.yaml files | []string | false |

## Quick start
### 1.Create api/v1/greeter.proto
```protobuf
syntax = "proto3";

package api.v1;

option go_package = "api/v1/greeter";

service Greeter {
  rpc Greeter (GreeterRequest) returns (GreeterResponse) {}
}

message GreeterRequest {
  string name = 1;
}

message GreeterResponse {
  string message = 1;
}
```

### 2.Create api/v1/gw_mapping.yaml
```yaml
type: google.api.Service
config_version: 3

# Please refer google.api.Http in https://github.com/googleapis/googleapis/blob/master/google/api/http.proto file for details.
http:
  rules:
    - selector: api.v1.Greeter.Greeter
      get: /api/v1/greeter
```

### 3.Create buf.yaml
```yaml
version: v1beta1
name: github.com/rk-dev/rk-demo
build:
  roots:
    - api
```

### 4.Create buf.gen.yaml
```yaml
version: v1beta1
plugins:
  # protoc-gen-go needs to be installed, generate go files based on proto files
  - name: go
    out: api/gen
    opt:
     - paths=source_relative
  # protoc-gen-go-grpc needs to be installed, generate grpc go files based on proto files
  - name: go-grpc
    out: api/gen
    opt:
      - paths=source_relative
      - require_unimplemented_servers=false
  # protoc-gen-grpc-gateway needs to be installed, generate grpc-gateway go files based on proto files
  - name: grpc-gateway
    out: api/gen
    opt:
      - paths=source_relative
      - grpc_api_configuration=api/v1/gw_mapping.yaml
  # protoc-gen-openapiv2 needs to be installed, generate swagger config files based on proto files
  - name: openapiv2
    out: api/gen
    opt:
      - grpc_api_configuration=api/v1/gw_mapping.yaml
```

### 5.Compile proto file
```shell script
$ buf generate
```

> There will be bellow files generated.
```shell script
$ tree api/gen 
api/gen
└── v1
    ├── greeter.pb.go
    ├── greeter.pb.gw.go
    ├── greeter.swagger.json
    └── greeter_grpc.pb.go
 
1 directory, 4 files
```

### 6.Create boot.yaml
> **grpc.gw.gwMappingFilePaths** is not required, however, we strongly recommend specify this path since bootstrapper will use this in bellow:
> 1. /rk/v1/apis
> 2. RK TV

```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 8080                      # Port of grpc entry
    gwMappingFilePaths:
      - "api/v1/gw_mapping.yaml"    # Boot will look for gateway mapping files and load information into memory
```

### 7. Create main.go
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-demo/api/gen/v1"
	"google.golang.org/grpc"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

    // ***************************************
    // ******* Register GRPC & Gateway *******
    // ***************************************

	// Get grpc entry with name
	grpcEntry := boot.GetGrpcEntry("greeter")
    // Register grpc registration function
	grpcEntry.AddRegFuncGrpc(registerGreeter)
    // Register grpc-gateway registration function
	grpcEntry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

// Implementation of [type GrpcRegFunc func(server *grpc.Server)]
func registerGreeter(server *grpc.Server) {
	greeter.RegisterGreeterServer(server, &GreeterServer{})
}

// Implementation of grpc service defined in proto file
type GreeterServer struct{}

func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```

### 8.Full structure
```shell script
$ tree
.
├── api
│   ├── gen
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
└── main.go

4 directories, 12 files
```

### 9.Validate
```shell script
$ go run main.go
```

```shell script
$ curl "localhost:8080/api/v1/greeter?name=rk-dev"
{"message":"Hello rk-dev!"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## Gateway server option
> RK introduced gateway server options with bellow functionality.
>
> [rkgrpc.RkGwServerMuxOptions](https://github.com/rookie-ninja/rk-grpc/blob/master/boot/gw_server_options.go)

| Functionality | Description |
| ---- | ---- |
| HttpErrorHandler | Mainly copied from original gateway error handler. RK wrap errors to RK style error response. |
| MarshalerOption | protojson.MarshalOptions{UseProtoNames: false, EmitUnpopulated: true},UnmarshalOptions: protojson.UnmarshalOptions{}} |
| Metadata | Inject x-forwarded-method, x-forwarded-path, x-forwarded-scheme, x-forwarded-user-agent and x-forwarded-remote-addr into grpc metadata |
| OutgoingHeaderMatcher | Bypass all grpc metadata to http header without prefix |
| IncomingHeaderMatcher | Bypass all http header to grpc metadata without prefix |

### 1.Enable rkServerOption
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 8080                      # Port of grpc entry
    enableRkGwOption: true          # Enable rk style grpc gateway server option
    gwMappingFilePaths:
      - "api/v1/gw_mapping.yaml"    # Bootstrapper will look for gateway mapping files and load information into memory
    interceptors:
      loggingZap:
        enabled: true               # Enable logging interceptor for validation
```

### 2.Validate log
```shell script
$ curl "localhost:8080/api/v1/greeter?name=rk-dev"
```

> gwMethod, gwPath, gwScheme, gwUserAgent would be logged
```shell script
------------------------------------------------------------------------
endTime=2021-07-09T21:03:43.518106+08:00
...
payloads={"grpcMethod":"Greeter","grpcService":"api.v1.Greeter","grpcType":"unaryServer","gwMethod":"GET","gwPath":"/v1/greeter","gwScheme":"http","gwUserAgent":"curl/7.64.1"}
...
```

### 3.Validate incoming metadata
```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
    // Print incoming headers
	fmt.Println(rkgrpcctx.GetIncomingHeaders(ctx))

	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```
```shell script
map[:authority:[0.0.0.0:8080] accept:[*/*] content-type:[application/grpc] user-agent:[grpc-go/1.38.0] x-forwarded-for:[::1] x-forwarded-host:[localhost:8080] x-forwarded-method:[GET] x-forwarded-path:[/v1/greeter] x-forwarded-remote-addr:[[::1]:57082] x-forwarded-scheme:[http] x-forwarded-user-agent:[curl/7.64.1]]
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## Error mapping
> It is important to understand grpc-gateway error to grpc error mapping.
>
> Here is the default error mapping defind in grpc-gateway.

| GRPC Error | GRPC Error Str | Gateway(Http) Error Code | Gateway(Http) Error Str |
| ---- | ---- | ---- | ---- |
| 0 | OK | 200 | OK |
| 1 | CANCELLED | 408 | Request Timeout |
| 2 | UNKNOWN | 500 | Internal Server Error |
| 3 | INVALID_ARGUMENT | 400 | Bad Request |
| 4 | DEADLINE_EXCEEDED | 504 | Gateway Timeout |
| 5 | NOT_FOUND | 404 | Not Found |
| 6 | ALREADY_EXISTS | 409 | Conflict |
| 7 | PERMISSION_DENIED | 403 | Forbidden |
| 8 | RESOURCE_EXHAUSTED | 429 | Too Many Requests |
| 9 | FAILED_PRECONDITION | 400 | Bad Request |
| 10 | ABORTED | 409 | Conflict |
| 11 | OUT_OF_RANGE | 400 | Bad Request |
| 12 | UNIMPLEMENTED | 501 | Not Implemented |
| 13 | INTERNAL | 500 | Internal Server Error |
| 14 | UNAVAILABLE | 503 | Service Unavailable |
| 15 | DATA_LOSS | 500 | Internal Server Error |
| 16 | UNAUTHENTICATED | 401 | Unauthorized |

### 1.Validate error(Standard go error)
> Based on default error mapping, we expect 500 response code.
```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	return nil, errors.New("error triggered manually")
}
```
```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{
    "error":{
        "code":500,
        "status":"Internal Server Error",
        "message":"error triggered manually",
        "details":[]
    }
}
```

### 2.Validate error(grpc error)
> grpc needs to specify errors with status.New().
> 
> We recommend use rkerror to generate errors as bellow.

```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	return nil, rkerror.PermissionDenied("permission denied manually").Err()
}
```

```shell script
curl "localhost:8080/v1/greeter?name=rk-dev"
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
![](/bootstrapper/user-guide/cheers.png)