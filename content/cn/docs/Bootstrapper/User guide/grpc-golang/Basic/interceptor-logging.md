---
title: "日志拦截器"
linkTitle: "日志拦截器"
weight: 7
description: >
  启动 RPC 请求的日志拦截器。
---

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

## 日志拦截器选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.loggingZap.enabled | 启动日志拦截器 | boolean | false |
| grpc.interceptors.loggingZap.zapLoggerEncoding | 日志格式：json 或者 console | string | console |
| grpc.interceptors.loggingZap.zapLoggerOutputPaths | 日志文件路径 | []string | stdout |
| grpc.interceptors.loggingZap.eventLoggerEncoding | 日志格式：json 或者 console | string | console |
| grpc.interceptors.loggingZap.eventLoggerOutputPaths | 日志文件路径 | []string | stdout |

## 概念
我们需要提前了解两个概念。

1. [EventLogger](https://github.com/rookie-ninja/rk-query) 
2. [ZapLogger](https://github.com/uber-go/zap)

### ZapLogger
RK 启动器使用 [zap](https://github.com/uber-go/zap) 为默认日志，[lumberjack](https://github.com/natefinch/lumberjack) 为日志滚动库。

> 例子
> 
> ```shell script
> 2021-07-09T23:52:13.667+0800    INFO    boot/grpc_entry.go:694  Bootstrapping grpcEntry.        {"eventId": "9bc192fb-567c-45d4-8775-7a097b0dab04", "entryName": "greeter", "entryType": "GrpcEntry", "grpcPort": 8080, "commonServiceEnabled": true, "tlsEnabled": false, "gwEnabled": true, "reflectionEnabled": false, "swEnabled": false, "tvEnabled": false, "promEnabled": false, "gwClientTlsEnabled": false, "gwServerTlsEnabled": false}
> ```

### EventLogger
RK 启动器把每一个 RPC 视作 **Event**，并且使用 [rk-query](https://github.com/rookie-ninja/rk-query) 中的 Event 类型来记录日志。

| 字段 | 详情 |
| ---- | ---- |
| endTime | 结束时间 |
| startTime | 开始时间 |
| elapsedNano | Event 时间开销（Nanoseconds） |
| timezone | 时区 |
| ids | 包含 eventId, requestId 和 traceId。如果原数据拦截器被启动，或者 event.SetRequest() 被用户调用，新的 RequestId 将会被使用，同时 eventId 与 requestId 会一模一样。 如果调用链拦截器被启动，traceId 将会被记录。|
| app | 包含 [appName, appVersion](https://github.com/rookie-ninja/rk-entry#appinfoentry), entryName, entryType。 |
| env | 包含 arch, az, domain, hostname, localIP, os, realm, region. realm, region, az, domain 字段。这些字段来自系统环境变量（REALM，REGION，AZ，DOMAIN）。 "*" 代表环境变量为空。|
| payloads | 包含 RPC 相关信息。 |
| error | 包含错误。|
| counters | 通过 event.SetCounter() 来操作。|
| pairs | 通过 event.AddPair() 来操作。 |
| timing | 通过 event.StartTimer() 和 event.EndTimer() 来操作。 |
| remoteAddr | RPC 远程地址。 |
| operation | RPC 名字。 |
| resCode | RPC 返回码。 |
| eventStatus | Ended 或者 InProgress |

> 例子
> 
> ```shell script
> ------------------------------------------------------------------------
> endTime=2021-07-09T23:44:09.81483+08:00
> startTime=2021-07-09T23:44:09.814784+08:00
> elapsedNano=46065
> timezone=CST
> ids={"eventId":"67d64dab-f3ea-4b77-93d0-6782caf4cfee"}
> app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GrpcEntry"}
> env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
> payloads={"grpcMethod":"Healthy","grpcService":"rk.api.v1.RkCommonService","grpcType":"unaryServer","gwMethod":"","gwPath":"","gwScheme":"","gwUserAgent":""}
> error={}
> counters={}
> pairs={"healthy":"true"}
> timing={}
> remoteAddr=localhost:58205
> operation=/rk.api.v1.RkCommonService/Healthy
> resCode=OK
> eventStatus=Ended
> EOE
> ```

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
      loggingZap:
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
> 发送请求

```shell script
$ curl -X GET localhost:8080/rk/v1/healthy
{"healthy":true}
```

> 日志

```shell script
------------------------------------------------------------------------
endTime=2021-07-09T23:44:09.81483+08:00
startTime=2021-07-09T23:44:09.814784+08:00
elapsedNano=46065
timezone=CST
ids={"eventId":"67d64dab-f3ea-4b77-93d0-6782caf4cfee"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GrpcEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"grpcMethod":"Healthy","grpcService":"rk.api.v1.RkCommonService","grpcType":"unaryServer","gwMethod":"","gwPath":"","gwScheme":"","gwUserAgent":""}
error={}
counters={}
pairs={"healthy":"true"}
timing={}
remoteAddr=localhost:58205
operation=/rk.api.v1.RkCommonService/Healthy
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.JSON 格式
```yaml
---
grpc:
  - name: greeter
    ...
    interceptors:
      loggingZap:
        ...
        zapLoggerEncoding: "json"          # Override to json format, option: json or console
        eventLoggerEncoding: "json"        # Override to json format, option: json or console
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.修改日志文件路径
> 修改日志文件路径之后，默认在 1GB 大小之后，进行切割，并压缩。

```yaml
---
grpc:
  - name: greeter
    ...
    interceptors:
      loggingZap:
        ...
        zapLoggerOutputPaths: ["logs/app.log"]        # Override output paths, option: json or console
        eventLoggerOutputPaths: ["logs/event.log"]    # Override output paths, option: json or console
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.获取 RPC 日志实例
根据 zap logger，当每一次 RPC 请求进来的时候，拦截器会把 RequestId（当启动了原数据拦截器）注入到日志实例中。换句话说，每一个 RPC 请求，都会有一个新的 Logger 实例。
```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	rkgrpcctx.GetLogger(ctx).Info("Received request")

	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```
```shell script
2021-07-09T23:50:39.318+0800    INFO    basic/main.go:36        Received request        {"requestId": "c33698f2-3071-48d4-9d92-b1aa311e6c06"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 6.修改 Event
日志拦截器会为每一个 RPC 请求创建一个 Event 实例。

用户可以添加 pairs，counters，errors。
```go
func (server *GreeterServer) Greeter(ctx context.Context, request *greeter.GreeterRequest) (*greeter.GreeterResponse, error) {
	event := rkgrpcctx.GetEvent(ctx)
	event.AddPair("key", "value")

	return &greeter.GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", request.Name),
	}, nil
}
```
```shell script
------------------------------------------------------------------------
endTime=2021-07-09T23:52:39.103351+08:00
startTime=2021-07-09T23:52:39.10332+08:00
elapsedNano=31154
timezone=CST
ids={"eventId":"92001951-80c1-4dda-8f14-f920834f5c61","requestId":"92001951-80c1-4dda-8f14-f920834f5c61"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GrpcEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"grpcMethod":"Greeter","grpcService":"api.v1.Greeter","grpcType":"unaryServer","gwMethod":"","gwPath":"","gwScheme":"","gwUserAgent":""}
error={}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=localhost:58269
operation=/api.v1.Greeter/Greeter
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)