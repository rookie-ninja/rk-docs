---
title: "例子"
linkTitle: "例子"
weight: 4
---
{{% pageinfo %}} 
完整例子: https://github.com/rookie-ninja/rk-boot/example/web/gin
{{% /pageinfo %}}

## 概述
我们将会使用 rk-boot 启动 [gin-gonic/gin](https://github.com/gin-gonic/gin) 微服务，并且添加 /v1/greeter API。

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
go get github.com/rookie-ninja/rk-gin/v2
```

## 1. 创建 boot.yaml
```yaml
gin:
  - name: greeter
    port: 8080                    # 监听端口
    enabled: true                 # 开启 gin 微服务
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
为了能给 [gin-gonic/gin](https://github.com/gin-gonic/gin) 提供 Swagger UI，我们需要在代码中添加一系列注视，然后通过 [swag](https://github.com/swaggo/swag) 命令行生成 swagger.json 文件。

```go
package main

import (
	"context"
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-gin/v2/boot"
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
	entry := rkgin.GetGinEntry("greeter")
	entry.Router.GET("/v1/greeter", Greeter)

	// 启动
	boot.Bootstrap(context.TODO())

	// 等待关闭信号
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
func Greeter(ctx *gin.Context) {
	ctx.JSON(http.StatusOK, &GreeterResponse{
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
2022-04-14T01:09:52.073+0800    INFO    boot/gin_entry.go:656   Bootstrap GinEntry      {"eventId": "432fcdfb-05e8-46a4-b7b2-d6b6967eba88", "entryName": "greeter", "entryType": "GinEntry"}
2022-04-14T01:09:52.074+0800    INFO    boot/gin_entry.go:413   SwaggerEntry: http://localhost:8080/sw/
2022-04-14T01:09:52.074+0800    INFO    boot/gin_entry.go:416   DocsEntry: http://localhost:8080/docs/
2022-04-14T01:09:52.074+0800    INFO    boot/gin_entry.go:419   PromEntry: http://localhost:8080/metrics
------------------------------------------------------------------------
endTime=2022-04-14T01:09:52.074233+08:00
startTime=2022-04-14T01:09:52.073337+08:00
elapsedNano=896392
timezone=CST
ids={"eventId":"432fcdfb-05e8-46a4-b7b2-d6b6967eba88"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"GinEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"docsEnabled":true,"docsPath":"/docs/","ginPort":8080,"promEnabled":true,"promPath":"/metrics","promPort":8080,"swEnabled":true,"swPath":"/sw/"}
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
< X-Request-Id: 7120529c-893b-4caa-a425-bc268de5cbc0
< X-Rk-App-Domain: *
< X-Rk-App-Unix-Time: 2022-04-14T01:18:39.013447+08:00
< X-Rk-Received-Time: 2022-04-14T01:18:39.013447+08:00
< ...
{"Message":"Hello rk-dev!"}
```

### 6.5 验证 API 日志
rk-boot 默认会使用如下格式打印 API 日志，也可以使用 JSON 格式，请参考用户指南。

```shell
------------------------------------------------------------------------
endTime=2022-04-14T01:18:39.013794+08:00
startTime=2022-04-14T01:18:39.013433+08:00
elapsedNano=360521
timezone=CST
ids={"eventId":"7120529c-893b-4caa-a425-bc268de5cbc0","requestId":"7120529c-893b-4caa-a425-bc268de5cbc0"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"GinEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=localhost:53333
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### 6.6 验证 Prometheus Metrics
访问 [http://localhost:8080/metrics](http://localhost:8080/metrics)

![](/rk-boot/example/api-metrics-gin.png)



