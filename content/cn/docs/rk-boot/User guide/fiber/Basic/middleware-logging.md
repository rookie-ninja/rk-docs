---
title: "日志中间件"
linkTitle: "日志中间件"
weight: 6
description: >
  启动 API 请求日志中间件。
---

## 安装
```shell
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## 选项
| 名字                                       | 描述                   | 类型       | 默认值     |
|------------------------------------------|----------------------|----------|---------|
| fiber.middleware.logging.enabled           | 启动日志拦截器              | boolean  | false   |
| fiber.middleware.logging.ignore            | 局部选项，忽略 API 路径       | []string | []      |
| fiber.middleware.logging.loggerEncoding    | 日志格式：json 或者 console | string   | console |
| fiber.middleware.logging.loggerOutputPaths | 日志文件路径               | []string | stdout  |
| fiber.middleware.logging.eventEncoding     | 日志格式：json 或者 console | string   | console |
| fiber.middleware.logging.eventOutputPaths  | 日志文件路径               | []string | stdout  |

## 概念
我们需要提前了解两个概念。

1. [Event](https://github.com/rookie-ninja/rk-query)
2. [Logger](https://github.com/uber-go/zap)

### Logger
rk-boot 使用 [zap](https://github.com/uber-go/zap) 为默认日志，[lumberjack](https://github.com/natefinch/lumberjack) 为日志滚动库。

> 例子
>
> ```shell script
> 2022-05-12T09:32:21.311+0800    INFO    boot/fiber_entry.go:705 Bootstrap fiberEntry    {"eventId": "feac04f5-8ddf-48a1-89e1-d1564d6de927", "entryName": "greeter", "entryType": "FiberEntry"}
> ```

### Event
rk-boot 把每一个 RPC 请求视作 **Event**，并且使用 [rk-query](https://github.com/rookie-ninja/rk-query) 中的 Event 类型来记录日志。

| 字段 | 详情 |
| ---- | ---- |
| endTime | 结束时间 |
| startTime | 开始时间 |
| elapsedNano | Event 时间开销（Nanoseconds） |
| timezone | 时区 |
| ids | 包含 eventId, requestId 和 traceId。如果元数据拦截器被启动，或者 event.SetRequest() 被用户调用，新的 RequestId 将会被使用，同时 eventId 与 requestId 会一模一样。 如果调用链拦截器被启动，traceId 将会被记录。|
| app | 包含 [appName, appVersion](https://github.com/rookie-ninja/rk-entry#appinfoentry), entryName, entryType。 |
| env | 包含 arch, domain, hostname, localIP, os 字段。这些字段来自系统环境变量（DOMAIN）。 "*" 代表环境变量为空。|
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
> app={"appName":"rk-gin","appVersion":"master-xxx","entryName":"greeter","entryType":"FiberEntry"}
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
### 1. 创建 boot.yaml
```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      logging:
        enabled: true                                     # Optional, default: false
#        ignore: [""]                                      # Optional, default: []
#        loggerEncoding: "console"                         # Optional, default: "console"
#        loggerOutputPaths: ["logs/app.log"]               # Optional, default: ["stdout"]
#        eventEncoding: "console"                          # Optional, default: "console"
#        eventOutputPaths: ["logs/event.log"]              # Optional, default: ["stdout"]
```

### 2. 创建 main.go
```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.

package main

import (
  "context"
  "fmt"
  "github.com/gofiber/fiber/v2"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-fiber/boot"
  "net/http"
)

// @title Swagger Example API
// @version 1.0
// @description This is a sample rk-demo server.
// @termsOfService http://swagger.io/terms/

// @securityDefinitions.basic BasicAuth

// @contact.name API Support
// @contact.url http://www.swagger.io/support
// @contact.email support@swagger.io

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html
func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Register handler
  entry := rkfiber.GetFiberEntry("greeter")
  entry.App.Get("/v1/greeter", Greeter)
  // This is required!!!
  entry.RefreshFiberRoutes()

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter
// @Id 1
// @Tags Hello
// @version 1.0
// @Param name query string true "name"
// @produce application/json
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(ctx *fiber.Ctx) error {
  ctx.Response().SetStatusCode(http.StatusOK)
  return ctx.JSON(&GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3. 验证
> 发送请求

```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

> 日志

```shell script
------------------------------------------------------------------------
endTime=2022-05-12T09:32:27.487736+08:00
startTime=2022-05-12T09:32:27.487473+08:00
elapsedNano=263091
timezone=CST
ids={"eventId":"37a14783-f2e9-49da-8839-64f6c4697f55","requestId":"37a14783-f2e9-49da-8839-64f6c4697f55"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"FiberEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=127.0.0.1:64411
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 4. JSON 格式
```yaml
---
fiber:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerEncoding: "json"
        eventEncoding: "json"
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 5. 修改日志文件路径
> 修改日志文件路径之后，默认在 1GB 大小之后，进行切割，并压缩。

```yaml
---
fiber:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerOutputPaths: ["logs/app.log"]
        eventOutputPaths: ["logs/event.log"]
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 6. 获取日志实例
当每一次 RPC 请求进来的时候，中间件会把 RequestId（当启动了原数据中间件）注入到日志实例中。换句话说，每一个 RPC 请求，都会有一个新的 Logger 实例。
```go
func Greeter(ctx *fiber.Ctx) error {
    rkfiberctx.GetLogger(ctx).Info("Received request")

    ctx.Response().SetStatusCode(http.StatusOK)
	return ctx.JSON(&GreeterResponse{
        Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
    })
}
```

```shell script
2022-04-15T19:17:01.380+0800    INFO    fiber/main.go:55  Received request        {"requestId": "446c1138-1b20-48f7-a479-7b3e8a26de40"}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 7. 修改 Event
日志中间件会为每一个 RPC 请求创建一个 Event 实例。

用户可以添加 pairs，counters，errors。
```go
func Greeter(ctx *fiber.Ctx) error {
    event := rkfiberctx.GetEvent(ctx)
    event.AddPair("key", "value")

    ctx.Response().SetStatusCode(http.StatusOK)
    return ctx.JSON(&GreeterResponse{
        Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
    })
}
```

```shell script
------------------------------------------------------------------------
endTime=2022-05-12T09:42:28.482306+08:00
startTime=2022-05-12T09:42:28.48184+08:00
elapsedNano=466210
timezone=CST
ids={"eventId":"32bf4198-3958-40b3-869c-fbe621909172","requestId":"32bf4198-3958-40b3-869c-fbe621909172"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"FiberEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=127.0.0.1:64542
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
