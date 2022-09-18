当使用 rk-boot 的时候，如何通过 flag 覆盖 boot.yaml 里的参数？

## 概述
rk-boot 支持通过命令行和环境变量参数来覆盖 boot.yaml 里的值。

- 覆盖 boot.yaml 里的参数 (通过 \-\-rkset)

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-fiber
```

### 2.创建 boot.yaml
```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.创建 main.go

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

### 5.通过 flag 覆盖 Port
如果是 List，请使用【fiber[0].port】格式访问。

```bash
$ go run main.go --rkset fiber[0].port=8081
2022-05-12T13:25:47.605+0800    INFO    entry/util.go:332       Found flag to override, applying...     {"flags": ["fiber[0].port=8081"]}
2022-05-12T13:25:47.606+0800    INFO    boot/fiber_entry.go:705 Bootstrap fiberEntry    {"eventId": "ad3915a1-2d2c-430a-8d94-6f74f2cbd19a", "entryName": "greeter", "entryType": "FiberEntry"}
------------------------------------------------------------------------
endTime=2022-05-12T13:25:47.606382+08:00
startTime=2022-05-12T13:25:47.606323+08:00
elapsedNano=58398
timezone=CST
ids={"eventId":"ad3915a1-2d2c-430a-8d94-6f74f2cbd19a"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"FiberEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"fiberPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

> 发送请求到：8081
```bash
$ curl "localhost:8081/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 6.通过环境变量覆盖 Port
覆盖 boot.yaml 里参数的时候，需要带上 【RK_】 作为前缀，此外，如果是 List 使用 【_X_】 访问。

例如：RK_FIBER_0_PORT

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
  "os"
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
  os.Setenv("RK_FIBER_0_PORT", "8081")

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

```bash
$ go run main.go
2022-05-12T13:27:16.125+0800    INFO    entry/util.go:299       Found ENV to override, applying...      {"env": ["RK_FIBER_0_PORT=8081 => fiber[0].port=8081"]}
2022-05-12T13:27:16.125+0800    INFO    boot/fiber_entry.go:705 Bootstrap fiberEntry    {"eventId": "3e789393-52a2-41bc-a2fe-cad3d4f78fea", "entryName": "greeter", "entryType": "FiberEntry"}
------------------------------------------------------------------------
endTime=2022-05-12T13:27:16.125842+08:00
startTime=2022-05-12T13:27:16.125747+08:00
elapsedNano=95107
timezone=CST
ids={"eventId":"3e789393-52a2-41bc-a2fe-cad3d4f78fea"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"FiberEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"fiberPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```