启动超时中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## 超时选项
| 名字                                       | 描述             | 类型       | 默认值   |
|------------------------------------------|----------------|----------|-------|
| echo.middleware.timeout.enabled           | 启动超时中间件        | boolean  | false |
| echo.middleware.timeout.ignore            | 局部选项，忽略 API 路径 | []string | []    |
| echo.middleware.timeout.timeoutMs         | 超时时间，毫秒        | int      | 5000  |
| echo.middleware.timeout.paths.path        | 路径             | string   | ""    |
| echo.interceptors.timeout.paths.timeoutMs | 基于访问路径的超时时间，毫秒 | int      | 5000  |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
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
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-echo/boot"
  "net/http"
  "time"
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
  time.Sleep(10*time.Millisecond)

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
        "code":408,
        "status":"Request Timeout",
        "message":"",
        "details":[]
    }
}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)