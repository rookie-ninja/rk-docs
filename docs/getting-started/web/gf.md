Create [gogf/gf](https://github.com/gogf/gf) server with rk-boot and rk-gf plugins.

## Overview
We will use rk-boot start [gogf/gf](https://github.com/gogf/gf) microservice and add /v1/greeter API into it.

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
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## 1. Create boot.yaml
```yaml
gf:
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
  "fmt"
  "github.com/gogf/gf/v2/net/ghttp"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gf/boot"
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
  boot := rkboot.NewBoot()

  // 注册 API
  entry := rkgf.GetGfEntry("greeter")
  entry.Server.BindHandler("/v1/greeter", Greeter)

  // 启动
  boot.Bootstrap(context.TODO())

  // 等待关闭信号
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
func Greeter(ctx *ghttp.Request) {
  ctx.Response.WriteHeader(http.StatusOK)
  ctx.Response.WriteJson(&GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name").String()),
  })
}

type GreeterResponse struct {
  Message string
}
```

## 3.Generate swagger.json

```bash
$ swag init
```

## 4.Directory hierarchy
```bash
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
```bash
$ go run main.go
2022-04-15T03:36:22.322+0800    INFO    boot/gf_entry.go:631    Bootstrap gfEntry       {"eventId": "b506d506-a0c7-4a90-b422-96a18633fe65", "entryName": "greeter", "entryType": "GoFrameEntry"}
2022-04-15T03:36:22.323+0800    INFO    boot/gf_entry.go:396    SwaggerEntry: http://localhost:8080/sw/
2022-04-15T03:36:22.324+0800    INFO    boot/gf_entry.go:399    DocsEntry: http://localhost:8080/docs/
2022-04-15T03:36:22.324+0800    INFO    boot/gf_entry.go:402    PromEntry: http://localhost:8080/metrics
------------------------------------------------------------------------
endTime=2022-04-15T03:36:22.324017+08:00
startTime=2022-04-15T03:36:22.322457+08:00
elapsedNano=1560663
timezone=CST
ids={"eventId":"b506d506-a0c7-4a90-b422-96a18633fe65"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GoFrameEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.6","os":"darwin"}
payloads={"docsEnabled":true,"docsPath":"/docs/","gfPort":8080,"promEnabled":true,"promPath":"/metrics","promPort":8080,"swEnabled":true,"swPath":"/sw/"}
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

![](../../img/example/sw.png)

### 6.2 API Docs UI
[http://localhost:8080/docs/](http://localhost:8080/docs/)

![](../../img/example/docs.png)

### 6.3 Prometheus Client
[http://localhost:8080/metrics](http://localhost:8080/metrics)

![](../../img/example/metrics.png)

### 6.4 Send request
```bash
$ curl -vs "localhost:8080/v1/greeter?name=rk-dev"
* ...
< X-Request-Id: e2fd58ea-a2cc-475d-b051-3649a066d1d6
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-15T03:37:22.848829+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-15T03:37:22.848829+08:00
< ...
{"Message":"Hello rk-dev!"}
```

### 6.5 API Log
rk-boot will use bellow format of logging. JSON is also supported, please see user guide for details.

```bash
------------------------------------------------------------------------
endTime=2022-04-15T03:37:22.848985+08:00
startTime=2022-04-15T03:37:22.848816+08:00
elapsedNano=168794
timezone=CST
ids={"eventId":"e2fd58ea-a2cc-475d-b051-3649a066d1d6","requestId":"e2fd58ea-a2cc-475d-b051-3649a066d1d6"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GoFrameEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.6","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=localhost:54183
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### 6.6 Prometheus Metrics
Access [http://localhost:8080/metrics](http://localhost:8080/metrics)

![](../../img/example/api-metrics-gin.png)

## _**Cheers**_
![](../../img/user-guide/cheers.png)