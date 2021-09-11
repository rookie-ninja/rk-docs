---
title: "GRPC Gateway"
linkTitle: "GRPC Gateway"
weight: 2
description: >
  启动 grpc-gateway.
---

## 概述
如果想要启动 [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)，并且提供企业级别的服务，我们需要写很多的代码。

我们可以通过启动器，通过编辑 boot.yaml 文件，快速启动 grpc-gateway。

> **启动 grpc-gateway 步骤:**
> 1. 创建 gw_mapping.yaml
> 2. 创建 buf.yaml & buf.gen.yaml
> 3. 基于 gw_mapping.yaml 文件，编译 protocol buffer
> 4. 把生成出来的 gateway 函数注册到 GRPC 服务中

## 先决条件
安装所需的工具
> 建议通过 [rk](https://github.com/rookie-ninja/rk) 命令行，快速安装所需要的工具。
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

| 工具 | 介绍 | 安装 |
| ---- | ---- | ---- |
| [buf](https://docs.buf.build) | 通过一站式命令行，快速编译 protocol buffer API | [Install](https://docs.buf.build/installation) |
| [protoc-gen-go](https://github.com/golang/protobuf/tree/master/protoc-gen-go) | 从 proto 文件，创建 .go 文件的插件 | [Install](https://grpc.io/docs/languages/go/quickstart/) |
| [protoc-gen-go-grpc](https://github.com/grpc/grpc-go) | 从 proto 文件，创建 GRPC 相关的 .go 文件的插件 | [Install](https://grpc.io/docs/languages/go/quickstart/) |
| [protoc-gen-grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) | 从 proto 文件，创建 grpc-gateway 相关的 .go 文件 | [Install](https://github.com/grpc-ecosystem/grpc-gateway#installation) |
| [protoc-gen-openapiv2](https://github.com/grpc-ecosystem/grpc-gateway) | 从 proto 文件，创建 swagger 界面所需的参数文件 | [Install](https://github.com/grpc-ecosystem/grpc-gateway#installation) |

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 GRPC 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 | 必要与否
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | GRPC 服务名称 | string | "", server won't start | Required |
| grpc.port | GRPC 服务端口 | integer | 0, server won't start | Required |
| grpc.description | GRPC 服务的描述 | string | "" | Optional |
| grpc.enableReflection | 启动 GRPC 反射功能 | boolean | false |
| grpc.enableRkGwOption | Rk 推荐的 grpc gateway server option | bool | false | Optional |
| grpc.gwMappingFilePaths | gw_mapping.yaml 文件路径 | []string | false |

## 快速开始
### 1.创建 api/v1/greeter.proto
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

### 2.创建 api/v1/gw_mapping.yaml
```yaml
type: google.api.Service
config_version: 3

# Please refer google.api.Http in https://github.com/googleapis/googleapis/blob/master/google/api/http.proto file for details.
http:
  rules:
    - selector: api.v1.Greeter.Greeter
      get: /api/v1/greeter
```

### 3.创建 buf.yaml
```yaml
version: v1beta1
name: github.com/rk-dev/rk-demo
build:
  roots:
    - api
```

### 4.创建 buf.gen.yaml
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

### 5.编译 proto file
```shell script
$ buf generate
```

> 如下的文件会被创建。
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

### 6.创建 boot.yaml
> **grpc.gw.gwMapingFilePaths** 是非必要的选项，但是，我们强烈推荐添加此选项，此文件能完整化如下的功能中。
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

### 7. 创建 main.go
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

### 8.文件夹结构
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

### 9.验证
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
> RK 推荐的 server option，提供如下的功能。
>
> [rkgrpc.RkGwServerMuxOptions](https://github.com/rookie-ninja/rk-grpc/blob/master/boot/gw_server_options.go)

| 功能 | 详情 |
| ---- | ---- |
| HttpErrorHandler | 主要代码从原有 grpc-gateway 代码中抄写而来，启动器会返回 RK 推荐的 API 错误结构 |
| MarshalerOption | protojson.MarshalOptions{UseProtoNames: false, EmitUnpopulated: true},UnmarshalOptions: protojson.UnmarshalOptions{}} |
| Metadata | 注入如下 GRPC metadata： x-forwarded-method, x-forwarded-path, x-forwarded-scheme, x-forwarded-user-agent 和 x-forwarded-remote-addr |
| OutgoingHeaderMatcher | 把原有的 grpc metadata 转发到 http 头部（没有前缀） |
| IncomingHeaderMatcher | 把原有的 http 头部 转发到 grpc metadata（没有前缀） |

### 1.启动 rkServerOption
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

### 2.验证日志
```shell script
$ curl "localhost:8080/api/v1/greeter?name=rk-dev"
```

> gwMethod, gwPath, gwScheme, gwUserAgent 将会记录到日志中
```shell script
------------------------------------------------------------------------
endTime=2021-07-09T21:03:43.518106+08:00
...
payloads={"grpcMethod":"Greeter","grpcService":"api.v1.Greeter","grpcType":"unaryServer","gwMethod":"GET","gwPath":"/v1/greeter","gwScheme":"http","gwUserAgent":"curl/7.64.1"}
...
```

### 3.验证传入的 metadata
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

## 错误映射
> grpc-gateway 中，我们需要了解 gateway 到 grpc 的错误映射，即 http 到 grpc 的错误映射。
>
> 这是默认的 grpc-gateway 中的错误映射

| GRPC 错误码 | GRPC 错误码描述 | Gateway(Http) 错误码 | Gateway(Http) 错误码描述 |
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

### 1.验证错误（标准 Go 语言错误）
> 根据错误映射，500 将会返回。
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

### 2.验证错误（grpc 错误）
> 我们需要通过 status.New() 来创建 GRPC 错误。
> 
> 我们推荐使用 rkerror 库中的函数来创建错误。

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