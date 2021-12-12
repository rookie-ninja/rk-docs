---
title: "GRPC-golang"
linkTitle: "GRPC-golang"
weight: 2
description: >
  Create a [GRPC](https://grpc.io/docs/languages/go/quickstart/) server with RK bootstrapper.
---

## Overview
This is an example demonstrate how to configure boot.yaml file to start a gin server with bellow functionality.

> Full demo: https://github.com/rookie-ninja/rk-demo/tree/master/grpc/getting-started

- [GRPC server](https://grpc.io/docs/languages/go/quickstart/)
- [GRPC Gateway](https://github.com/grpc-ecosystem/grpc-gateway)
- [Swagger UI](https://swagger.io/tools/swagger-ui/)
- CommonService API
- RK TV Web UI
- User defined API

## Create server
### 1.Install dependency
```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-grpc
```

### 2.Create boot.yaml
By default, grpc gateway and grpc will use same port. rk-grpc will distinguish connection between grpc and http requests.

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

### 3.Create main.go
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

### 4.Start server
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

### 5.Validate
> **Swagger:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/grpc-golang/grpc-sw.png)

> **TV:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/grpc-golang/grpc-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## Add an API
We will define a Greeter API with the path of "/v1/greeter" with [protocol buffer](https://grpc.io/docs/languages/go/quickstart/). 

### 1.Install tools
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

### 2.Create greeter.proto
Create greeter.proto under **api/v1** folder

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

### 3.Create compilation config file
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

> Create **api/v1/gw_mapping.yaml**
> 
> Please refer google.api.Http in https://github.com/googleapis/googleapis/blob/master/google/api/http.proto file for details.
```yaml
type: google.api.Service
config_version: 3

http:
  rules:
    - selector: api.v1.Greeter.Greeter
      get: /api/v1/greeter
```

> **Run buf**
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

### 4.Register API
> buf will help us generate the interface of grpc service and grpc-gateway registration function.
>
> We need to implement the interface of grpc service and register it to bootstrapper with grpc-gateway by calling:
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

### 5.Validate
```shell script
$ curl "http://localhost:8080/api/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## Support Swagger UI
With above example, we already generated greeter.swagger.json file.

What we need to do is add lines in boot.yaml to make process load the files into memory.

### 1.Add path to boot.yaml
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

### 2.Validate
- **Swagger:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/grpc-golang/grpc-sw-api.png)

- **TV:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/grpc-golang/grpc-tv-api.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
