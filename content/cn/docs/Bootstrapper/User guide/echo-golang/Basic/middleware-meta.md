---
title: "元数据拦截器"
linkTitle: "元数据拦截器"
weight: 8
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
> 启动器包含了如下通用选项，这些选项是启动 Echo 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.name | Echo 服务名称 | string | N/A |
| echo.port | Echo 服务端口 | integer | nil, 服务不会启动 |
| echo.enabled | Echo 服务启动开关 | bool | false |
| echo.description | Echo 服务的描述 | string | "" |

## 元数据选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.interceptors.meta.enabled | 启动元数据拦截器 | boolean | false |
| echo.interceptors.meta.prefix | X-<Prefix>-XXX | string | RK |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true          # Enable common service for testing
    interceptors:
      meta:
        enabled: true        # Enable meta interceptor/middleware
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
$ curl -vs -X GET localhost:8080/rk/v1/healthy
  ...
  < X-Request-Id: c2ec237e-e141-45af-a4ff-89ce8a9d0636
  < X-Rk-App-Name: rk-demo
  < X-Rk-App-Unix-Time: 2021-07-06T02:46:31.977902+08:00
  < X-Rk-App-Version: master-f414049
  < X-Rk-Received-Time: 2021-07-06T02:46:31.977902+08:00
  ...
  {"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.覆盖 requestId
```go
func Greeter(ctx echo.Context) error {
    // Override request id
	rkechoctx.SetHeaderToClient(ctx, rkechoctx.RequestIdKey, "request-id-override")
    // We expect new request id attached to logger
	rkechoctx.GetLogger(ctx).Info("Received request")

	return ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
	})
}
```

```shell script
$ curl -vs -X GET "localhost:8080/v1/greeter?name=rk-dev"
...
< X-Request-Id: request-id-override
< X-Rk-App-Name: rk-demo
< X-Rk-App-Unix-Time: 2021-07-06T02:49:34.27756+08:00
< X-Rk-App-Version: master-f414049
< X-Rk-Received-Time: 2021-07-06T02:49:34.27756+08:00
...
{"Message":"Hello rk-dev!"}
```

> 如果我们启动了日志拦截器，那我们会看到如下的日志。 

```shell script
2021-07-06T02:49:34.277+0800    INFO    basic/main.go:41        Received request        {"requestId": "request-id-override"}
```
```shell script
------------------------------------------------------------------------
endTime=2021-07-06T02:49:34.27773+08:00
startTime=2021-07-06T02:49:34.277544+08:00
elapsedNano=185412
timezone=CST
ids={"eventId":"request-id-override","requestId":"request-id-override"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"EchoEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
error={}
counters={}
pairs={}
timing={}
remoteAddr=localhost:57600
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.覆盖 header prefix
```yaml
---
echo:
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
< X-Override-App-Unix-Time: 2021-07-06T02:52:53.035941+08:00
< X-Override-App-Version: master-f414049
< X-Override-Received-Time: 2021-07-06T02:52:53.035941+08:00
< X-Request-Id: c3097361-1833-4bba-9867-e8d1f2fb2207
...
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)