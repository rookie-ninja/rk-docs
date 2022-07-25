Enable CSRF Middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## CSRF options
| options                             | description                        | type     | default |
|-------------------------------------|----------------------------------------------------|----------|-----------------------|
| gf.middleware.csrf.enabled          | Enable CSRF middleware                             | boolean  | false                 |
| gf.middleware.csrf.ignore           | Ignore by path                                     | []string | []                    |
| gf.middleware.csrf.tokenLength      | Length of CSRF Token                               | int      | 32                    |
| gf.middleware.csrf.tokenLookup      | Scheme of CSRF Token, See code comment for details | string   | "header:X-CSRF-Token" |
| gf.middleware.csrf.cookieName        | Name of cookie                                     | string   | _csrf                 |
| gf.middleware.csrf.cookieDomain   | Name of cookie domain                              | string   | ""                    |
| gf.middleware.csrf.cookiePath     | Name of cookie path                                | string   | ""                    |
| gf.middleware.csrf.cookieMaxAge   | Name of cookie max age                             | int      | 86400                 |
| gf.middleware.csrf.cookieHttpOnly | Support http only                                  | bool     | false                 |
| gf.middleware.csrf.cookieSameSite | SameSite mode, options: lax, strict, none, default | string   | default               |

## Quick start
### 1.Create boot.yaml
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

### 2.Create main.go
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

### 3.Validation
- GET request, a new cookie is expected

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

- POST request

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

- POST request, invalid CSRF。

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