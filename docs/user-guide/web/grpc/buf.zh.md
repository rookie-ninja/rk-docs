使用 buf 编译 protocol buf。

## 概述
如果想要使用 [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) ，并且提供企业级别的服务，我们需要写很多的代码。

我们可以通过启动器，通过编辑 boot.yaml 文件，快速启动 grpc-gateway。

> **启动 grpc-gateway 步骤:**
> 
> 1. 创建 gw_mapping.yaml
> 
> 2. 创建 buf.yaml & buf.gen.yaml
> 
> 3. 基于 gw_mapping.yaml 文件，编译 protocol buffer
> 
> 4. 把生成出来的 gateway 函数注册到 gRPC 服务中

## 先决条件
安装所需的工具
> 建议通过 [rk](https://github.com/rookie-ninja/rk) 命令行，快速安装所需要的工具。

```bash
# Install RK CLI
$ go get github.com/rookie-ninja/rk/cmd/rk

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

### 2.复制 googleapis
rk-grpc 插件用到了一些 googleapi 里面的基础结构。我们推荐在项目工程中，存放这些文件。

可以直接到 [github/googleapi](https://github.com/googleapis/googleapis) 复制相应的文件，也可以从 [rk-boot](https://github.com/rookie-ninja/rk-boot/tree/main/example/web/grpc/third-party) 的例子中，复制相应的文件。

把如下文件复制到 third-party 文件夹中。

### 3.创建 api/v1/gw_mapping.yaml
```yaml
type: google.api.Service
config_version: 3

# Please refer google.api.Http in https://github.com/googleapis/googleapis/blob/master/google/api/http.proto file for details.
http:
  rules:
    - selector: api.v1.Greeter.Greeter
      get: /api/v1/greeter
```

### 4.创建 buf.yaml
```yaml
version: v1beta1
name: github.com/rk-dev/rk-demo
build:
  roots:
    - api
```

### 5.创建 buf.gen.yaml
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

### 6.编译 proto file
```bash
$ buf generate --path api/v1
```

> 如下的文件会被创建。
```bash
$ tree api/gen 
api/gen
└── v1
    ├── greeter.pb.go
    ├── greeter.pb.gw.go
    ├── greeter.swagger.json
    └── greeter_grpc.pb.go
 
1 directory, 4 files
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
