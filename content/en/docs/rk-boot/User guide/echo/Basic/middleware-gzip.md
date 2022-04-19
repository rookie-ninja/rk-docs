---
title: "Middleware gzip"
linkTitle: "Middleware gzip"
weight: 13
description: >
  Enable gzip middleware
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## gzip options
| options                     | description                        | type     | default |
|-----------------------------|--------------------------------------------------------------------------------------|----------|--------------------|
| echo.middleware.gzip.enabled | Enable gzip middleware                                                               | boolean  | false              |
| echo.middleware.gzip.ignore  | Ignore by path                                                                       | []string | []                 |
| echo.middleware.gzip.level   | options: noCompression, bestSpeed, bestCompression, defaultCompression, huffmanOnly. | string   | defaultCompression |

## Quick start
### 1.Create boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      gzip:
        enabled: true
        level: bestSpeed
#        ignore: [""]
```

### 2.Create main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-echo/boot"
  "net/http"
  "io"
  "strings"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  echoEntry := rkecho.GetEchoEntry("greeter")
  echoEntry.Echo.POST("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx echo.Context) error {
  buf := new(strings.Builder)
  io.Copy(buf, ctx.Request.Body)
  
  return ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Received %s!", buf.String()),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.Validate
> Send request

```shell script
$ echo 'this is message' | gzip | curl --compressed --data-binary @- -H "Content-Encoding: gzip" -H "Accept-Encoding: gzip" localhost:8080/v1/greeter
{"Message":"Received this is message\n!"}%
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)