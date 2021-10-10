---
title: "Swagger 界面"
linkTitle: "Swagger 界面"
weight: 3
description: >
  启动 Swagger 界面.
---

## 先决条件
启动 Swagger，我们需要先启动 grpc-gateway。

请根据如下步骤启动： [启动 gRPC-gateway](/cn/docs/bootstrapper/user-guide/go/grpc/basic/grpc-gateway/#prerequisite)

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 GRPC 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 | 必要与否
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | GRPC 服务名称 | string | "", 服务不会启动 | Required |
| grpc.port | GRPC 服务端口 | integer | 0, 服务不会启动 | Required |
| grpc.enabled | GRPC 服务启动开关 ｜ bool | false | Required |
| grpc.description | GRPC 服务的描述 | string | "" | Optional |

## Swagger 选项
| 名字 | 描述 | 类型 | 默认值 | 必要与否
| ------ | ------ | ------ | ------ | ------ |
| grpc.sw.enabled | 启动 Swagger | boolean | false |
| grpc.sw.path | Swagger Web 界面路径 | string | /sw |
| grpc.sw.jsonPath | 本地 Swagger 参数文件（swagger.json）路径本地 Swagger 参数文件（swagger.json）路径 | string | "" |
| grpc.sw.headers | 每次 Swagger 界面请求，都会带着这些头部。格式： [key:value] | []string | [] |

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

> 如下文件将会被自动创建。
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
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 8080                      # Port of grpc entry
    enabled: true                   # Enable grpc entry
    enableRkGwOption: true          # Enable RK style server options
    gwMappingFilePaths:
      - "api/v1/gw_mapping.yaml"    # Bootstrapper will look for gateway mapping files and load information into memory
    sw:
      enabled: true                 # Enable swagger
      jsonPath: "api/gen/v1"            # Provide swagger config file path
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
> **Swagger:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/go/grpc/grpc-sw-api.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
