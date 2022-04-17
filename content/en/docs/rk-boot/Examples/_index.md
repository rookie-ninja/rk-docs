---
title: "Examples"
linkTitle: "Examples"
weight: 4
---
{{% pageinfo %}} 
Please visit: https://github.com/rookie-ninja/rk-boot/example/web/gin for full demo
{{% /pageinfo %}}

## Overview
We will use rk-boot start [gin-gonic/gin](https://github.com/gin-gonic/gin) microservice and add /v1/greeter API into it.

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
go get github.com/rookie-ninja/rk-gin/v2
```

## 1. Create boot.yaml
```yaml
gin:
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
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-gin/v2/boot"
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

	// Register API
	entry := rkgin.GetGinEntry("greeter")
	entry.Router.GET("/v1/greeter", Greeter)

	boot.Bootstrap(context.TODO())
	
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
func Greeter(ctx *gin.Context) {
	ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
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
2022-04-14T01:09:52.073+0800    INFO    boot/gin_entry.go:656   Bootstrap GinEntry      {"eventId": "432fcdfb-05e8-46a4-b7b2-d6b6967eba88", "entryName": "greeter", "entryType": "GinEntry"}
2022-04-14T01:09:52.074+0800    INFO    boot/gin_entry.go:413   SwaggerEntry: http://localhost:8080/sw/
2022-04-14T01:09:52.074+0800    INFO    boot/gin_entry.go:416   DocsEntry: http://localhost:8080/docs/
2022-04-14T01:09:52.074+0800    INFO    boot/gin_entry.go:419   PromEntry: http://localhost:8080/metrics
------------------------------------------------------------------------
endTime=2022-04-14T01:09:52.074233+08:00
startTime=2022-04-14T01:09:52.073337+08:00
elapsedNano=896392
timezone=CST
ids={"eventId":"432fcdfb-05e8-46a4-b7b2-d6b6967eba88"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GinEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"docsEnabled":true,"docsPath":"/docs/","ginPort":8080,"promEnabled":true,"promPath":"/metrics","promPort":8080,"swEnabled":true,"swPath":"/sw/"}
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
< X-Request-Id: 0aefef1a-1506-4e42-91d8-18e13f995a71
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-16T04:19:29.196441+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-16T04:19:29.196441+08:00
< ...
{"Message":"Hello rk-dev!"}
```

### 6.5 API Log
rk-boot will use bellow format of logging. JSON is also supported, please see user guide for details.

```shell
------------------------------------------------------------------------
endTime=2022-04-14T01:18:39.013794+08:00
startTime=2022-04-14T01:18:39.013433+08:00
elapsedNano=360521
timezone=CST
ids={"eventId":"7120529c-893b-4caa-a425-bc268de5cbc0","requestId":"7120529c-893b-4caa-a425-bc268de5cbc0"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GinEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=localhost:53333
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### 6.6 Prometheus Metrics
Access [http://localhost:8080/metrics](http://localhost:8080/metrics)

![](/rk-boot/example/api-metrics-gin.png)



