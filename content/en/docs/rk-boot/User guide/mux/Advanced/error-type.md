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
$ go get github.com/rookie-ninja/rk-mux
```

### 2.Create boot.yaml
```yaml
mux:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.Create main.go
```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
  "context"
  _ "embed"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-mux/boot"
  "github.com/rookie-ninja/rk-mux/middleware"
  "net/http"
)

// @title RK Swagger for Mux
// @version 1.0
// @description This is a greeter service with rk-boot.
func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Get MuxEntry
  muxEntry := rkmux.GetMuxEntry("greeter")
  // Use *mux.Router adding handler.
  muxEntry.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(writer http.ResponseWriter, req *http.Request) {
  err := rkerror.New(http.StatusAlreadyReported, "Trigger manually!",
    "This is detail.",
    false, -1,
    0.1)
  rkmuxmid.WriteJson(writer, http.StatusAlreadyReported, err)
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
