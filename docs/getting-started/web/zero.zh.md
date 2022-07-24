通过 rk-boot，配合 rk-zero 插件，创建 [zeromicro/go-zero](https://github.com/zeromicro/go-zero) 后台服务。

## 概述
我们将会使用 rk-boot 启动 [zeromicro/go-zero](https://github.com/zeromicro/go-zero) 微服务，并且添加 /v1/greeter API。

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
go get github.com/rookie-ninja/rk-zero
```

## 1. 创建 boot.yaml
```yaml
zero:
  - name: greeter
    port: 8080                    # 监听端口
    enabled: true                 # 开启 zero 微服务
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
为了能给 [zeromicro/go-zero](https://github.com/zeromicro/go-zero) 提供 Swagger UI，我们需要在代码中添加一系列注视，然后通过 [swag](https://github.com/swaggo/swag) 命令行生成 swagger.json 文件。

```go
package main

import (
  "context"
  "encoding/json"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-zero/boot"
  "github.com/zeromicro/go-zero/rest"
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
  zeroEntry := rkzero.GetZeroEntry("greeter")
  zeroEntry.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: Greeter,
  })

  // Bootstrap
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
func Greeter(writer http.ResponseWriter, request *http.Request) {
  writer.WriteHeader(http.StatusOK)
  resp := &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", request.URL.Query().Get("name")),
  }
  bytes, _ := json.Marshal(resp)
  writer.Write(bytes)
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
2022-05-09T13:37:24.802+0800    INFO    boot/zero_entry.go:761  Bootstrap zeroEntry     {"eventId": "c2451ccd-b517-4b37-8691-ff621a9a0223", "entryName": "greeter", "entryType": "ZeroEntry"}
2022-05-09T13:37:24.803+0800    INFO    boot/zero_entry.go:519  SwaggerEntry: http://localhost:8080/sw/
2022-05-09T13:37:24.803+0800    INFO    boot/zero_entry.go:522  DocsEntry: http://localhost:8080/docs/
2022-05-09T13:37:24.803+0800    INFO    boot/zero_entry.go:525  PromEntry: http://localhost:8080/metrics
------------------------------------------------------------------------
endTime=2022-05-09T13:37:24.803892+08:00
startTime=2022-05-09T13:37:24.802285+08:00
elapsedNano=1607377
timezone=CST
ids={"eventId":"c2451ccd-b517-4b37-8691-ff621a9a0223"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"docsEnabled":true,"docsPath":"/docs/","promEnabled":true,"promPath":"/metrics","promPort":8080,"swEnabled":true,"swPath":"/sw/","zeroPort":8080}
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
< X-Request-Id: 7120529c-893b-4caa-a425-bc268de5cbc0
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-15T03:37:22.848829+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-14T01:18:39.013447+08:00
< ...
{"Message":"Hello rk-dev!"}
```

### 6.5 验证 API 日志
rk-boot 默认会使用如下格式打印 API 日志，也可以使用 JSON 格式，请参考用户指南。

```bash
------------------------------------------------------------------------
endTime=2022-05-09T13:37:47.631765+08:00
startTime=2022-05-09T13:37:47.63152+08:00
elapsedNano=245785
timezone=CST
ids={"eventId":"7d461811-81e6-4b48-ae0a-a03c5ed66b5d","requestId":"7d461811-81e6-4b48-ae0a-a03c5ed66b5d","traceId":"872dfba05620f46e12ad9ba75cd9ada4"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=localhost:64509
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
