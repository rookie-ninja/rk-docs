---
title: "Fiber 框架"
linkTitle: "Fiber 框架"
weight: 7
description: >
  通过 rk-boot，配合 rk-fiber 插件，创建 [gofiber/fiber](https://github.com/gofiber/fiber) 后台服务。
---

## 概述
我们将会使用 rk-boot 启动 [gofiber/fiber](https://github.com/gofiber/fiber) 微服务，并且添加 /v1/greeter API。

为了微服务的完整性，我们还会开启如下几个附加功能。

| 功能             | 介绍                                  |
|:---------------|:------------------------------------|
| Swagger UI     | 开启 Swagger UI                       |
| API Docs UI    | 开启 RapiDoc UI                       |
| Prometheus 客户端 | 开启 Prometheus 本地客户端                 |
| 日志中间件          | 针对每个 API 请求，自动记录日志                  |
| Prometheus 中间件 | 针对每个 API 请求，自动记录 Prometheus 记录      |
| 原数据中间件         | 针对每个 API 请求，自动添加 RequestID 到 Header |

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## 1. 创建 boot.yaml
```yaml
fiber:
  - name: greeter
    port: 8080                    # 监听端口
    enabled: true                 # 开启微服务
    sw:
      enabled: true               # 开启 Swagger UI，默认路径为 /sw
    docs:
      enabled: true               # 开启 API Doc UI，默认路径为 /docs
    prom:
      enabled: true               # 开启 Prometheus 客户端，默认路径为 /metrics
    middleware:
      logging:
        enabled: true             # 开启 API 日志中间件
      prom:
        enabled: true             # 开启 API Prometheus 中间件
      meta:
        enabled: true             # 开启 API 原数据中间件，自动生成 RequestID
```

## 2. 创建 main.go
为了能给 [gofiber/fiber](https://github.com/gofiber/fiber) 提供 Swagger UI，我们需要在代码中添加一系列注视，然后通过 [swag](https://github.com/swaggo/swag) 命令行生成 swagger.json 文件。

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

## 3.生成 swagger.json

```shell
$ swag init
```

## 4.文件夹结构
```shell script
$ tree
.
├── boot.yaml
├── docs
│   ├── swagger.json
│   └── swagger.yaml
├── go.mod
├── go.sum
└── main.go
```

## 5.运行 main.go
```go
$ go run main.go
2022-05-12T09:31:45.405+0800    INFO    boot/fiber_entry.go:705 Bootstrap fiberEntry    {"eventId": "d43cfb13-e7ee-420a-9143-545b016723f4", "entryName": "greeter", "entryType": "FiberEntry"}
2022-05-12T09:31:45.407+0800    INFO    boot/fiber_entry.go:417 SwaggerEntry: http://localhost:8080/sw/
2022-05-12T09:31:45.407+0800    INFO    boot/fiber_entry.go:420 DocsEntry: http://localhost:8080/docs/
2022-05-12T09:31:45.407+0800    INFO    boot/fiber_entry.go:423 PromEntry: http://localhost:8080/metrics
------------------------------------------------------------------------
endTime=2022-05-12T09:31:45.407095+08:00
startTime=2022-05-12T09:31:45.405534+08:00
elapsedNano=1560417
timezone=CST
ids={"eventId":"d43cfb13-e7ee-420a-9143-545b016723f4"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"FiberEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"docsEnabled":true,"docsPath":"/docs/","fiberPort":8080,"promEnabled":true,"promPath":"/metrics","promPort":8080,"swEnabled":true,"swPath":"/sw/"}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

## 6.验证
### 6.1 Swagger UI
[http://localhost:8080/sw/](http://localhost:8080/sw/)

![](/rk-boot/example/sw.png)

### 6.2 API Docs UI
我们使用了 RapiDocs 作为 API Docs UI。其实 RapiDocs 也可以替换 Swagger UI 测试 API，后续我们会考虑替换 Swagger UI。

[http://localhost:8080/docs/](http://localhost:8080/docs/)

![](/rk-boot/example/docs.png)

### 6.3 Prometheus 客户端
[http://localhost:8080/metrics](http://localhost:8080/metrics)

![](/rk-boot/example/metrics.png)

### 6.4 发送请求
```shell
$ curl -vs "localhost:8080/v1/greeter?name=rk-dev"
* ...
< X-Request-Id: d95d3b00-0d10-4784-83bf-bf392cf8bd77
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-14T17:04:39.458193+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-14T17:04:39.458193+08:00
< ...
{"Message":"Hello rk-dev!"}
```

### 6.5 验证 API 日志
rk-boot 默认会使用如下格式打印 API 日志，也可以使用 JSON 格式，请参考用户指南。

```shell
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

### 6.6 验证 Prometheus Metrics
访问 [http://localhost:8080/metrics](http://localhost:8080/metrics)

![](/rk-boot/example/api-metrics-gin.png)

## _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
