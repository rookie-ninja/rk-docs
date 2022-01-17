---
title: "例子"
linkTitle: "例子"
weight: 4
description: >
  RK 推荐的标准服务格式。
---
{{% pageinfo %}} 
完整例子: https://github.com/rookie-ninja/rk-demo
{{% /pageinfo %}}

## 概述
我们将会添加 /v1/greeter API 到服务中，并且开启如下功能。

- 通用 API
- Swagger 界面
- RK TV
- Prometheus 客户端
- 日志拦截器
- 监控拦截器
- 元数据拦截器
- 调用链拦截器（没有 Exporter）

## 布局
```shell script
$ tree
.
├── Dockerfile
├── LICENSE
├── Makefile
├── README.md
├── boot.yaml
├── build.yaml
├── docs
│   ├── docs.go
│   ├── swagger.json
│   └── swagger.yaml
├── go.mod
├── go.sum
└── internal
    ├── api
    │   └── v1
    │       ├── greeter.go
    │       └── greeter_test.go
    └── main.go

4 directories, 14 files
```

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/bin
```

## 代码
### boot.yaml
```yaml
---
app:
  description: "Write your description"
zapLogger:
  - name: appLog
    zap:
      outputPaths:
        - "logs/app.log"
        - "stdout"
eventLogger:
  - name: eventLog
    outputPaths:
      - "logs/event.log"
      - "stdout"
gin:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
      jsonPath: "docs"
    commonService:
      enabled: true
    tv:
      enabled:  true
    prom:
      enabled: true
    logger:
      zapLogger:
        ref: appLog
      eventLogger:
        ref: eventLog
    interceptors:
      loggingZap:
        enabled: true
      metricsProm:
        enabled: true
      meta:
        enabled: true
      tracingTelemetry:
        enabled: true
```

### internal/main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-boot/gin"
	"github.com/rookie-ninja/rk-demo/internal/api/v1"
)

// @title RK Swagger for Gin
// @version 1.0
// @description This is a greeter service with rk-boot.
// @termsOfService http://swagger.io/terms/

// @securityDefinitions.basic BasicAuth

// @contact.name API Support
// @contact.url http://www.swagger.io/support
// @contact.email support@swagger.io

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html

// @name Authorization

// @schemes http https

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
	rkbootgin.GetGinEntry("greeter").Router.GET("/v1/greeter", api.Greeter)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### internal/api/v1/greeter.go
```go
package api

import (
	"fmt"
	"github.com/gin-gonic/gin"
	"net/http"
)

// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(ctx *gin.Context) {
	ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
	})
}

// Response.
type GreeterResponse struct {
	Message string
}
```

## 运行
```go
$ go run internal/main.go
```

## 验证
- RK TV: [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)
- Swagger 界面: [http://localhost:8080/sw](http://localhost:8080/sw)
- Prometheus 客户端: [http://localhost:8080/metrics](http://localhost:8080/metrics)