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
go get github.com/rookie-ninja/rk-gin/v2
```

## Options
| options                     | description                        | type     | default                |
|----------------------------------------|--------------------------------| ------ |------------------------|
| gin.middleware.jwt.enabled             | Enable JWT middleware                     | boolean | false                  |
| gin.middleware.jwt.ignore  | Ignore by path                                                                       | []string | []                     |
| gin.middleware.jwt.skipVerify           | Skip verify JWT token                                                   | boolean  | false                  |
| gin.middleware.jwt.signerEntry         | SignerEntry name                 | string | ""                     |
| gin.middleware.jwt.symmetric.algorithm | Symmetric algorithm, options: HS256, HS384, HS512                         | string | ""                     |
| gin.middleware.jwt.symmetric.token     | Symmetric token                         | string | ""                     |
| gin.middleware.jwt.symmetric.tokenPath | Symmetric token path                     | string | ""                     |
| gin.middleware.jwt.asymmetric.algorithm| Asymmetric algorithm, options: RS256, RS384, RS512, ES256, ES384, ES512                        | string | ""                     |
| gin.middleware.jwt.tokenLookup         | JWT Token format，see example bellow | string | "header:Authorization" |
| gin.middleware.jwt.authScheme          | Auth Scheme                 | string | Bearer                 |

**tokenLookup** format

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
gin:
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
#        ignore: [ "" ]
#        skipVerify: false
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
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gin/v2/boot"
  "github.com/rookie-ninja/rk-gin/v2/middleware/context"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgin.GetGinEntry("greeter")
  entry.Router.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *gin.Context) {
  // Get JWT token
  rkginctx.GetJwtToken(ctx)
  
  ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.Validation
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