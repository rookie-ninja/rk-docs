启动限流中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## 选项
| 名字                                       | 描述                   | 类型      | 默认值         |
|------------------------------------------|----------------------|---------|-------------|
| echo.middleware.rateLimit.enabled         | 启动限流中间件              | boolean | false       |
| echo.middleware.rateLimit.ignore            | 局部选项，忽略 API 路径       | []string | []      |
| echo.middleware.rateLimit.algorithm       | 限流算法， 支持 leakyBucket | string  | leakyBucket |
| echo.middleware.rateLimit.reqPerSec       | 全局限流值                | int     | 1000000     |
| echo.middleware.rateLimit.paths.path      | 访问路径                 | string  | ""          |
| echo.middleware.rateLimit.paths.reqPerSec | 基于访问路径的限流值           | int     | 1000000     |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      rateLimit:
        enabled: true
        paths:
          - path: "/v1/greeter"
            reqPerSec: 0
#        algorithm: "leakyBucket"
#        reqPerSec: 100
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
> 发送请求

```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{
    "error":{
        "code":429,
        "status":"Too Many Requests",
        "message":"",
        "details":[
            "slow down your request"
        ]
    }
}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)