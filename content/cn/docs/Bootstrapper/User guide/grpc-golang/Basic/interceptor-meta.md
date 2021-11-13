---
title: "元数据拦截器"
linkTitle: "元数据拦截器"
weight: 9
description: >
  启动元数据拦截器。
---

## 概述
元数据拦截器将会把下面的信息，以 HTTP 头部的形式，返回给客户。

| Header 键 | 详情 |
| ---- | ---- |
| X-Request-Id | 拦截器会自动生成请求 ID。|
| X-[Prefix]-App | 服务名称。 |
| X-[Prefix]-App-Version | 服务版本。 |
| X-[Prefix]-App-Unix-Time | 当前服务的 Unix 时间。 |
| X-[Prefix]-Request-Received-Time | 接收到请求的时间戳。 |

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 GRPC 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 | 必要与否
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | gRPC 服务名称 | string | "", 服务不会启动 | Required |
| grpc.port | gRPC 服务端口 | integer | 0, 服务不会启动 | Required |
| grpc.enabled | gRPC 服务启动开关 ｜ bool | false | Required |
| grpc.description | gRPC 服务的描述 | string | "" | Optional |
| grpc.enableReflection | 启动 gRPC 反射功能 | boolean | false | Optional |
| grpc.enableRkGwOption | 启动 RK 自定义 Gateway Option，此 option 会默认透传所有 Header | boolean | false | Optional |
| grpc.noRecvMsgSizeLimit | 从 gRPC 服务端取消 4MB 最大接收限制 | boolean | false | Optional |
| grpc.gwMappingFilePaths | gw_mapping.yaml 文件路径，用于 RK TV | []string | [] | Optional |

## 元数据选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.meta.enabled | 启动元数据拦截器 | boolean | false |
| grpc.interceptors.meta.prefix | X-<Prefix>-XXX | string | RK |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 8080                      # Port of grpc entry
    enabled: true                   # Enable grpc entry
    commonService:
      enabled: true                 # Enable common service for testing
    interceptors:
      meta:
        enabled: true
```

### 2.创建 main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
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

### 3.验证
> 在上面的 boot.yaml 中，我们没有启动 enableRkGwOption，因此，返回的头部当中，会有 **Grpc-Metadata** 前缀。

```shell script
$ curl -vs -X GET localhost:8080/rk/v1/healthy
  ...
  < Grpc-Metadata-Content-Type: application/grpc
  < Grpc-Metadata-X-Request-Id: cb8c0c77-39cd-4e15-89d2-66ba30fa7a46
  < Grpc-Metadata-X-Rk-App-Name: rk-demo
  < Grpc-Metadata-X-Rk-App-Unix-Time: 2021-07-10T00:19:58.170222+08:00
  < Grpc-Metadata-X-Rk-App-Version: master-f414049
  < Grpc-Metadata-X-Rk-Received-Time: 2021-07-10T00:19:58.170222+08:00
  ...
  {"healthy":true}
```

> 如果我们启动 enableRkGwOption, **Grpc-Metadata** 将会消失！
> ```yaml
> ---
> grpc:
>   ...
>   enableRkGwOption: true
> ```

```shell script
$ curl -vs -X GET localhost:8080/rk/v1/healthy
  ...
  < X-Request-Id: 2ae18df6-d132-42ce-841c-571f19a88787
  < X-Rk-App-Name: rk-demo
  < X-Rk-App-Unix-Time: 2021-07-10T00:24:10.281868+08:00
  < X-Rk-App-Version: master-f414049
  < X-Rk-Received-Time: 2021-07-10T00:24:10.281868+08:00
  ...
  {"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.覆盖 requestId
```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	// Override request id
	rkgrpcctx.AddHeaderToClient(ctx, rkginctx.RequestIdKey, "request-id-override")
	// We expect new request id attached to logger
	rkgrpcctx.GetLogger(ctx).Info("Received request")

	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```

> 返回的头部中，有两个 X-Request-Id，这是因为，对于同一个 Key 进行设置，GRPC 会融合这些 Value，而非替换。
> 
> 然而，客户端中，会提取最后一个值。

```shell script
$ curl -vs -X GET "localhost:8080/v1/greeter?name=rk-dev"
...
< X-Request-Id: 4e858644-f2e2-48b2-adec-a1d329497abf
< X-Request-Id: request-id-override
< X-Rk-App-Name: rk-demo
< X-Rk-App-Unix-Time: 2021-07-10T00:29:19.030555+08:00
< X-Rk-App-Version: master-f414049
< X-Rk-Received-Time: 2021-07-10T00:29:19.030555+08:00
...
{"Message":"Hello rk-dev!"}
```

> 如果我们启动了日志拦截器，那我们会看到如下的日志。 

```shell script
2021-07-10T00:29:19.030+0800    INFO    basic/main.go:40        Received request        {"requestId": "request-id-override"}
```
```shell script
------------------------------------------------------------------------
endTime=2021-07-10T00:29:19.030673+08:00
startTime=2021-07-10T00:29:19.030535+08:00
elapsedNano=138133
timezone=CST
ids={"eventId":"request-id-override","requestId":"request-id-override"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GrpcEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"grpcMethod":"Greeter","grpcService":"api.v1.Greeter","grpcType":"unaryServer","gwMethod":"GET","gwPath":"/v1/greeter","gwScheme":"http","gwUserAgent":"curl/7.64.1"}
error={}
counters={}
pairs={}
timing={}
remoteAddr=localhost:58601
operation=/api.v1.Greeter/Greeter
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.覆盖 header prefix
```yaml
---
grpc:
  - name: greeter
    ...
    interceptors:
      meta:
        enabled: true        # Enable meta interceptor/middleware
        prefix: "Override"   # Override prefix which formed as X-[Prefix]-xxx
```
```shell script
$ curl -vs -X GET localhost:8080/rk/v1/healthy
...
< X-Override-App-Name: rk-demo
< X-Override-App-Unix-Time: 2021-07-10T00:34:59.974205+08:00
< X-Override-App-Version: master-f414049
< X-Override-Received-Time: 2021-07-10T00:34:59.974205+08:00
< X-Request-Id: 4deded12-2d39-42a5-b740-b4f74dd73c90
...
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)