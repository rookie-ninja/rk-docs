---
title: "CSRF 中间件"
linkTitle: "CSRF 中间件"
weight: 17
description: >
  启动 CSRF 中间件。
---

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## CSRF 选项
| 名字                                  | 描述                                                          | 类型       | 默认值                   |
|-------------------------------------|-------------------------------------------------------------|----------|-----------------------|
| grpc.middleware.csrf.enabled        | 启动 CSRF 中间件                                                 | boolean  | false                 |
| grpc.middleware.csrf.ignore         | 局部选项，忽略 API 路径                                              | []string | []                    |
| grpc.middleware.csrf.tokenLength    | 生成的 CSRF Token 的长度                                          | int      | 32                    |
| grpc.middleware.csrf.tokenLookup    | 寻找 CSRF Token 的方法，请参考代码注释                                   | string   | "header:X-CSRF-Token" |
| grpc.middleware.csrf.cookieName     | CSRF cookie 名字                                              | string   | _csrf                 |
| grpc.middleware.csrf.cookieDomain   | CSRF cookie 的 Domain 名称                                     | string   | ""                    |
| grpc.middleware.csrf.cookiePath     | CSRF cookie 路径                                              | string   | ""                    |
| grpc.middleware.csrf.cookieMaxAge   | CSRF cookie 的 MaxAge（秒）                                     | int      | 86400                 |
| grpc.middleware.csrf.cookieHttpOnly | CSRF cookie 是否只支持 HTTP                                      | bool     | false                 |
| grpc.middleware.csrf.cookieSameSite | 描述 CSRF cookie 的 SameSite 模式。选项: lax, strict, none, default | string   | default               |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
    middleware:
      csrf:
        enabled: true
```

### 2.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "net/http"
)

func main() {
  boot := rkboot.NewBoot()

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // register grpc
  entry := rkgrpc.GetGrpcEntry("greeter")
  entry.GwMux.HandlePath("GET", "/v1/greeter", func(w http.ResponseWriter, r *http.Request, pathParams map[string]string) {
    w.Write([]byte(fmt.Sprintf("Hello %s!", r.URL.Query().Get("name"))))
  })

  entry.GwMux.HandlePath("POST", "/v1/greeter", func(w http.ResponseWriter, r *http.Request, pathParams map[string]string) {
    w.Write([]byte(fmt.Sprintf("Hello %s!", r.URL.Query().Get("name"))))
  })

  // Wait for shutdown sig
  boot.WaitForShutdownSig(context.TODO())
}
```

### 3.验证
- 发送 GET 请求, 对于 GET 请求，一个新的 Cookie 将会返回。

```bash
$ curl -X GET -vs "localhost:8080/v1/greeter?name=rk-dev"
...
< HTTP/1.1 200 OK
< Set-Cookie: _csrf=FpLSjFbcXoEFfRsWxPLDnJObCsNVlgTe; Expires=Mon, 18 Apr 2022 12:24:08 GMT
< Vary: Cookie
< Date: Sun, 17 Apr 2022 12:24:08 GMT
< Content-Length: 13
< Content-Type: text/plain; charset=utf-8
< 
Hello rk-dev!
```

- 发送 POST 请求，通过 CSRF 验证。

```bash
$ curl -X POST -v --cookie "_csrf=my-test-csrf-token" -H "X-CSRF-Token:my-test-csrf-token" "localhost:8080/v1/greeter?name=rk-dev"
...
> Cookie: _csrf=my-test-csrf-token
> X-CSRF-Token:my-test-csrf-token
> 
< HTTP/1.1 200 OK
< Set-Cookie: _csrf=my-test-csrf-token; Expires=Mon, 18 Apr 2022 12:25:01 GMT
< Vary: Cookie
< Date: Sun, 17 Apr 2022 12:25:01 GMT
< Content-Length: 13
< Content-Type: text/plain; charset=utf-8
Hello rk-dev!
```

- 发送 POST 请求，携带非法 CSRF。

```bash
$ curl -X POST -v -H "X-CSRF-Token:my-test-csrf-token" "localhost:8080/v1/greeter?name=rk-dev"
...
> X-CSRF-Token:my-test-csrf-token
> 
< HTTP/1.1 403 Forbidden
< Date: Sun, 17 Apr 2022 12:25:50 GMT
< Content-Length: 87
< Content-Type: text/plain; charset=utf-8
< 
* Connection #0 to host localhost left intact
{"error":{"code":403,"status":"Forbidden","message":"Invalid csrf token","details":[]}}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)