Compile protobuf with buf.

## Overview
There will be a lot of codes needs to write if we want to use [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway).

rk-boot will help us reduce a lot of codes that doesn't need to be updated once written.

> **Steps to enable grpc-gateway:**
> 
> 1. Create gw_mapping.yaml
> 
> 2. Create buf.yaml & buf.gen.yaml
> 
> 3. Compile protobuf with gw_mapping.yaml
> 
> 4. Register generated codes into gRPC

## Prerequisite
> Install required CLI with [rk](https://github.com/rookie-ninja/rk) 

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

| tools                                                                         | Description                             | Install                                                                |
|-------------------------------------------------------------------------------|-----------------------------------------|------------------------------------------------------------------------|
| [buf](https://docs.buf.build)                                                 | Compile protocol buffer API             | [Install](https://docs.buf.build/installation)                         |
| [protoc-gen-go](https://github.com/golang/protobuf/tree/master/protoc-gen-go) | Generate .go files                      | [Install](https://grpc.io/docs/languages/go/quickstart/)               |
| [protoc-gen-go-grpc](https://github.com/grpc/grpc-go)                         | Generate gRPC related .go files         | [Install](https://grpc.io/docs/languages/go/quickstart/)               |
| [protoc-gen-grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)     | Generate grpc-gateway related .go files | [Install](https://github.com/grpc-ecosystem/grpc-gateway#installation) |
| [protoc-gen-openapiv2](https://github.com/grpc-ecosystem/grpc-gateway)        | Generate swagger related config files   | [Install](https://github.com/grpc-ecosystem/grpc-gateway#installation) |

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

### 2.Copy googleapis
rk-grpc use some data structures from googleapi package. We suggest copy those .proto files into project.

Copy files from [github/googleapi](https://github.com/googleapis/googleapis) . Or from [rk-boot](https://github.com/rookie-ninja/rk-boot/tree/main/example/web/grpc/third-party)

Copy files into third-party folder.

### 3.Create api/v1/gw_mapping.yaml
```yaml
type: google.api.Service
config_version: 3

# Please refer google.api.Http in https://github.com/googleapis/googleapis/blob/master/google/api/http.proto file for details.
http:
  rules:
    - selector: api.v1.Greeter.Greeter
      get: /api/v1/greeter
```

### 4.Create buf.yaml
```yaml
version: v1beta1
name: github.com/rk-dev/rk-demo
build:
  roots:
    - api
```

### 5.Create buf.gen.yaml
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

### 6.Compile proto file
```bash
$ buf generate --path api/v1
```

> Bellow files will be generated.
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
