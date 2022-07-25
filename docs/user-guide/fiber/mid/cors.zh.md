启动 CORS 中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## CORS 选项
| 名字                                 | 描述                                           | 类型       | 默认值                  |
|------------------------------------|----------------------------------------------|----------|----------------------|
| fiber.middleware.cors.enabled      | 启动 CORS 中间件                                  | boolean  | false                |
| fiber.middleware.cors.ignore       | 局部选项，忽略 API 路径                               | []string | []                   |
| fiber.middleware.cors.allowOrigins | 可通过验证的 Origin 地址。                            | []string | *                    |
| fiber.middleware.cors.allowMethods | 可通过的 http method, 会包含在 OPTIONS 请求的 Header 中。 | []string | All http methods     |
| fiber.middleware.cors.allowHeaders | 可通过的 http header, 会包含在 OPTIONS 请求的 Header 中。 | []string | Headers from request |
| fiber.middleware.cors.allowCredentials | 会包含在 OPTIONS 请求的 Header 中。                   | bool     | false                |
| fiber.middleware.cors.exposeHeaders | 会包含在 OPTIONS 请求的 Header 中的 Header。           | []string | ""                   |
| fiber.middleware.cors.maxAge        | 会包含在 OPTIONS 请求的 Header 中的 MaxAge。           | int      | 0                    |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      cors:
        enabled: true
        allowOrigins:
          - "http://localhost:*"
#        ignore: [""]
#        allowCredentials: false
#        allowHeaders: []
#        allowMethods: []
#        exposeHeaders: []
#        maxAge: 0
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

### 3.创建 cors.html
```html
<!DOCTYPE html>
<html>
<body>

<h1>CORS Test</h1>

<p>Call http://localhost:8080/v1/greeter</p>

<script type="text/javascript">
    window.onload = function() {
        var apiUrl = 'http://localhost:8080/v1/greeter';
        fetch(apiUrl).then(response => response.json()).then(data => {
            document.getElementById("res").innerHTML = data["Message"]
        }).catch(err => {
            document.getElementById("res").innerHTML = err
        });
    };
</script>

<h4>Response: </h4>
<p id="res"></p>

</body>
</html>
```

### 4.文件夹结构
```bash
.
├── boot.yaml
├── cors.html
├── go.mod
├── go.sum
└── main.go

0 directories, 5 files
```

### 5.验证
打开 cors.html

![](../../../img/user-guide/gin/basic/cors-success.png)

### 6.被拦截的 CORS
将 fiber.middleware.cors.allowOrigins 设置成 http://localhost:8080。

```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      cors:
        enabled: true
        allowOrigins:
          - "http://localhost:8080"
```

![](../../../img/user-guide/gin/basic/cors-fail.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)