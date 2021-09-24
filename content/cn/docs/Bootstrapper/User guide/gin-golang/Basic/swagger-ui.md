---
title: "Swagger 界面"
linkTitle: "Swagger 界面"
weight: 2
description: >
  启动 Swagger 界面.
---

## 先决条件
我们将使用 [swag](https://github.com/swaggo/swag) 命令行工具来生成 Swagger 界面所需要的参数文件。

> **选项 1:** 通过 [RK CMD](https://github.com/rookie-ninja/rk)
```shell script
# Install RK CMD
$ go get -u github.com/rookie-ninja/rk/cmd/rk

# Install swag with rk
$ rk install swag
```

> **选项 2:** 通过 [swag](https://github.com/swaggo/swag) 官网
```shell script
$ go get -u github.com/swaggo/swag/cmd/swag
```

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 Gin 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.name | Gin 服务名称 | string | N/A |
| gin.port | Gin 服务端口 | integer | nil, 服务不会启动 |
| gin.enabled | Gin 服务启动开关 ｜ bool | false |
| gin.description | Gin 服务的描述 | string | "" |

## Swagger 选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.sw.enabled | 启动 Swagger | boolean | false |
| gin.sw.path | Swagger Web 界面路径 | string | /sw |
| gin.sw.jsonPath | 本地 Swagger 参数文件（swagger.json）路径 | string | "" |
| gin.sw.headers | 每次 Swagger 界面请求，都会带着这些头部。格式： [key:value] | []string | [] |

## 快速开始
### 1.创建 boot.yaml
> 为了能让启动器读取生成出来的 Swagger 参数文件，我们需要在 boot.yaml 文件中指定 **gin.sw.jsonPath**。
> 
> 在下面的例子中，我们使用了 "docs" 路径，因为 swag 命令行会把参数文件生成到 docs 文件夹中。

```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
      jsonPath: "docs"
#      path: "sw"        # Default value is "sw", change it as needed
#      headers: []       # Headers that will be set while accessing swagger UI main page.
```

### 2.创建 main.go
> 为了能让 swag 命令行生成 Swagger 参数文件，我们需要在代码中写注释。
>
> 详情可参考 [swag](https://github.com/swaggo/swag) 官方文档。

```go
package main

import (
	"context"
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// @title RK Swagger for Gin
// @version 1.0
// @description This is a greeter service with rk-boot.

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
	boot.GetGinEntry("greeter").Router.GET("/v1/greeter", Greeter)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

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

### 3.生成 swagger 参数文件
```shell script
$ swag init

# swagger.json, swagger.yaml, docs.go files will be generated under ./docs folder.
$ tree
.
├── boot.yaml
├── docs
│   ├── docs.go
│   ├── swagger.json
│   └── swagger.yaml
├── go.mod
├── go.sum
└── main.go

1 directory, 7 files
```

### 4.验证
> **Swagger:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/gin-golang/gin-sw-api.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)