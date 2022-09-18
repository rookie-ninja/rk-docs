启动原数据中间件。

## 概述
原数据中间件将会把下面的信息，以 HTTP 头部的形式，返回给客户。

| Header 键                 | 详情             |
|--------------------------|----------------|
| X-Request-Id             | 自动生成请求 ID。     |
| X-[Prefix]-App-Domain    | 当前环境。          |
| X-[Prefix]-App-Unix-Time | 当前服务的 Unix 时间。 |
| X-[Prefix]-Received-Time | 接收到请求的时间戳。     |

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## 原数据选项
| 名字                          | 描述             | 类型       | 默认值   |
|-----------------------------|----------------|----------|-------|
| fiber.middleware.meta.enabled | 启动原数据中间件       | boolean  | false |
| fiber.middleware.meta.prefix  | X-<Prefix>-XXX | string   | RK    |
| fiber.middleware.meta.ignore  | 局部选项，忽略 API 路径 | []string | []    |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      meta:
        enabled: true                                      # Optional, default: false
#        ignore: [""]                                      # Optional, default: []
#        prefix: ""                                        # Optional, default: "RK"
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
> 发送请求

```bash
$ curl -vs -X GET "localhost:8080/v1/greeter?name=rk-dev"
...
< X-Request-Id: dad72f8c-cd35-43b6-9414-493edf7e0d10
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-15T23:14:52.495+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-15T23:14:52.495+08:00
...
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 4.覆盖 requestId
```go
func Greeter(ctx *fiber.Ctx) error {
    // Override request id
    rkfiberctx.SetHeaderToClient(ctx, rkmid.HeaderRequestId, "request-id-override")
    // We expect new request id attached to logger
    rkfiberctx.GetLogger(ctx).Info("Received request")

	return ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
	})
}
```

```bash
$ curl -vs -X GET "localhost:8080/v1/greeter?name=rk-dev"
...
< X-Request-Id: request-id-override
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-15T23:20:37.763436+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-15T23:20:37.763436+08:00
...
{"Message":"Hello rk-dev!"}
```

> 如果我们启动了日志中间件，那我们会看到如下的日志。

```bash
2022-05-12T09:44:07.785+0800    INFO    fiber/main.go:61        Received request        {"requestId": "request-id-override"}
------------------------------------------------------------------------
endTime=2022-05-12T09:44:07.785433+08:00
startTime=2022-05-12T09:44:07.785006+08:00
elapsedNano=427310
timezone=CST
ids={"eventId":"request-id-override","requestId":"request-id-override"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"FiberEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=127.0.0.1:64561
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5.覆盖 header prefix
```yaml
---
fiber:
  - name: greeter
    ...
    middleware:
      meta:
        enabled: true
        prefix: "Override"
```

```bash
$ curl -X GET -vs "localhost:8080/v1/greeter?name=rk-dev"
...
< X-Override-App-Domain: *
< X-Override-App-Name: rk
< X-Override-App-Unix-Time: 2022-04-15T23:22:18.087064+08:00
< X-Override-App-Version: local
< X-Override-Received-Time: 2022-04-15T23:22:18.087064+08:00
< X-Request-Id: request-id-override
...
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)