Enable secure middleware.

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## Secure options
| options                     | description                        | type     | default |
|---------------------------------------------|------------------------------------|----------|-----------------|
| echo.middleware.secure.enabled               | Enable secure middleware           | boolean  | false           |
| echo.middleware.secure.ignore  | Ignore by path                     | []string | []    |
| echo.middleware.secure.xssProtection         | X-XSS-Protection                   | string   | "1; mode=block" |
| echo.middleware.secure.contentTypeNosniff    | X-Content-Type-Options             | string   | nosniff         |
| echo.middleware.secure.xFrameOptions         | X-Frame-Options                    | string   | SAMEORIGIN      |
| echo.middleware.secure.hstsMaxAge            | Strict-Transport-Security          | int      | 0               |
| echo.middleware.secure.hstsExcludeSubdomains | HSTS SubDomains                    | bool     | false           |
| echo.middleware.secure.hstsPreloadEnabled    | Enable HSTS preloading             | bool     | false           |
| echo.middleware.secure.contentSecurityPolicy | Content-Security-Policy            | string   | ""              |
| echo.middleware.secure.cspReportOnly         | Content-Security-Policy-Report-Only | bool     | false           |
| echo.middleware.secure.referrerPolicy        | Referrer-Policy                    | string   | ""              |

## Quick start
### 1.Create boot.yaml
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

### 2.Create main.go
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

### 3.Validate
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
![](../../../img/user-guide/cheers.png)