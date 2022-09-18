---
title: "监控中间件"
linkTitle: "监控中间件"
weight: 7
description: >
  启动 Prometheus 中间件。
---

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## 选项
| 名字                          | 描述                | 类型       | 默认值   |
|-----------------------------|-------------------|----------|-------|
| grpc.middleware.prom.enabled | 启动 Prometheus 中间件 | boolean  | false |
| grpc.middleware.prom.ignore  | 局部选项，忽略 API 路径    | []string | []    |

## 概念
Prometheus 中间件会默认记录如下监控。

| 监控项         | 数据类型    | 详情             |
|-------------|---------|----------------|
| elapsedNano | Summary | RPC 耗时         |
| resCode     | Counter | 基于 RPC 返回码的计数器 |

上述三项监控，都有如下的标签。

| 标签         | 详情                                                                                                |
|------------|---------------------------------------------------------------------------------------------------|
| entryName  | Entry 名称                                                                                          |
| entryType  | Entry 类型                                                                                          |
| domain     | 环境变量: DOMAIN, eg: prod                                                                            |
| instance   | 本地 Hostname                                                                                       |
| restMethod | eg: GET                                                                                           |
| restPath   | eg: /rk/v1/alive                                                                                  |
| resCode    | 返回码, eg: 200                                                                                      |

例子

```bash
rk_prom_elapsedNano{domain="*",entryName="greeter",entryType="gRPCEntry",grpcMethod="Hello",grpcService="api.v1.Greeter",grpcType="UnaryServer",instance="lark.local",resCode="OK",restMethod="",restPath="",quantile="0.5"} 11570
...
rk_prom_resCode{domain="*",entryName="greeter",entryType="gRPCEntry",grpcMethod="Hello",grpcService="api.v1.Greeter",grpcType="UnaryServer",instance="lark.local",resCode="OK",restMethod="",restPath=""} 1```
```

## 快速开始
### 1.创建并编译 protocol buffer
[使用 buf 编译 protocol buf](../buf)

### 2.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    prom:
      enabled : true
    middleware:
      prom:
        enabled: true
#        ignore: [""]
```

### 3.创建 main.go
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

### 4.验证
> 发送请求

```bash
$ curl localhost:8080/v1/hello
{"message":"hello!"}
```

> Prometheus 客户端:
>
> http://localhost:8080/metrics

![prom-inter](../../../../img/user-guide/gin/basic/gin-prom-inter.png)

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)
