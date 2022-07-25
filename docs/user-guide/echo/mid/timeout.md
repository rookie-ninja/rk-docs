Enable timeout middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## Options
| options                                   | description               | type     | default |
|-------------------------------------------|---------------------------|----------|---------|
| echo.middleware.timeout.enabled           | Enable timeout middleware | boolean  | false   |
| echo.middleware.timeout.ignore            | Ignore by path            | []string | []      |
| echo.middleware.timeout.timeoutMs         | Timeouts in milliseconds  | int      | 5000    |
| echo.middleware.timeout.paths.path        | API path                  | string   | ""      |
| echo.interceptors.timeout.paths.timeoutMs | Timeouts in milliseconds  | int      | 5000    |

## Quick start
### 1.Create boot.yaml
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

### 3.Validate
> Send request

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
![](../../../img/user-guide/cheers.png)