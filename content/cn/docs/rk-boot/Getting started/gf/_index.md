---
title: "GoFrame 框架"
linkTitle: "GoFrame 框架"
weight: 4
description: >
  通过 rk-boot，配合 rk-gf 插件，创建 [gogf/gf](https://github.com/gogf/gf) 后台服务。
---

## 概述
我们将会使用 rk-boot 启动 [gogf/gf](https://github.com/gogf/gf) 微服务，并且添加 /v1/greeter API。

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
go get github.com/rookie-ninja/rk-gf
```

## 1. 创建 boot.yaml
```yaml
gf:
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
为了能给 [gogf/gf](https://github.com/gogf/gf) 提供 Swagger UI，我们需要在代码中添加一系列注视，然后通过 [swag](https://github.com/swaggo/swag) 命令行生成 swagger.json 文件。

```go
package main

import (
  "context"
  "fmt"
  "github.com/gogf/gf/v2/net/ghttp"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gf/boot"
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
  boot := rkboot.NewBoot()

  // 注册 API
  entry := rkgf.GetGfEntry("greeter")
  entry.Server.BindHandler("/v1/greeter", Greeter)

  // 启动
  boot.Bootstrap(context.TODO())

  // 等待关闭信号
  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(ctx *ghttp.Request) {
  ctx.Response.WriteHeader(http.StatusOK)
  ctx.Response.WriteJson(&GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name").String()),
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
2022-04-15T03:36:22.322+0800    INFO    boot/gf_entry.go:631    Bootstrap gfEntry       {"eventId": "b506d506-a0c7-4a90-b422-96a18633fe65", "entryName": "greeter", "entryType": "GoFrameEntry"}
2022-04-15T03:36:22.323+0800    INFO    boot/gf_entry.go:396    SwaggerEntry: http://localhost:8080/sw/
2022-04-15T03:36:22.324+0800    INFO    boot/gf_entry.go:399    DocsEntry: http://localhost:8080/docs/
2022-04-15T03:36:22.324+0800    INFO    boot/gf_entry.go:402    PromEntry: http://localhost:8080/metrics
------------------------------------------------------------------------
endTime=2022-04-15T03:36:22.324017+08:00
startTime=2022-04-15T03:36:22.322457+08:00
elapsedNano=1560663
timezone=CST
ids={"eventId":"b506d506-a0c7-4a90-b422-96a18633fe65"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GoFrameEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.6","os":"darwin"}
payloads={"docsEnabled":true,"docsPath":"/docs/","gfPort":8080,"promEnabled":true,"promPath":"/metrics","promPort":8080,"swEnabled":true,"swPath":"/sw/"}
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
< X-Request-Id: e2fd58ea-a2cc-475d-b051-3649a066d1d6
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-15T03:37:22.848829+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-15T03:37:22.848829+08:00
< ...
{"Message":"Hello rk-dev!"}
```

### 6.5 验证 API 日志
rk-boot 默认会使用如下格式打印 API 日志，也可以使用 JSON 格式，请参考用户指南。

```shell
------------------------------------------------------------------------
endTime=2022-04-15T03:37:22.848985+08:00
startTime=2022-04-15T03:37:22.848816+08:00
elapsedNano=168794
timezone=CST
ids={"eventId":"e2fd58ea-a2cc-475d-b051-3649a066d1d6","requestId":"e2fd58ea-a2cc-475d-b051-3649a066d1d6"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GoFrameEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.6","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=localhost:54183
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
