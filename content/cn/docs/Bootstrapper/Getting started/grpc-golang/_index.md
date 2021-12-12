---
title: "GRPC-golang"
linkTitle: "GRPC-golang"
weight: 2
description: >
  通过启动器，创建基于 [GRPC](https://grpc.io/docs/languages/go/quickstart/) 框架的服务。
---

## 概述
让我们通过编辑 boot.yaml 文件来创建基于 GRPC 框架的服务。

> 例子: https://github.com/rookie-ninja/rk-demo/tree/master/grpc/getting-started

- [GRPC 服务](https://grpc.io/docs/languages/go/quickstart/)
- [GRPC Gateway](https://github.com/grpc-ecosystem/grpc-gateway)
- [Swagger 界面](https://swagger.io/tools/swagger-ui/)
- 通用 API
- RK TV Web 界面
- 用户自定义 API

## 创建服务
### 1.安装
```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-grpc
```

### 2.创建 boot.yaml
grpc 与 grpc-gateway 会默认使用同一个端口，rk-grpc 将会根据链接类型，自动判断 http 请求和 grpc 请求。

```yaml
---
grpc:
  - name: greeter            # Required, Name of grpc entry
    port: 8080               # Required, Port of grpc entry
    enabled: true            # Required, Enable grpc entry
    enableReflection: true   # Optional, Enable grpc server reflection, https://github.com/grpc/grpc/blob/master/doc/server-reflection.md
    enableRkGwOption: true   # Optional, Enable grpc gateway server option as RK style
    commonService:
      enabled: true          # Optional, Enable common service
    tv:
      enabled: true          # Optional, Enable RK TV
    sw:
      enabled: true          # Optional, Enable Swagger UI
```

### 3.创建 main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-grpc/boot"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 4.启动服务
```go
$ go run main.go
...
2021-09-11T04:43:23.219+0800    INFO    boot/grpc_entry.go:829  Bootstrapping grpcEntry.        {"eventId": "2df7fa50-3d1f-44cb-9db6-c4c26d9a29ce", "entryName": "greeter", "entryType": "GrpcEntry", "port": 8080, "swEnabled": true, "tvEnabled": true, "promEnabled": true, "commonServiceEnabled": true, "tlsEnabled": false, "reflectionEnabled": true, "swPath": "/sw/", "headers": {}, "tvPath": "/rk/v1/tv"}
------------------------------------------------------------------------
endTime=2021-09-11T04:43:23.217156+08:00
startTime=2021-09-11T04:43:23.217145+08:00
elapsedNano=11496
timezone=CST
ids={"eventId":"2df7fa50-3d1f-44cb-9db6-c4c26d9a29ce"}
app={"appName":"rk-grpc","appVersion":"master-bd63f29","entryName":"greeter","entryType":"GrpcPromEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.6","os":"darwin","realm":"*","region":"*"}
payloads={"entryName":"greeter","entryType":"GrpcPromEntry","path":"/metrics","port":8080}
error={}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=bootstrap
resCode=OK
eventStatus=Ended
EOE
```

### 5.验证
> **Swagger 界面:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/grpc-golang/grpc-sw.png)

> **TV 界面:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/grpc-golang/grpc-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## 添加一个 API
我们将会通过 [protocol buffer](https://grpc.io/docs/languages/go/quickstart/) 添加 "/v1/greeter" API。 

### 1.安装工具
> 编译 [protocol buffer](https://grpc.io/docs/languages/go/quickstart/) 的过程较为复杂，我们需要一系列工具的支持。
> 
> 我们通过 [rk](https://github.com/rookie-ninja/rk) 来快速安装所需要的工具。
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

### 2.创建 greeter.proto
在 **api/v1** 文件夹中创建 greeter.proto 文件。

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

### 3.创建 buf 参数文件
> Create **buf.yaml**
```yaml
version: v1beta1
name: github.com/rk-dev/rk-boot
build:
  roots:
    - api
```

> Create **buf.gen.yaml**
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
  - name: grpc-gateway
    out: api/gen
    opt:
      - paths=source_relative
      - grpc_api_configuration=api/v1/gw_mapping.yaml
  - name: openapiv2
    out: api/gen
    opt:
      - grpc_api_configuration=api/v1/gw_mapping.yaml
```

> 创建 **api/v1/gw_mapping.yaml**
> 
> 请参考 https://github.com/googleapis/googleapis/blob/master/google/api/http.proto
```yaml
type: google.api.Service
config_version: 3

http:
  rules:
    - selector: api.v1.Greeter.Greeter
      get: /api/v1/greeter
```

> **执行 buf**
```shell script
$ buf generate
# By configuration in buf.gen.yaml, generated files would be write to api/gen folder
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
```

### 4.注册 API
> buf 命令行会帮助我们创建 grpc 和 grpc-gateway 接口函数。
>
> 我们需要实现 grpc 和 grpc-gateway 接口函数，并且通过下面的函数来注册我们的实现。
> 
> - AddRegFuncGrpc
> - AddRegFuncGw
```go
package main

import (
	"context"
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-demo/api/gen/v1"
	"github.com/rookie-ninja/rk-grpc/boot"
	"google.golang.org/grpc"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Get grpc entry with name
	grpcEntry := boot.GetEntry("greeter").(*rkgrpc.GrpcEntry)
	grpcEntry.AddRegFuncGrpc(registerGreeter)
	grpcEntry.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

func registerGreeter(server *grpc.Server) {
	greeter.RegisterGreeterServer(server, &GreeterServer{})
}

type GreeterServer struct{}

func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```

### 5.验证
```shell script
$ curl "http://localhost:8080/api/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## 支持 Swagger 界面
通过上面的例子，我们已经生成了 greeter.swagger.json 文件。

下一步，我们需要做的就是在 boot.yaml 文件中指出 greeter.swagger.json 文件和 http->grpc 的路径映射文件的路径。

### 1.在 boot.yaml 中添加 Swagger 参数文件路径
```yaml
---
grpc:
  - name: greeter
    ...
    gwMappingFilePaths:
      - "api/v1/gw_mapping.yaml"  # Boot will look for gateway mapping files and load information into memory
    sw:
      enabled: true
      jsonPath: "api/gen/v1"      # Boot will look for swagger config files from this folder
```

### 2.验证
- **Swagger 界面:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/grpc-golang/grpc-sw-api.png)

- **TV 界面:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/grpc-golang/grpc-tv-api.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
