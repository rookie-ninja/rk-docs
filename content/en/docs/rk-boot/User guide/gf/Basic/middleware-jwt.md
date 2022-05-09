---
title: "Middleware JWT"
linkTitle: "Middleware JWT"
weight: 14
description: >
  Enable JWT middleware
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## Options
| options                                  | description                        | type     | default                |
|------------------------------------------|--------------------------------| ------ |------------------------|
| gf.middleware.jwt.enabled                | Enable JWT middleware                     | boolean | false                  |
| gf.middleware.jwt.ignore               | Ignore by path                                                                       | []string | []                     |
| gf.middleware.jwt.signerEntry          | SignerEntry name                 | string | ""                     |
| gf.middleware.jwt.symmetric.algorithm  | Symmetric algorithm, options: HS256, HS384, HS512                         | string | ""                     |
| gf.middleware.jwt.symmetric.token      | Symmetric token                         | string | ""                     |
| gf.middleware.jwt.symmetric.tokenPath  | Symmetric token path                     | string | ""                     |
| gf.middleware.jwt.asymmetric.algorithm | Asymmetric algorithm, options: RS256, RS384, RS512, ES256, ES384, ES512                        | string | ""                     |
| gf.middleware.jwt.tokenLookup          | JWT Token format，see example bellow | string | "header:Authorization" |
| gf.middleware.jwt.authScheme           | Auth Scheme                 | string | Bearer                 |

**tokenLookup**

```
// Optional. Default value "header:Authorization".
// Possible values:
// - "header:<name>"
// - "query:<name>"
// Multiply sources example:
// - "header: Authorization,cookie: myowncookie"
```

## Quick start
### 1.Create boot.yaml
```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      jwt:
        enabled: true
        symmetric:
          algorithm: HS256
          token: "my-secret"
#          tokenPath: ""
#        signerEntry: ""
#        asymmetric:
#          algorithm: ""
#          privateKey: ""
#          privateKeyPath: ""
#          publicKey: ""
#          publicKeyPath: ""
#        tokenLookup: "header:<name>"
#        authScheme: "Bearer"
```

### 2.Create main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/gogf/gf/v2/net/ghttp"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gf/boot"
  "github.com/rookie-ninja/rk-gf/middleware/context"
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
  // Get JWT token
  rkgfctx.GetJwtToken(ctx)
  
  ctx.Response.WriteHeader(http.StatusOK)
  ctx.Response.WriteJson(&GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name").String()),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.Validate
- Valid JWT Token

```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.EpM5XBzTJZ4J8AfoJEcJrjth8pfH28LWdjLo90sYb9g"
{"Message":"Hello rk-dev!"}
```

- Invalid JWT Token
```shell script
$ curl localhost:8080/v1/greeter -H "Authorization: Bearer invalid-jwt-token"
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"Invalid or expired jwt",
        "details":[]
    }
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)