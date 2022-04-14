---
title: "CSRF 拦截器"
linkTitle: "CSRF 拦截器"
weight: 17
description: >
  启动 CSRF 拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-grpc
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 grpc 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.name | grpc 服务名称 | string | N/A |
| grpc.port | grpc 服务端口 | integer | nil, 服务不会启动 |
| grpc.enabled | grpc 服务启动开关 | bool | false |
| grpc.description | grpc 服务的描述 | string | "" |

## CSRF 选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.csrf.enabled | 启动 CSRF 拦截器 | boolean | false |
| grpc.interceptors.csrf.tokenLength | 生成的 CSRF Token 的长度。 | int | 32 |
| grpc.interceptors.csrf.tokenLookup | 拦截器寻找 CSRF Token 的方法，请参考代码注释。| string | "header:X-CSRF-Token" |
| grpc.interceptors.csrf.cookieName | CSRF cookie 名字。 | string | _csrf |
| grpc.interceptors.csrf.cookieDomain | CSRF cookie 的 Domain 名称。 | string | "" |
| grpc.interceptors.csrf.cookiePath | CSRF cookie 路径。 | string | "" |
| grpc.interceptors.csrf.cookieMaxAge | CSRF cookie 的 MaxAge（秒）。 | int | 86400 |
| grpc.interceptors.csrf.cookieHttpOnly | CSRF cookie 是否只支持 HTTP。 | bool | false |
| grpc.interceptors.csrf.cookieSameSite | 描述 CSRF cookie 的 SameSite 模式。选项: lax, strict, none, default | string | default |
| grpc.interceptors.csrf.ignorePrefix | 可忽略的路径。 | []string | [] |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter                     # Required
    port: 8080                        # Required
    enabled: true                     # Required
    commonService:
      enabled: true                   # Optional, default: false
    interceptors:
      csrf:
        enabled: true                 # Optional, default: false
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
	"fmt"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-grpc/boot"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Register handler in grpc-gateway
	grpcEntry := boot.GetEntry("greeter").(*rkgrpc.GrpcEntry)
	grpcEntry.GwMux.HandlePath("GET", "/v1/greeter", func(w http.ResponseWriter, r *http.Request, pathParams map[string]string) {
		w.Write([]byte(fmt.Sprintf("Hello %s!", r.URL.Query().Get("name"))))
	})

	grpcEntry.GwMux.HandlePath("POST", "/v1/greeter", func(w http.ResponseWriter, r *http.Request, pathParams map[string]string) {
		w.Write([]byte(fmt.Sprintf("Hello %s!", r.URL.Query().Get("name"))))
	})

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 3.验证
- 发送 GET 请求, 对于 GET 请求，一个新的 Cookie 将会返回。

```shell script
$ curl -X GET -vs localhost:8080/v1/greeter
  ...
  < HTTP/1.1 200 OK
  < Content-Type: application/json; charset=UTF-8
  < Set-Cookie: _csrf=WyOJLwzhfUGAMDHglkuIRucdpalxolWg; Expires=Mon, 06 Dec 2021 18:38:45 GMT
  < Vary: Cookie
  < Date: Sun, 05 Dec 2021 18:38:45 GMT
  < Content-Length: 22
  <
  {"Message":"Hello !"}
```

- 发送 POST 请求，通过 CSRF 验证。

```shell script
$ curl -X POST -v --cookie "_csrf=my-test-csrf-token" -H "X-CSRF-Token:my-test-csrf-token" localhost:8080/v1/greeter
  ...
  > Cookie: _csrf=my-test-csrf-token
  > X-CSRF-Token:my-test-csrf-token
  >
  < HTTP/1.1 200 OK
  < Content-Type: application/json; charset=UTF-8
  < Set-Cookie: _csrf=my-test-csrf-token; Expires=Mon, 06 Dec 2021 18:40:13 GMT
  < Vary: Cookie
  < Date: Sun, 05 Dec 2021 18:40:13 GMT
  < Content-Length: 22
  <
  {"Message":"Hello !"}
```

- 发送 POST 请求，携带非法 CSRF。

```shell script
$ curl -X POST -v -H "X-CSRF-Token:my-test-csrf-token" localhost:8080/v1/greeter
  ...
  > X-CSRF-Token:my-test-csrf-token
  >
  < HTTP/1.1 403 Forbidden
  < Content-Type: application/json; charset=UTF-8
  < Date: Sun, 05 Dec 2021 18:41:00 GMT
  < Content-Length: 92
  <
  {"error":{"code":403,"status":"Forbidden","message":"invalid csrf token","details":[null]}}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)