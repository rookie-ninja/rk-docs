Enable CSRF middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-grpc/v2
```

## CSRF options
| options                     | description                        | type     | default |
|------------------------------------|----------------------------------------------------|----------|-----------------------|
| grpc.middleware.csrf.enabled        | Enable CSRF middleware                             | boolean  | false                 |
| grpc.middleware.csrf.ignore         | Ignore by path                                     | []string | []                    |
| grpc.middleware.csrf.tokenLength    | Length of CSRF Token                               | int      | 32                    |
| grpc.middleware.csrf.tokenLookup    | Scheme of CSRF Token, See code comment for details | string   | "header:X-CSRF-Token" |
| grpc.middleware.csrf.cookieName     | Name of cookie                                     | string   | _csrf                 |
| grpc.middleware.csrf.cookieDomain   | Name of cookie domain                              | string   | ""                    |
| grpc.middleware.csrf.cookiePath     | Name of cookie path                                | string   | ""                    |
| grpc.middleware.csrf.cookieMaxAge   | Name of cookie max age                             | int      | 86400                 |
| grpc.middleware.csrf.cookieHttpOnly | Support http only                                  | bool     | false                 |
| grpc.middleware.csrf.cookieSameSite | SameSite mode, options: lax, strict, none, default | string   | default               |

## Quick start
### 1.Create boot.yaml
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

### 2.Create main.go
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

### 3.Validate
- Send GET request where a new cookie would be returned.

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

- Send POST request with valid CSRF

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

- Send POST request with invalid CSRF

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