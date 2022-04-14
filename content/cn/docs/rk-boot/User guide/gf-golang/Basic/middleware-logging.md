---
title: "日志拦截器"
linkTitle: "日志拦截器"
weight: 6
description: >
  启动 RPC 请求的日志拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-gf
```
## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 GoFrame 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gf.name | GoFrame 服务名称 | string | N/A |
| gf.port | GoFrame 服务端口 | integer | nil, 服务不会启动 |
| gf.enabled | GoFrame 服务启动开关 | bool | false |
| gf.description | GoFrame 服务的描述 | string | "" |

## 日志拦截器选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gf.interceptors.loggingZap.enabled | 启动日志拦截器 | boolean | false |
| gf.interceptors.loggingZap.zapLoggerEncoding | 日志格式：json 或者 console | string | console |
| gf.interceptors.loggingZap.zapLoggerOutputPaths | 日志文件路径 | []string | stdout |
| gf.interceptors.loggingZap.eventLoggerEncoding | 日志格式：json 或者 console | string | console |
| gf.interceptors.loggingZap.eventLoggerOutputPaths | 日志文件路径 | []string | false |

## 概念
我们需要提前了解两个概念。

1. [EventLogger](https://github.com/rookie-ninja/rk-query) 
2. [ZapLogger](https://github.com/uber-go/zap)

### ZapLogger
RK 启动器使用 [zap](https://github.com/uber-go/zap) 为默认日志，[lumberjack](https://github.com/natefinch/lumberjack) 为日志滚动库。

> 例子
> 
> ```shell script
> 2021-12-06T15:16:15.366+0800    INFO    boot/gf_entry.go:848    Bootstrapping GfEntry.  {"eventId": "abec7c4f-7678-45a2-8824-df315864eea0", "entryName": "greeter", "entryType": "GfEntry", "port": 8080}
> ```

### EventLogger
RK 启动器把每一个 RPC 视作 **Event**，并且使用 [rk-query](https://github.com/rookie-ninja/rk-query) 中的 Event 类型来记录日志。

| 字段 | 详情 |
| ---- | ---- |
| endTime | 结束时间 |
| startTime | 开始时间 |
| elapsedNano | Event 时间开销（Nanoseconds） |
| timezone | 时区 |
| ids | 包含 eventId, requestId 和 traceId。如果元数据拦截器被启动，或者 event.SetRequest() 被用户调用，新的 RequestId 将会被使用，同时 eventId 与 requestId 会一模一样。 如果调用链拦截器被启动，traceId 将会被记录。|
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
> endTime=2021-06-25T01:30:45.144023+08:00
> startTime=2021-06-25T01:30:45.143767+08:00
> elapsedNano=255948
> timezone=CST
> ids={"eventId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","requestId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","traceId":"65b9aa7a9705268bba492fdf4a0e5652"}
> app={"appName":"rk-gf","appVersion":"master-xxx","entryName":"greeter","entryType":"GfEntry"}
> env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
> payloads={"apiMethod":"GET","apiPath":"/rk/v1/healthy","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
> error={}
> counters={}
> pairs={}
> timing={}
> remoteAddr=localhost:60718
> operation=/rk/v1/healthy
> resCode=200
> eventStatus=Ended
> EOE
> ```

## 快速开始
### 1.创建 boot.yaml
```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true          # Enable common service for testing
    interceptors:
      loggingZap:
        enabled: true        # Enable logging interceptor/middleware, by default, stdout would be destination of log
```

### 2.创建 main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
    _ "github.com/rookie-ninja/rk-gf/boot"
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
endTime=2021-07-05T23:42:35.588164+08:00
startTime=2021-07-05T23:42:35.588095+08:00
elapsedNano=69414
timezone=CST
ids={"eventId":"9b874eea-b16b-4c46-b0f5-d2b7cff6844e"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GfEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"apiMethod":"GET","apiPath":"/rk/v1/healthy","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
error={}
counters={}
pairs={}
timing={}
remoteAddr=localhost:56274
operation=/rk/v1/healthy
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.JSON 格式
```yaml
---
gf:
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
gf:
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
根据 zap logger，当每一次 RPC 请求进来的时候，拦截器会把 RequestId（当启动了元数据拦截器）注入到日志实例中。换句话说，每一个 RPC 请求，都会有一个新的 Logger 实例。
```go
func Greeter(ctx *ghttp.Request) {
	rkgfctx.GetLogger(ctx).Info("Received request")

	ctx.Response.WriteHeader(http.StatusOK)
	err := ctx.Response.WriteJson(&GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name")),
	})

	if err != nil {
		panic(err)
	}
}
```
```shell script
2021-07-06T02:04:55.306+0800    INFO    basic/main.go:43        Received request        {"requestId": "b8522178-9fac-47d3-b866-ce43e119f7a3"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 6.修改 Event
日志拦截器会为每一个 RPC 请求创建一个 Event 实例。

用户可以添加 pairs，counters，errors。
```go
func Greeter(ctx *ghttp.Request) {
	event := rkgfctx.GetEvent(ctx)
	event.AddPair("key", "value")

	ctx.Response.WriteHeader(http.StatusOK)
	err := ctx.Response.WriteJson(&GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name")),
	})

	if err != nil {
		panic(err)
	}
}
```

```shell script
------------------------------------------------------------------------
endTime=2021-12-06T15:26:53.364949+08:00
startTime=2021-12-06T15:26:53.364839+08:00
elapsedNano=110410
timezone=CST
ids={"eventId":"1271b868-b875-40f7-9b74-e0013675acc1"}
app={"appName":"rk-demo","appVersion":"master-878d8ab","entryName":"greeter","entryType":"GfEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
error={}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=localhost:63073
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
