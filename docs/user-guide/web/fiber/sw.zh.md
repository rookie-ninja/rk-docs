启动 Swagger UI

## 先决条件
我们将使用 [swag](https://github.com/swaggo/swag) 命令行工具来生成 Swagger UI 所需要的 swagger.json 文件。

> 通过 [swag](https://github.com/swaggo/swag) 官网
```bash
$ go get -u github.com/swaggo/swag/cmd/swag
```

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## Swagger 选项
| 名字                | 描述                                       | 类型       | 默认值   |
|-------------------|------------------------------------------|----------|-------|
| fiber.sw.enabled  | 启动 Swagger                               | boolean  | false |
| fiber.sw.path     | Swagger Web 界面路径                         | string   | sw    |
| fiber.sw.jsonPath | 本地 Swagger 参数文件（swagger.json）路径          | string   | ""    |
| fiber.sw.headers  | 每次 Swagger 界面请求，都会带着这些头部。格式： [key:value] | []string | []    |

## 快速开始
### 1.创建 boot.yaml
> rk-boot 默认会在 docs/, api/gen/v1/ 文件夹中寻找 swagger JSON 文件。
>
> 如果 swagger JSON 文件在其他的路径，需要在 boot.yaml 文件中指定 **fiber.sw.jsonPath**。

```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
#      jsonPath: ""
#      path: "sw"
#      headers: []
```

### 2.创建 main.go
> 为了能让 swag 命令行生成 Swagger JSON 文件，我们需要在代码中写注释。
>
> 详情可参考 [swag](https://github.com/swaggo/swag) 官方文档。

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

### 3.生成 swagger 参数文件
```bash
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

![](../../../img/example/sw.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)