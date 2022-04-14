---
title: "gRPC 框架"
linkTitle: "gRPC 框架"
weight: 2
description: >
  通过 rk-boot，配合 rk-grpc 插件，创建 [gRPC](https://grpc.io/docs/languages/go/quickstart/) 后台服务。
---

## 概述
我们将会使用 rk-boot 启动 [gRPC](https://grpc.io) 微服务，并且添加 /v1/hello API。

为了微服务的完整性，我们还会开启如下几个附加功能。

| 功能             | 介绍                                  |
|:---------------|:------------------------------------|
| grpc-gateway   | 默认开启 grpc-gateway                   |
| Swagger UI     | 开启 Swagger UI                       |
| API Docs UI    | 开启 RapiDoc UI                       |
| Prometheus 客户端 | 开启 Prometheus 本地客户端                 |
| 日志中间件          | 针对每个 API 请求，自动记录日志                  |
| Prometheus 中间件 | 针对每个 API 请求，自动记录 Prometheus 记录      |
| 原数据中间件         | 针对每个 API 请求，自动添加 RequestID 到 Header |

## 安装 gRPC 配套命令行
> 编译 [protocol buffer](https://grpc.io/docs/languages/go/quickstart/) 的过程较为复杂，我们需要一系列工具的支持。
>
> 通过 [rk](https://github.com/rookie-ninja/rk) 来快速安装所需要的工具。

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

| 工具                                                                            | 介绍                                    | 安装                                                                     |
|-------------------------------------------------------------------------------|---------------------------------------|------------------------------------------------------------------------|
| [buf](https://docs.buf.build)                                                 | 通过一站式命令行，快速编译 protocol buffer API     | [Install](https://docs.buf.build/installation)                         |
| [protoc-gen-go](https://github.com/golang/protobuf/tree/master/protoc-gen-go) | 从 proto 文件，创建 .go 文件的插件               | [Install](https://grpc.io/docs/languages/go/quickstart/)               |
| [protoc-gen-go-grpc](https://github.com/grpc/grpc-go)                         | 从 proto 文件，创建 GRPC 相关的 .go 文件的插件      | [Install](https://grpc.io/docs/languages/go/quickstart/)               |
| [protoc-gen-grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)     | 从 proto 文件，创建 grpc-gateway 相关的 .go 文件 | [Install](https://github.com/grpc-ecosystem/grpc-gateway#installation) |
| [protoc-gen-openapiv2](https://github.com/grpc-ecosystem/grpc-gateway)        | 从 proto 文件，创建 swagger 界面所需的参数文件       | [Install](https://github.com/grpc-ecosystem/grpc-gateway#installation) |

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## 1.创建 greeter.proto
在 **api/v1** 文件夹中创建 greeter.proto 文件。

```protobuf
syntax = "proto3";

package api.v1;

option go_package = "api/v1/greeter";

service Greeter {
  rpc Hello (HelloRequest) returns (HelloResponse) {}
}

message HelloRequest {}

message HelloResponse {
  string message = 1;
}
```

## 2.复制 googleapis
rk-grpc 插件用到了一些 googleapi 里面的基础结构。我们推荐在项目工程中，存放这些文件。

可以直接到 [github/googleapi](https://github.com/googleapis/googleapis) 复制相应的文件，也可以从 [rk-boot](https://github.com/rookie-ninja/rk-boot/tree/main/example/web/grpc/third-party) 的例子中，复制相应的文件。

把如下文件复制到 third-party 文件夹中。

```shell
third-party
    └── googleapis
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

## 3.创建 buf 参数文件
> **buf.yaml**

```yaml
version: v1beta1
name: github.com/rk-dev/rk-boot
build:
  roots:
    - api
    - third-party/googleapis
```

> **buf.gen.yaml**

```yaml
version: v1beta1
plugins:
  - name: go
    out: api/gen
    opt:
      - paths=source_relative
  - name: go-grpc
    out: api/gen
    opt:
      - paths=source_relative
      - require_unimplemented_servers=false
  - name: grpc-gateway
    out: api/gen
    opt:
      - paths=source_relative
      - grpc_api_configuration=api/v1/gw_mapping.yaml
      - allow_repeated_fields_in_body=true
      - generate_unbound_methods=true
  - name: openapiv2
    out: api/gen
    opt:
      - grpc_api_configuration=api/v1/gw_mapping.yaml
      - allow_repeated_fields_in_body=true
```

> 创建 **api/v1/gw_mapping.yaml**

```yaml
type: google.api.Service
config_version: 3

# Please refer google.api.Http in third-party/googleapis/google/api/http.proto file for details.
http:
  rules:
    - selector: api.v1.Greeter.Hello
      get: /v1/hello
```

> **执行 buf**

```shell script
$ buf generate --path api/v1
# By configuration in buf.gen.yaml, generated files would be write to api/gen folder
$ tree
.
├── api
│   ├── gen
│   │   ├── google
│   │   │   ...
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
        ...
```

## 4.创建 boot.yaml
```yaml
grpc:
  - name: greeter
    port: 8080                    # 监听端口
    enabled: true                 # 开启 微服务
    enableReflection: true        # 开启 gRPC 反射，用于 grpcurl
    enableRkGwOption: true        # 开启 RK 默认 grpc-gateway 选项
    sw:
      enabled: true               # 开启 Swagger UI，默认路径为 /sw
    docs:
      enabled: true               # 开启 API Doc UI，默认路径为 /docs
    prom:
      enabled: true               # 开启 Prometheus 客户端，默认路径为 /metrics
    middleware:
      logging:
        enabled: true             # 开启 API 日志中间件
      prom:
        enabled: true             # 开启 API Prometheus 中间件
      meta:
        enabled: true             # 开启 API 原数据中间件，自动生成 RequestID
```

## 5.创建 main.go
```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.

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

	// 注册 RPC
	entry := rkgrpc.GetGrpcEntry("greeter")
	entry.AddRegFuncGrpc(registerGreeter)
	entry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

	// 启动
	boot.Bootstrap(context.TODO())

	// 等待关闭信号
	boot.WaitForShutdownSig(context.TODO())
}

func registerGreeter(server *grpc.Server) {
	greeter.RegisterGreeterServer(server, &GreeterServer{})
}

type GreeterServer struct{}

func (server *GreeterServer) Hello(_ context.Context, _ *greeter.HelloRequest) (*greeter.HelloResponse, error) {
	return &greeter.HelloResponse{
		Message: "Hello!",
	}, nil
}
```

## 6.启动 main.go
```shell
$ go run main.go 
2022-04-14T16:25:23.538+0800    INFO    boot/grpc_entry.go:960  Bootstrap grpcEntry     {"eventId": "f09d0b94-b491-4148-8438-3e65610fbdde", "entryName": "greeter", "entryType": "gRPCEntry"}
2022-04-14T16:25:23.542+0800    INFO    boot/grpc_entry.go:681  SwaggerEntry: http://localhost:8080/sw/
2022-04-14T16:25:23.542+0800    INFO    boot/grpc_entry.go:684  DocsEntry: http://localhost:8080/docs/
2022-04-14T16:25:23.542+0800    INFO    boot/grpc_entry.go:687  PromEntry: http://localhost:8080/metrics
------------------------------------------------------------------------
endTime=2022-04-14T16:25:23.54236+08:00
startTime=2022-04-14T16:25:23.537974+08:00
elapsedNano=4385739
timezone=CST
ids={"eventId":"f09d0b94-b491-4148-8438-3e65610fbdde"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin"}
payloads={"docsEnabled":true,"docsPath":"/docs/","grpcPort":8080,"gwPort":8080,"promEnabled":true,"promPath":"/metrics","promPort":8080,"swEnabled":true,"swPath":"/sw/"}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

## 7.验证
### 7.1 Swagger UI
[http://localhost:8080/sw/](http://localhost:8080/sw/)

![](/rk-boot/getting-started/grpc/sw-grpc.png)

### 7.2 API Docs UI
我们使用了 RapiDocs 作为 API Docs UI。其实 RapiDocs 也可以替换 Swagger UI 测试 API，后续我们会考虑替换 Swagger UI。

[http://localhost:8080/docs/](http://localhost:8080/docs/)

![](/rk-boot/getting-started/grpc/docs-grpc.png)

### 7.3 Prometheus 客户端
[http://localhost:8080/metrics](http://localhost:8080/metrics)

![](/rk-boot/getting-started/grpc/metrics-grpc.png)

### 7.4 发送请求
> Restful API

```shell
$ curl -vs localhost:8080/v1/hello               
...
< X-Request-Id: b047072d-a433-4b98-b2ff-3ba54bbb0243
< X-Rk-App-Domain: *
< X-Rk-App-Name: 
< X-Rk-App-Unix-Time: 2022-04-14T16:35:28.387937+08:00
< X-Rk-App-Version: 
< X-Rk-Received-Time: 2022-04-14T16:35:28.387937+08:00
...
{"message":"Hello!"}
```

> gRCP

```shell
$ grpcurl -v -plaintext localhost:8080 api.v1.Greeter.Hello

Resolved method descriptor:
rpc Hello ( .api.v1.HelloRequest ) returns ( .api.v1.HelloResponse );

Request metadata to send:
(empty)

Response headers received:
content-type: application/grpc
x-request-id: c783eeaa-2a77-44ec-bb9c-9cbf19a58ee6
x-rk-app-domain: *
x-rk-app-name: 
x-rk-app-unix-time: 2022-04-14T16:36:59.258112+08:00
x-rk-app-version: 
x-rk-received-time: 2022-04-14T16:36:59.258112+08:00

Response contents:
{
  "message": "Hello!"
}
...
```

### 7.5 验证 API 日志
rk-boot 默认会使用如下格式打印 API 日志，也可以使用 JSON 格式，请参考用户指南。

```shell
------------------------------------------------------------------------
endTime=2022-04-14T16:35:28.387969+08:00
startTime=2022-04-14T16:35:28.387928+08:00
elapsedNano=41020
timezone=CST
ids={"eventId":"b047072d-a433-4b98-b2ff-3ba54bbb0243","requestId":"b047072d-a433-4b98-b2ff-3ba54bbb0243"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin"}
payloads={"apiMethod":"","apiPath":"/api.v1.Greeter/Hello","apiProtocol":"","apiQuery":"","grpcMethod":"Hello","grpcService":"api.v1.Greeter","grpcType":"UnaryServer","gwMethod":"GET","gwPath":"/v1/hello","gwScheme":"http","gwUserAgent":"curl/7.64.1","userAgent":""}
counters={}
pairs={}
timing={}
remoteAddr=127.0.0.1:56369
operation=/api.v1.Greeter/Hello
resCode=OK
eventStatus=Ended
EOE
```

### 7.6 验证 Prometheus Metrics
访问 [http://localhost:8080/metrics](http://localhost:8080/metrics)

![](/rk-boot/getting-started/grpc/api-metrics-grpc.png)

## _**Cheers**_
![](/rk-boot/user-guide/cheers.png)