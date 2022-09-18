Enable API middleware logging

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## Options
| options                                   | description                 | type     | default |
|-------------------------------------------|-----------------------------|----------|---------|
| fiber.middleware.logging.enabled           | Enable logging middleware   | boolean  | false   |
| fiber.middleware.logging.ignore            | Ignore by path              | []string | []      |
| fiber.middleware.logging.loggerEncoding    | Logger format：json, console | string   | console |
| fiber.middleware.logging.loggerOutputPaths | Logger output path          | []string | stdout  |
| fiber.middleware.logging.eventEncoding     | Event format：json, console  | string   | console |
| fiber.middleware.logging.eventOutputPaths  | Event output path           | []string | stdout  |

## Concept
1. [Event](https://github.com/rookie-ninja/rk-query)
2. [Logger](https://github.com/uber-go/zap)

### Logger
rk-boot use [zap](https://github.com/uber-go/zap) as underlying logger instance，[lumberjack](https://github.com/natefinch/lumberjack) as log rotation.

> Example
>
> ```bash
> 2022-05-12T09:32:21.311+0800    INFO    boot/fiber_entry.go:705 Bootstrap fiberEntry    {"eventId": "feac04f5-8ddf-48a1-89e1-d1564d6de927", "entryName": "greeter", "entryType": "FiberEntry"}
> ```

### Event
rk-boot will record RPC metadata into **Event**.

| Fields      | Description                                                                                                 |
|-------------|-------------------------------------------------------------------------------------------------------------|
| endTime     | End time                                                                                                    |
| startTime   | Start time                                                                                                  |
| elapsedNano | Event elapsed（Nanoseconds）                                                                                  |
| timezone    | Timezone                                                                                                    |
| ids         | Includes eventId, requestId and traceId                                                                     |
| app         | Includes [appName, appVersion](https://github.com/rookie-ninja/rk-entry#appinfoentry), entryName, entryType |
| env         | Includes arch, domain, hostname, localIP, os 字段。这些字段来自系统环境变量（DOMAIN）。 "*" 代表环境变量为空                          |
| payloads    | Includes RPC metadata                                                                                       |
| error       | Includes errors                                                                                             |
| counters    | Set through event.SetCounter()                                                                              |
| pairs       | Set through event.AddPair()                                                                                 |
| timing      | Set through event.StartTimer() and event.EndTimer()                                                         |
| remoteAddr  | RPC remote address                                                                                          |
| operation   | RPC name                                                                                                    |
| resCode     | RPC return code                                                                                             |
| eventStatus | Ended or InProgress                                                                                         |

> Example
>
> ```bash
> ------------------------------------------------------------------------
> endTime=2021-06-25T01:30:45.144023+08:00
> startTime=2021-06-25T01:30:45.143767+08:00
> elapsedNano=255948
> timezone=CST
> ids={"eventId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","requestId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","traceId":"65b9aa7a9705268bba492fdf4a0e5652"}
> app={"appName":"rk-gin","appVersion":"master-xxx","entryName":"greeter","entryType":"FiberEntry"}
> env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
> payloads={"apiMethod":"GET","apiPath":"/rk/v1/alive","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
> error={}
> counters={}
> pairs={}
> timing={}
> remoteAddr=localhost:60718
> operation=/rk/v1/alive
> resCode=200
> eventStatus=Ended
> EOE
> ```

## Quick start
### 1.Create boot.yaml
```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      logging:
        enabled: true                                     # Optional, default: false
#        ignore: [""]                                      # Optional, default: []
#        loggerEncoding: "console"                         # Optional, default: "console"
#        loggerOutputPaths: ["logs/app.log"]               # Optional, default: ["stdout"]
#        eventEncoding: "console"                          # Optional, default: "console"
#        eventOutputPaths: ["logs/event.log"]              # Optional, default: ["stdout"]
```

### 2.Create main.go
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

### 3.Validate
> Send request

```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

> Log

```bash
------------------------------------------------------------------------
endTime=2022-05-12T09:32:27.487736+08:00
startTime=2022-05-12T09:32:27.487473+08:00
elapsedNano=263091
timezone=CST
ids={"eventId":"37a14783-f2e9-49da-8839-64f6c4697f55","requestId":"37a14783-f2e9-49da-8839-64f6c4697f55"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"FiberEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=127.0.0.1:64411
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 4. JSON format
```yaml
---
fiber:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerEncoding: "json"
        eventEncoding: "json"
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5. Log output path
> By default, log will be rotated every 1GB and compressed by lumberjack.

```yaml
---
fiber:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerOutputPaths: ["logs/app.log"]
        eventOutputPaths: ["logs/event.log"]
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 6. Log instance
rk-boot will add RequestId into logger instance for every RPC calls.

```go
func Greeter(ctx *fiber.Ctx) error {
    rkfiberctx.GetLogger(ctx).Info("Received request")

    ctx.Response().SetStatusCode(http.StatusOK)
    return ctx.JSON(&GreeterResponse{
        Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
    })
}
```

```bash
2022-04-15T19:17:01.380+0800    INFO    fiber/main.go:55  Received request        {"requestId": "446c1138-1b20-48f7-a479-7b3e8a26de40"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 7. Change Event
rk-boot will create Event for every RPC call.

```go
func Greeter(ctx *fiber.Ctx) error {
    event := rkfiberctx.GetEvent(ctx)
    event.AddPair("key", "value")

    ctx.Response().SetStatusCode(http.StatusOK)
    return ctx.JSON(&GreeterResponse{
        Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
    })
}
```

```bash
------------------------------------------------------------------------
endTime=2022-05-12T09:42:28.482306+08:00
startTime=2022-05-12T09:42:28.48184+08:00
elapsedNano=466210
timezone=CST
ids={"eventId":"32bf4198-3958-40b3-869c-fbe621909172","requestId":"32bf4198-3958-40b3-869c-fbe621909172"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"FiberEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=127.0.0.1:64542
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)
