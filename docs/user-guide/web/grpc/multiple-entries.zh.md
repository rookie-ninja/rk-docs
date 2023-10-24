如何在 rk-boot 中，启动多个服务？

## 概述
通过 rk-boot，用户可以在一个进程里面，启动多个 Entry。

用户还可以同时启动 Gin 服务和 gRPC 等服务。

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
  - name: alice
    port: 8080
#   gwPort: 8081                  # 可选项，如果不指定，会使用与 port 一样的端口
    enabled: true
  - name: bob
    port: 8081
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
  alice := rkgrpc.GetGrpcEntry("alice")
  alice.AddRegFuncGrpc(registerGreeter)
  alice.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

  bob := rkgrpc.GetGrpcEntry("bob")
  bob.AddRegFuncGrpc(registerGreeter)
  bob.AddRegFuncGw(greeter.RegisterGreeterHandlerFromEndpoint)

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

### 5.验证
```bash
$ curl localhost:8080/v1/hello
{"message":"hello!"}

$ curl localhost:8081/v1/hello
{"message":"hello!"}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)