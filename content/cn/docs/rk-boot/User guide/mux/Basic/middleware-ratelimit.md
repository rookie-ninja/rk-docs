---
title: "限流中间件"
linkTitle: "限流中间件"
weight: 10
description: >
  启动限流中间件。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-mux
```

## 选项
| 名字                                       | 描述                   | 类型      | 默认值         |
|------------------------------------------|----------------------|---------|-------------|
| mux.middleware.rateLimit.enabled         | 启动限流中间件              | boolean | false       |
| mux.middleware.rateLimit.ignore            | 局部选项，忽略 API 路径       | []string | []      |
| mux.middleware.rateLimit.algorithm       | 限流算法， 支持 leakyBucket | string  | leakyBucket |
| mux.middleware.rateLimit.reqPerSec       | 全局限流值                | int     | 1000000     |
| mux.middleware.rateLimit.paths.path      | 访问路径                 | string  | ""          |
| mux.middleware.rateLimit.paths.reqPerSec | 基于访问路径的限流值           | int     | 1000000     |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
mux:
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

### 2.创建 main.go
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

// Greeter handler
// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(writer http.ResponseWriter, req *http.Request) {
  rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.验证
> 发送请求

```shell script
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
![](/rk-boot/user-guide/cheers.png)