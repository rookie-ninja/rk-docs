---
title: "安全中间件"
linkTitle: "安全中间件"
weight: 16
description: >
  启动安全中间件。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-mux
```

## Secure 选项
| 名字                                          | 描述                                  | 类型       | 默认值             |
|---------------------------------------------|-------------------------------------|----------|-----------------|
| mux.middleware.secure.enabled               | 启动安全中间件                             | boolean  | false           |
| mux.middleware.secure.ignore                | 局部选项，忽略 API 路径                      | []string | []              |
| mux.middleware.secure.xssProtection         | X-XSS-Protection                    | string   | "1; mode=block" |
| mux.middleware.secure.contentTypeNosniff    | X-Content-Type-Options              | string   | nosniff         |
| mux.middleware.secure.xFrameOptions         | X-Frame-Options                     | string   | SAMEORIGIN      |
| mux.middleware.secure.hstsMaxAge            | Strict-Transport-Security           | int      | 0               |
| mux.middleware.secure.hstsExcludeSubdomains | HSTS SubDomains                     | bool     | false           |
| mux.middleware.secure.hstsPreloadEnabled    | 开启 HSTS 预加载。                        | bool     | false           |
| mux.middleware.secure.contentSecurityPolicy | Content-Security-Policy             | string   | ""              |
| mux.middleware.secure.cspReportOnly         | Content-Security-Policy-Report-Only | bool     | false           |
| mux.middleware.secure.referrerPolicy        | Referrer-Policy                     | string   | ""              |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
mux:
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
```shell script
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
![](/rk-boot/user-guide/cheers.png)