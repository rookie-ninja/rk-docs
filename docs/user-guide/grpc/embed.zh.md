rk-boot 中，可以使用 embed.FS 给 rk-boot 提供 boot.yaml 等静态文件。

## 概述
目前，用户可以把 Swagger UI 参数文件, API Docs 参数文件和 boot.yaml 文件通过 embed.FS 传递给 rk-boot.

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### 2.创建并编译 protocol buffer
[使用 buf 编译 protocol buf](../buf)

### 3.创建 boot.yaml
```yaml
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
```

### 4.创建 main.go
```go
package main

import (
  "context"
  _ "embed"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-demo/api/gen/v1"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "google.golang.org/grpc"
)

//go:embed boot.yaml
var bootRaw []byte

func main() {
  boot := rkboot.NewBoot(rkboot.WithBootConfigRaw(bootRaw))

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

### 5.启动 main.go
```bash
$ go run main.go
2022-04-17T22:45:36.589+0800    INFO    boot/grpc_entry.go:965  Bootstrap grpcEntry     {"eventId": "010319cd-4a66-40db-81b1-970bb23440dd", "entryName": "greeter", "entryType": "gRPCEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T22:45:36.589583+08:00
startTime=2022-04-17T22:45:36.589326+08:00
elapsedNano=256907
timezone=CST
ids={"eventId":"010319cd-4a66-40db-81b1-970bb23440dd"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"gRPCEntry"}
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

### 6.验证
```bash
$ curl localhost:8080/v1/hello  
{"message":"hello!"}
```

## Swagger UI 与 Docs UI
我们同时还可以让 rk-boot 使用 embedFS 读取 swagger 相关的参数文件。

### 1.已存在的 Swagger JSON 文件
```bash
├── api
│   ├── gen
│   │   └── v1
│   │       ├── greeter.swagger.json
```

### 2.创建 boot.yaml
```yaml
grpc:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
    docs:
      enabled: true
```

### 3.创建 main.go
```go
package main

import (
  "context"
  "embed"
  _ "embed"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-demo/api/gen/v1"
  "github.com/rookie-ninja/rk-entry/v2/entry"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "google.golang.org/grpc"
)

//go:embed boot.yaml
var bootRaw []byte

//go:embed api/gen/v1
var docsFS embed.FS

//go:embed api/gen/v1
var swFS embed.FS

func init() {
  rkentry.GlobalAppCtx.AddEmbedFS(rkentry.SWEntryType, "greeter", &docsFS)
  rkentry.GlobalAppCtx.AddEmbedFS(rkentry.DocsEntryType, "greeter", &swFS)
}

func main() {
  boot := rkboot.NewBoot(rkboot.WithBootConfigRaw(bootRaw))

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
