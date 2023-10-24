当使用 rk-boot 的时候，如何通过 flag 覆盖 boot.yaml 里的参数？

## 概述
rk-boot 支持通过命令行和环境变量参数来覆盖 boot.yaml 里的值。

- 覆盖 boot.yaml 里的参数 (通过 \-\-rkset)

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
---
grpc:
  - name: greeter
    port: 8080
#   gwPort: 8081                  # 可选项，如果不指定，会使用与 port 一样的端口
    enabled: true
```

### 4.创建 main.go
```go
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

### 5.通过 flag 覆盖 Port
如果是 List，请使用【grpc[0].port】格式访问。

```bash
$ go run main.go --rkset grpc[0].port=8081
2022-04-17T23:04:53.960+0800    INFO    entry/util.go:332       Found flag to override, applying...     {"flags": ["grpc[0].port=8081"]}
2022-04-17T23:04:53.960+0800    INFO    boot/grpc_entry.go:965  Bootstrap grpcEntry     {"eventId": "df4a9ff3-6537-433e-8718-bb07880070c4", "entryName": "greeter", "entryType": "gRPCEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T23:04:53.961165+08:00
startTime=2022-04-17T23:04:53.960979+08:00
elapsedNano=185688
timezone=CST
ids={"eventId":"df4a9ff3-6537-433e-8718-bb07880070c4"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"grpcPort":8081,"gwPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

> 发送请求到：8081
```bash
$ curl localhost:8081/v1/hello
{"message":"hello!"}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 6.通过环境变量覆盖 Port
覆盖 boot.yaml 里参数的时候，需要带上 【RK_】 作为前缀，此外，如果是 List 使用 【_X_】 访问。

例如：RK_GRPC_0_PORT

```go
package main

import (
  "context"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-demo/api/gen/v1"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "google.golang.org/grpc"
  "os"
)

func main() {
  os.Setenv("RK_GRPC_0_PORT", "8081")

  boot := rkboot.NewBoot()

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

```bash
$ go run main.go
2022-04-17T23:05:49.493+0800    INFO    entry/util.go:299       Found ENV to override, applying...      {"env": ["RK_GRPC_0_PORT=8081 => grpc[0].port=8081"]}
2022-04-17T23:05:49.493+0800    INFO    boot/grpc_entry.go:965  Bootstrap grpcEntry     {"eventId": "fa9dba2d-ec47-4720-ae89-537b61063013", "entryName": "greeter", "entryType": "gRPCEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T23:05:49.493936+08:00
startTime=2022-04-17T23:05:49.49368+08:00
elapsedNano=256504
timezone=CST
ids={"eventId":"fa9dba2d-ec47-4720-ae89-537b61063013"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"gRPCEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"grpcPort":8081,"gwPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```