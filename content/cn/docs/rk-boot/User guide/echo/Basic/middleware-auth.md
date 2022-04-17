---
title: "权限中间件"
linkTitle: "权限中间件"
weight: 10
description: >
  启动权限中间件。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## 权限选项
| 名字                          | 描述                           | 类型       | 默认值   |
|-----------------------------|------------------------------|----------|-------|
| echo.middleware.auth.enabled | 启动权限中间件                      | boolean  | false |
| echo.middleware.auth.ignore  | 局部选项，忽略 API 路径               | []string | []    |
| echo.middleware.auth.basic   | Basic Auth 信息，格式：<user:pass> | []string | []    |
| echo.middleware.auth.apiKey  | X-API-Key                    | []string | []    |
| echo.middleware.auth.ignore  | 提供字符串前缀，中间件会忽略包含这些字符串的请求路径   | []string | []    |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      auth:
        enabled: true
        basic: ["user:pass"]
#        ignore: [""]
#        apiKey:
#          - "keys"
```

### 2.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-echo/boot"
  "net/http"
)

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

func Greeter(ctx echo.Context) error {
  return ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.QueryParam("name")),
  })
}

type GreeterResponse struct {
  Message string
}

```

### 3.验证
```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev"
# This is RK style error code if unauthorized
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"Missing authorization, provide one of bellow auth header:[Basic Auth]",
        "details":[]
    }
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 4.X-API-Key 类型授权
```yaml
---
echo:
  - name: greeter
    ...
    middleware:
      auth:
        enabled: true
        apiKey: ["token"]
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 5.忽略请求路径
```yaml
---
echo:
  - name: greeter
    ...
    interceptors:
      auth:
        enabled: true
        basic: ["user:pass"]
        ignorePrefix: ["/v1/greeter"]
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)