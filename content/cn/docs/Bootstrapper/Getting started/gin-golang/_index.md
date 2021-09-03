---
title: "Gin-golang"
linkTitle: "Gin-golang"
weight: 1
description: >
  通过 RK 启动器，创建基于 [Gin](https://github.com/gin-gonic/gin) 框架的服务。
---

## 概述
让我们通过编辑 boot.yaml 文件来启动基于 Gin 框架的服务。

> 例子: https://github.com/rookie-ninja/rk-demo/tree/master/gin/getting-started

- [Gin 服务](https://github.com/gin-gonic/gin)
- [Swagger 界面](https://swagger.io/tools/swagger-ui/)
- 通用 API
- RK TV Web 界面
- 用户自定义 API

## 创建服务
### 1.安装
```shell script
$ go get github.com/rookie-ninja/rk-boot
```

### 2.创建 boot.yaml
```yaml
---
gin:
  - name: greeter       # Name of gin entry
    port: 8080          # Port of gin entry
    sw:
      enabled: true     # Enable swagger UI
    commonService:
      enabled: true     # Enable common service
    tv:
      enabled:  true    # Enable RK TV
```

### 3.创建 main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 4.启动服务
```go
$ go run main.go
...
2021-07-02T04:53:25.921+0800    INFO    boot/gin_entry.go:630   Bootstrapping GinEntry. {"eventId": "8fd734a2-549a-4b81-bea2-fe8e94666ab7", "entryName": "greeter", "entryType": "GinEntry", "port": 8080, "interceptorsCount": 1, "swEnabled": true, "tlsEnabled": false, "commonServiceEnabled": true, "tvEnabled": true, "swPath": "/sw/"}
------------------------------------------------------------------------
endTime=2021-07-02T04:53:25.921604+08:00
startTime=2021-07-02T04:53:25.919344+08:00
elapsedNano=2259975
timezone=CST
ids={"eventId":"8fd734a2-549a-4b81-bea2-fe8e94666ab7"}
app={"appName":"rk-demo","appVersion":"master-f414049","entryName":"greeter","entryType":"GinEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.6","os":"darwin","realm":"*","region":"*"}
payloads={"commonServiceEnabled":true,"entryName":"greeter","entryType":"GinEntry","interceptorsCount":1,"port":8080,"swEnabled":true,"swPath":"/sw/","tlsEnabled":false,"tvEnabled":true}
error={}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=bootstrap
resCode=OK
eventStatus=Ended
EOE
```

### 5.验证
> **Swagger 界面:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/gin-golang/gin-sw.png)

> **TV 界面:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/gin-golang/gin-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## 添加一个 API
我们将会添加 "/v1/greeter" API。

### 1.注册 API
> 基于 Gin 框架，我们需要拿到 **gin.Engine** 实例，才能注册 **HandlerFunc**。
>
> 在 GinEntry 中， 启动器会初始化 **gin.Engine**，并更名为 **Router**。

```go
package main

import (
	"context"
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler before bootstrap!
	boot.GetGinEntry("greeter").Router.GET("/v1/greeter", Greeter)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

// Gin handler function
func Greeter(ctx *gin.Context) {
	ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
	})
}

// Response.
type GreeterResponse struct {
	Message string
}
```

### 2.验证
```shell script
$ curl "http://localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## 支持 Swagger 界面
我们推荐使用 [swag](https://github.com/swaggo/swag) 来创建 Swagger 界面所需要的参数文件。

### 1.安装 swag
> **选项 1:** 
>
> 从 swag 官网下载:
```shell script
$ go get -u github.com/swaggo/swag/cmd/swag
```

> **选项 2:** 
>
> 通过 [RK CLI](https://github.com/rookie-ninja/rk)
```shell script
# Install RK CMD
$ go get -u github.com/rookie-ninja/rk/cmd/rk

# Install swag with rk
$ rk install swag
```

### 2.添加 Swagger 注释
```go
package main

import (
	"context"
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// @title RK Swagger for Gin
// @version 1.0
// @description This is a greeter service with rk-boot.

// Application entrance.
func main() {...}

// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(ctx *gin.Context) {...}
```

### 3.生成 swagger 参数文件
```shell script
$ swag init

# swagger.json, swagger.yaml, docs.go files will be generated under ./docs folder.
$ tree
.
├── boot.yaml
├── docs
│   ├── docs.go
│   ├── swagger.json
│   └── swagger.yaml
├── go.mod
├── go.sum
└── main.go

1 directory, 7 files
```

### 4.添加 Swagger 参数文件路径到 boot.yaml
为了能让启动器能够识别 Swagger 参数文件，我们需要在 boot.yaml 文件中添加 **gin.sw.jsonPath**。
```yaml
---
gin:
  - name: greeter
    ...
    sw:
      enabled: true
      jsonPath: "docs"  # Boot will look for swagger config files from this folder
    ...
```

### 5.验证
> **Swagger 界面:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/gin-golang/gin-sw-api.png)

> **TV 界面:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/gin-golang/gin-tv-api.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
