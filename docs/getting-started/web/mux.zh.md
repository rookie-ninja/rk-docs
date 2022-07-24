通过 rk-boot，配合 rk-mux 插件，创建 [gorilla/mux](https://github.com/gorilla/mux) 后台服务。

## 概述
我们将会使用 rk-boot 启动 [gorilla/mux](https://github.com/gorilla/mux) 微服务，并且添加 /v1/greeter API。

为了微服务的完整性，我们还会开启如下几个附加功能。

| 功能             | 介绍                                  |
|:---------------|:------------------------------------|
| Swagger UI     | 开启 Swagger UI                       |
| API Docs UI    | 开启 RapiDoc UI                       |
| Prometheus 客户端 | 开启 Prometheus 本地客户端                 |
| 日志中间件          | 针对每个 API 请求，自动记录日志                  |
| Prometheus 中间件 | 针对每个 API 请求，自动记录 Prometheus 记录      |
| 原数据中间件         | 针对每个 API 请求，自动添加 RequestID 到 Header |

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-mux
```

## 1. 创建 boot.yaml
```yaml
mux:
  - name: greeter
    port: 8080                    # 监听端口
    enabled: true                 # 开启微服务
    sw:
      enabled: true               # 开启 Swagger UI，默认路径为 /sw
    docs:
      enabled: true               # 开启 API Doc UI，默认路径为 /docs
    prom:
      enabled: true               # 开启 Prometheus 客户端，默认路径为 /metrics
    middleware:
      logging:
        enabled: true             # 开启 API 日志中间件
      prom:
        enabled: true             # 开启 API Prometheus 中间件
      meta:
        enabled: true             # 开启 API 原数据中间件，自动生成 RequestID
```

## 2. 创建 main.go
为了能给 [gorilla/mux](https://github.com/gorilla/mux) 提供 Swagger UI，我们需要在代码中添加一系列注视，然后通过 [swag](https://github.com/swaggo/swag) 命令行生成 swagger.json 文件。

```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
  "context"
  _ "embed"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-mux/boot"
  "github.com/rookie-ninja/rk-mux/middleware"
  "net/http"
)

// @title RK Swagger for Mux
// @version 1.0
// @description This is a greeter service with rk-boot.
func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Get MuxEntry
  muxEntry := rkmux.GetMuxEntry("greeter")
  // Use *mux.Router adding handler.
  muxEntry.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

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
func Greeter(writer http.ResponseWriter, req *http.Request) {
  rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

## 3.生成 swagger.json

```bash
$ swag init
```

## 4.文件夹结构
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

## 5.运行 main.go
```bash
$ go run main.go
2022-05-10T12:32:31.008+0800    INFO    boot/mux_entry.go:687   Bootstrap muxEntry      {"eventId": "aea73912-5a52-4ce2-9790-bc11ed4632dc", "entryName": "greeter", "entryType": "MuxEntry"}
2022-05-10T12:32:31.010+0800    INFO    boot/mux_entry.go:423   SwaggerEntry: http://localhost:8080/sw/
2022-05-10T12:32:31.011+0800    INFO    boot/mux_entry.go:426   DocsEntry: http://localhost:8080/docs/
2022-05-10T12:32:31.011+0800    INFO    boot/mux_entry.go:429   PromEntry: http://localhost:8080/metrics
------------------------------------------------------------------------
endTime=2022-05-10T12:32:31.011029+08:00
startTime=2022-05-10T12:32:31.008675+08:00
elapsedNano=2354002
timezone=CST
ids={"eventId":"aea73912-5a52-4ce2-9790-bc11ed4632dc"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"MuxEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"docsEnabled":true,"docsPath":"/docs/","muxPort":8080,"promEnabled":true,"promPath":"/metrics","promPort":8080,"swEnabled":true,"swPath":"/sw/"}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

## 6.验证
### 6.1 Swagger UI
[http://localhost:8080/sw/](http://localhost:8080/sw/)

![](../../img/example/sw.png)

### 6.2 API Docs UI
我们使用了 RapiDocs 作为 API Docs UI。其实 RapiDocs 也可以替换 Swagger UI 测试 API，后续我们会考虑替换 Swagger UI。

[http://localhost:8080/docs/](http://localhost:8080/docs/)

![](../../img/example/docs.png)

### 6.3 Prometheus 客户端
[http://localhost:8080/metrics](http://localhost:8080/metrics)

![](../../img/example/metrics.png)

### 6.4 发送请求
```bash
$ curl -vs "localhost:8080/v1/greeter?name=rk-dev"
* ...
< X-Request-Id: d95d3b00-0d10-4784-83bf-bf392cf8bd77
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-14T17:04:39.458193+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-14T17:04:39.458193+08:00
< ...
{"Message":"Hello rk-dev!"}
```

### 6.5 验证 API 日志
rk-boot 默认会使用如下格式打印 API 日志，也可以使用 JSON 格式，请参考用户指南。

```bash
------------------------------------------------------------------------
endTime=2022-05-10T12:32:53.25746+08:00
startTime=2022-05-10T12:32:53.256441+08:00
elapsedNano=1019084
timezone=CST
ids={"eventId":"af009c7f-f190-4935-8ee2-89c676c09546","requestId":"af009c7f-f190-4935-8ee2-89c676c09546"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"MuxEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=127.0.0.1:56043
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### 6.6 验证 Prometheus Metrics
访问 [http://localhost:8080/metrics](http://localhost:8080/metrics)

![](../../img/example/api-metrics-gin.png)

## _**Cheers**_
![](../../img/user-guide/cheers.png)
