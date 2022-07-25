启动 CSRF 中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## CSRF 选项
| 名字                                   | 描述                                               | 类型 | 默认值 |
|--------------------------------------|--------------------------------------------------| ------ | ------ |
| gf.middleware.csrf.enabled          | 启动 CSRF 中间件                                      | boolean | false |
| gf.middleware.csrf.ignore           | 局部选项，忽略 API 路径                               | []string | []                   |
| gf.middleware.csrf.tokenLength    | 生成的 CSRF Token 的长度                               | int | 32 |
| gf.middleware.csrf.tokenLookup    | 寻找 CSRF Token 的方法，请参考代码注释                        | string | "header:X-CSRF-Token" |
| gf.middleware.csrf.cookieName     | CSRF cookie 名字                                   | string | _csrf |
| gf.middleware.csrf.cookieDomain   | CSRF cookie 的 Domain 名称                          | string | "" |
| gf.middleware.csrf.cookiePath     | CSRF cookie 路径                                   | string | "" |
| gf.middleware.csrf.cookieMaxAge   | CSRF cookie 的 MaxAge（秒）                          | int | 86400 |
| gf.middleware.csrf.cookieHttpOnly | CSRF cookie 是否只支持 HTTP                           | bool | false |
| gf.middleware.csrf.cookieSameSite | 描述 CSRF cookie 的 SameSite 模式。选项: lax, strict, none, default | string | default |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      csrf:
        enabled: true
#        ignore: [""]
#        tokenLength: 32
#        tokenLookup: "header:X-CSRF-Token"
#        cookieName: "_csrf"
#        cookieDomain: ""
#        cookiePath: ""
#        cookieMaxAge: 86400
#        cookieHttpOnly: false
#        cookieSameSite: "default"
```

### 2.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gf/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgf.GetGfEntry("greeter")
  entry.Server.BindHandler("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *ghttp.Request) {
  ctx.Response.WriteHeader(http.StatusOK)
  ctx.Response.WriteJson(&GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name").String()),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.验证
- 发送 GET 请求, 对于 GET 请求，一个新的 Cookie 将会返回。

```bash
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

```bash
$ curl -X POST -v --cookie "_csrf=my-test-csrf-token" -H "X-CSRF-Token:my-test-csrf-token" localhost:8080/v1/greeter
...
> Accept: */*
> Cookie: _csrf=my-test-csrf-token
> X-CSRF-Token:my-test-csrf-token
> 
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=utf-8
< Set-Cookie: _csrf=my-test-csrf-token; Expires=Sun, 17 Apr 2022 12:41:46 GMT
< Vary: Cookie
< Date: Sat, 16 Apr 2022 12:41:46 GMT
< Content-Length: 27
< 
{"Message":"Hello rk-dev!"}
```

- 发送 POST 请求，携带非法 CSRF。

```bash
$ curl -X POST -v -H "X-CSRF-Token:my-test-csrf-token" localhost:8080/v1/greeter
...
> X-CSRF-Token:my-test-csrf-token
>
< HTTP/1.1 403 Forbidden
< Content-Type: application/json; charset=UTF-8
< Date: Sat, 16 Apr 2022 12:42:27 GMT
< Content-Length: 114
<
{"error":{"code":403,"status":"Forbidden","message":"Invalid csrf token","details":[null]}}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)