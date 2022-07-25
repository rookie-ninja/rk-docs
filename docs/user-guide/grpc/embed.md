User can use embedFS to provide static files to rk-boot.

## Overview
User can provide Swagger UI config file, API Docs config file and boot.yaml file to rk-boot with embedFS.

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### 2.Create and compile protocol buffer
[Compile protobuf](buf)

### 3.Create boot.yaml
```yaml
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
```

### 4.Create main.go
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

### 5.Start main.go
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

### 6.Validate
```bash
$ curl localhost:8080/v1/hello  
{"message":"hello!"}
```

## Swagger UI & Docs UI
### 1.Swagger JSON in local FS
```bash
├── api
│   ├── gen
│   │   └── v1
│   │       ├── greeter.swagger.json
```

### 2.Create boot.yaml
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

### 3.Create main.go
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
