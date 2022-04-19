---
title: "Middleware auth"
linkTitle: "Middleware auth"
weight: 10
description: >
  Enable auth middleware.
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## Options
| options                      | description                        | type     | default |
|------------------------------|------------------------------------|----------|---------|
| echo.middleware.auth.enabled | Enable auth middleware             | boolean  | false   |
| echo.middleware.auth.ignore  | Ignore by path                     | []string | []      |
| echo.middleware.auth.basic   | Basic Auth info，format：<user:pass> | []string | []      |
| echo.middleware.auth.apiKey  | X-API-Key                          | []string | []      |

## Quick start
### 1.Create boot.yaml
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

### 4.X-API-Key
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

### 5.Ignore
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