启动安全中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## Secure 选项
| 名字                                          | 描述                                  | 类型       | 默认值             |
|---------------------------------------------|-------------------------------------|----------|-----------------|
| echo.middleware.secure.enabled               | 启动安全中间件                             | boolean  | false           |
| echo.middleware.secure.ignore                | 局部选项，忽略 API 路径                      | []string | []              |
| echo.middleware.secure.xssProtection         | X-XSS-Protection                    | string   | "1; mode=block" |
| echo.middleware.secure.contentTypeNosniff    | X-Content-Type-Options              | string   | nosniff         |
| echo.middleware.secure.xFrameOptions         | X-Frame-Options                     | string   | SAMEORIGIN      |
| echo.middleware.secure.hstsMaxAge            | Strict-Transport-Security           | int      | 0               |
| echo.middleware.secure.hstsExcludeSubdomains | HSTS SubDomains                     | bool     | false           |
| echo.middleware.secure.hstsPreloadEnabled    | 开启 HSTS 预加载。                        | bool     | false           |
| echo.middleware.secure.contentSecurityPolicy | Content-Security-Policy             | string   | ""              |
| echo.middleware.secure.cspReportOnly         | Content-Security-Policy-Report-Only | bool     | false           |
| echo.middleware.secure.referrerPolicy        | Referrer-Policy                     | string   | ""              |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      secure:
        enabled: true
#        ignore: [""]
#        xssProtection: ""
#        contentTypeNosniff: ""
#        xFrameOptions: ""
#        hstsMaxAge: 0
#        hstsExcludeSubdomains: false
#        hstsPreloadEnabled: false
#        contentSecurityPolicy: ""
#        cspReportOnly: false
#        referrerPolicy: ""

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
```bash
$ curl -vs "localhost:8080/v1/greeter?name=rk-dev"
...
< Content-Type: application/json; charset=utf-8
< X-Content-Type-Options: nosniff
< X-Frame-Options: SAMEORIGIN
< X-Xss-Protection: 1; mode=block
< Date: Sat, 16 Apr 2022 12:35:05 GMT
< Content-Length: 27
...
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)