---
title: "使用 embedFS"
linkTitle: "使用 embedFS"
weight: 11
description: >
  rk-boot 中，可以使用 embed.FS 给 rk-boot 提供 boot.yaml 等静态文件。
---

## 概述
目前，用户可以把 Swagger UI 参数文件, API Docs 参数文件和 boot.yaml 文件通过 embed.FS 传递给 rk-boot.

## 快速开始
### 1.安装

```shell script
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-fiber
```

### 2.创建 boot.yaml
```yaml
fiber:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/gofiber/fiber/v2"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-fiber/boot"
  "net/http"
)

//go:embed boot.yaml
var bootRaw []byte

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot(rkboot.WithBootConfigRaw(bootRaw))

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Register handler
  entry := rkfiber.GetFiberEntry("greeter")
  entry.App.Get("/v1/greeter", Greeter)
  // This is required!!!
  entry.RefreshFiberRoutes()

  boot.WaitForShutdownSig(context.TODO())
}

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

### 4.启动 main.go
```shell
$ go run main.go
2022-05-12T09:44:05.588+0800    INFO    boot/fiber_entry.go:705 Bootstrap fiberEntry    {"eventId": "7fb03193-ce01-4a8e-bc9d-5315f3d9a18c", "entryName": "greeter", "entryType": "FiberEntry"}
------------------------------------------------------------------------
endTime=2022-05-12T09:44:05.589397+08:00
startTime=2022-05-12T09:44:05.588124+08:00
elapsedNano=1273119
timezone=CST
ids={"eventId":"7fb03193-ce01-4a8e-bc9d-5315f3d9a18c"}
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

### 5.验证
```shell
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}%
```

## Swagger UI 与 Docs UI
我们同时还可以让 rk-boot 使用 embedFS 读取 swagger 相关的参数文件。

### 1.已存在的 Swagger JSON 文件
```shell
├── docs
│   ├── swagger.json
│   └── swagger.yaml
```

### 2.创建 boot.yaml
```yaml
fiber:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
    docs:
      enabled: true
```

### 3.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/gofiber/fiber/v2"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-fiber/boot"
  "net/http"
)

//go:embed boot.yaml
var bootRaw []byte

//go:embed docs
var docsFS embed.FS

//go:embed docs
var swFS embed.FS

func init() {
  rkentry.GlobalAppCtx.AddEmbedFS(rkentry.SWEntryType, "greeter", &docsFS)
  rkentry.GlobalAppCtx.AddEmbedFS(rkentry.DocsEntryType, "greeter", &swFS)
}

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot(rkboot.WithBootConfigRaw(bootRaw))

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Register handler
  entry := rkfiber.GetFiberEntry("greeter")
  entry.App.Get("/v1/greeter", Greeter)
  // This is required!!!
  entry.RefreshFiberRoutes()

  boot.WaitForShutdownSig(context.TODO())
}

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
