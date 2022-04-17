---
title: "超时中间件"
linkTitle: "超时中间件"
weight: 11
description: >
  启动超时中间件。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

## 超时选项
| 名字                                       | 描述             | 类型       | 默认值   |
|------------------------------------------|----------------|----------|-------|
| gin.middleware.timeout.enabled           | 启动超时中间件        | boolean  | false |
| gin.middleware.timeout.ignore            | 局部选项，忽略 API 路径 | []string | []    |
| gin.middleware.timeout.timeoutMs         | 超时时间，毫秒        | int      | 5000  |
| gin.middleware.timeout.paths.path        | 路径             | string   | ""    |
| gin.interceptors.timeout.paths.timeoutMs | 基于访问路径的超时时间，毫秒 | int      | 5000  |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      timeout:
        enabled: true
#        ignore: [""]
        timeoutMs: 5000
        paths:
          - path: "/v1/greeter"
            timeoutMs: 1
```

### 2.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gin/v2/boot"
  "net/http"
  "time"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgin.GetGinEntry("greeter")
  entry.Router.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *gin.Context) {
  time.Sleep(10*time.Millisecond)

  ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.验证
> 发送请求

```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{
    "error":{
        "code":408,
        "status":"Request Timeout",
        "message":"",
        "details":[]
    }
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)