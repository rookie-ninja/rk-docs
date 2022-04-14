---
title: "Echo"
linkTitle: "Echo"
weight: 3
description: >
  Create [labstack/echo](https://github.com/labstack/echo) server with rk-boot and rk-echo plugins.
---

## Overview
We will use rk-boot start [labstack/echo](https://github.com/labstack/echo) microservice and add /v1/greeter API into it.

Furthermore, we will enable bellow functionalities.

| Functionality         | Description                                                 |
|:----------------------|:------------------------------------------------------------|
| Swagger UI            | Enable Swagger UI                                           |
| API Docs UI           | Enable RapiDoc UI                                           |
| Prometheus Client     | Enable Prometheus Client                                    |
| Logging middleware    | Automatically record logs for every API calls               |
| Prometheus middleware | Automatically record prometheus metrics for every API calls |
| Meta middleware       | Automatically add requestID for every API response          |

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## 1. Create boot.yaml
```yaml
echo:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true               # Enable Swagger UI，default path: /sw
    docs:
      enabled: true               # Enable API Doc UI，default path: /docs
    prom:
      enabled: true               # Enable Prometheus Client，default path: /metrics
    middleware:
      logging:
        enabled: true
      prom:
        enabled: true
      meta:
        enabled: true
```

## 2. Create main.go
In order to generate swagger.json file, we add comments and generate swagger.json file with [swag](https://github.com/swaggo/swag) CLI.

```go
package main

import (
  "context"
  _ "embed"
  "fmt"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-echo/boot"
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

  // Register handler
  echoEntry := rkecho.GetEchoEntry("greeter")
  echoEntry.Echo.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(ctx echo.Context) error {
  return ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.QueryParam("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

## 3.Generate swagger.json

```shell
$ swag init
```

## 4.Directory hierarchy
```shell
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

## 5.Run main.go
```go
$ go run main.go
2022-04-14T16:53:20.148+0800    INFO    boot/echo_entry.go:668  Bootstrap EchoEntry     {"eventId": "f94b667c-edec-42b2-8d0f-c55b459dcf42", "entryName": "greeter", "entryType": "EchoEntry"}
2022-04-14T16:53:20.149+0800    INFO    boot/echo_entry.go:434  SwaggerEntry: http://localhost:8080/sw/
2022-04-14T16:53:20.149+0800    INFO    boot/echo_entry.go:437  DocsEntry: http://localhost:8080/docs/
2022-04-14T16:53:20.149+0800    INFO    boot/echo_entry.go:440  PromEntry: http://localhost:8080/metrics
------------------------------------------------------------------------
endTime=2022-04-14T16:53:20.149574+08:00
startTime=2022-04-14T16:53:20.148641+08:00
elapsedNano=933258
timezone=CST
ids={"eventId":"f94b667c-edec-42b2-8d0f-c55b459dcf42"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"EchoEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.101","os":"darwin"}
payloads={"docsEnabled":true,"docsPath":"/docs/","echoPort":8080,"promEnabled":true,"promPath":"/metrics","promPort":8080,"swEnabled":true,"swPath":"/sw/"}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

## 6.Validate
### 6.1 Swagger UI
[http://localhost:8080/sw/](http://localhost:8080/sw/)

![](/rk-boot/example/sw.png)

### 6.2 API Docs UI
[http://localhost:8080/docs/](http://localhost:8080/docs/)

![](/rk-boot/example/docs.png)

### 6.3 Prometheus Client
[http://localhost:8080/metrics](http://localhost:8080/metrics)

![](/rk-boot/example/metrics.png)

### 6.4 Send request
```shell
$ curl -vs "localhost:8080/v1/greeter?name=rk-dev"
* ...
< X-Request-Id: d95d3b00-0d10-4784-83bf-bf392cf8bd77
< X-Rk-App-Domain: *
< X-Rk-App-Name: 
< X-Rk-App-Unix-Time: 2022-04-14T17:04:39.458193+08:00
< X-Rk-App-Version: 
< X-Rk-Received-Time: 2022-04-14T17:04:39.458193+08:00
< ...
{"Message":"Hello rk-dev!"}
```

### 6.5 API Log
rk-boot will use bellow format of logging. JSON is also supported, please see user guide for details.

```shell
------------------------------------------------------------------------
endTime=2022-04-14T17:04:39.458396+08:00
startTime=2022-04-14T17:04:39.458172+08:00
elapsedNano=223722
timezone=CST
ids={"eventId":"d95d3b00-0d10-4784-83bf-bf392cf8bd77","requestId":"d95d3b00-0d10-4784-83bf-bf392cf8bd77"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"EchoEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.101","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=localhost:64006
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### 6.6 Prometheus Metrics
Access [http://localhost:8080/metrics](http://localhost:8080/metrics)

![](/bootstrapper/example/api-metrics-gin.png)

## _**Cheers**_
![](/rk-boot/user-guide/cheers.png)