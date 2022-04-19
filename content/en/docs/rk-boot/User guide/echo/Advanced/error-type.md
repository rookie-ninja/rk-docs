---
title: "Error type"
linkTitle: "Error type"
weight: 8
description: >
  What is the best way to return an RPC error?
---

## Overview
It is always not easy to define a user-friendly API. However, errors could happen anywhere.

In order to return standard error type to user, it is **important** to define an error type.

By default, panic middleware/interceptor will be attached which will catch panic and return internal server error as bellow:

```json
{
  "error":{
    "code":500,
    "status":"Internal Server Error",
    "message":"Panic manually!",
    "details":[]
  }
}
```

## Quick start
### 1.Install

```shell script
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-echo
```

### 2.Create boot.yaml
```yaml
echo:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.Create main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-entry/v2/error"
  "github.com/rookie-ninja/rk-echo/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Add shutdown hook function
  boot.AddShutdownHookFunc("shutdown-hook", func() {
    fmt.Println("shutting down")
  })

  // Register handler
  echoEntry := rkecho.GetEchoEntry("greeter")
  echoEntry.Echo.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx echo.Context) error {
  err := rkerror.New(http.StatusAlreadyReported, "Trigger manually!",
    "This is detail.",
    false, -1,
    0.1)

  return ctx.JSON(http.StatusAlreadyReported, err)
}

type GreeterResponse struct {
  Message string
}
```


```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{
    "error":{
        "code":208,
        "status":"Already Reported",
        "message":"Trigger manually!",
        "details":[
            "This is detail.",
            false,
            -1,
            0.1
        ]
    }
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
