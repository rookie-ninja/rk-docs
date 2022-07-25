Enable ratelimit middleware.

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## Option
| options                                   | description                   | type     | default     |
|-------------------------------------------|-------------------------------|----------|-------------|
| echo.middleware.rateLimit.enabled         | Enable ratelimit middleware   | boolean  | false       |
| echo.middleware.rateLimit.ignore          | Ignore by path                | []string | []          |
| echo.middleware.rateLimit.algorithm       | leakyBucket is supported only | string   | leakyBucket |
| echo.middleware.rateLimit.reqPerSec       | global limit                  | int      | 1000000     |
| echo.middleware.rateLimit.paths.path      | API path                      | string   | ""          |
| echo.middleware.rateLimit.paths.reqPerSec | limit value of path           | int      | 1000000     |

## Quick start
### 1.Create boot.yaml
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
> Send request

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
![](../../../img/user-guide/cheers.png)