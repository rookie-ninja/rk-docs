---
title: "错误类型"
linkTitle: "错误类型"
weight: 8
description: >
  rk-boot 推荐的 RPC 标准错误类型。
---

## 概述
设计一个合理的 API 是一件不容易的事情，同时，API 还会产生各种不同的错误。

为了能让 API 使用者对于 API 的错误有一个清晰的视图，定义一个标准的 RPC 错误类型是**非常重要**的事情。

当使用启动器的时候，panic 拦截器会默认加载到服务中，拦截器会从 panic 中恢复，并且返回[内部错误]给用户。

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

## 快速开始
### 1.安装

```shell script
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-mux
```

### 2.创建 boot.yaml
```yaml
mux:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.创建 main.go
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
