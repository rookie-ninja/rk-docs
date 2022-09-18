Enable secure middleware.

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

## Secure options
| options                     | description                        | type     | default |
|---------------------------------------------|------------------------------------|----------|-----------------|
| gin.middleware.secure.enabled               | Enable secure middleware           | boolean  | false           |
| gin.middleware.secure.ignore  | Ignore by path                     | []string | []    |
| gin.middleware.secure.xssProtection         | X-XSS-Protection                   | string   | "1; mode=block" |
| gin.middleware.secure.contentTypeNosniff    | X-Content-Type-Options             | string   | nosniff         |
| gin.middleware.secure.xFrameOptions         | X-Frame-Options                    | string   | SAMEORIGIN      |
| gin.middleware.secure.hstsMaxAge            | Strict-Transport-Security          | int      | 0               |
| gin.middleware.secure.hstsExcludeSubdomains | HSTS SubDomains                    | bool     | false           |
| gin.middleware.secure.hstsPreloadEnabled    | Enable HSTS preloading             | bool     | false           |
| gin.middleware.secure.contentSecurityPolicy | Content-Security-Policy            | string   | ""              |
| gin.middleware.secure.cspReportOnly         | Content-Security-Policy-Report-Only | bool     | false           |
| gin.middleware.secure.referrerPolicy        | Referrer-Policy                    | string   | ""              |

## Quick start
### 1.Create boot.yaml
```yaml
---
gin:
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
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gin/v2/boot"
  "net/http"
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
  ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
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
![](../../../../img/user-guide/cheers.png)