启动权限中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## 权限选项
| 名字                          | 描述                           | 类型       | 默认值   |
|-----------------------------|------------------------------|----------|-------|
| fiber.middleware.auth.enabled | 启动权限中间件                      | boolean  | false |
| fiber.middleware.auth.ignore  | 局部选项，忽略 API 路径               | []string | []    |
| fiber.middleware.auth.basic   | Basic Auth 信息，格式：<user:pass> | []string | []    |
| fiber.middleware.auth.apiKey  | X-API-Key                    | []string | []    |
| fiber.middleware.auth.ignore  | 提供字符串前缀，中间件会忽略包含这些字符串的请求路径   | []string | []    |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      auth:
        enabled: true
        basic: ["user:pass"]
#        ignore: [""]
#        apiKey:
#          - "keys"
```

### 2.创建 main.go
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

### 3.验证
```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
# This is RK style error code if unauthorized
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"Missing authorization, provide one of bellow auth header:[Basic Auth]",
        "details":[]
    }
}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 4.X-API-Key 类型授权
```yaml
---
fiber:
  - name: greeter
    ...
    middleware:
      auth:
        enabled: true
        apiKey: ["token"]
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5.忽略请求路径
```yaml
---
fiber:
  - name: greeter
    ...
    middleware:
      auth:
        enabled: true
        basic: ["user:pass"]
        ignorePrefix: ["/v1/greeter"]
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)