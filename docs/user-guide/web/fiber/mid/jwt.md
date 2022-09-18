Enable JWT middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## Options
| options                                   | description                                                             | type     | default                |
|-------------------------------------------|-------------------------------------------------------------------------|----------|------------------------|
| fiber.middleware.jwt.enabled              | Enable JWT middleware                                                   | boolean  | false                  |
| fiber.middleware.jwt.ignore               | Ignore by path                                                          | []string | []                     |
| fiber.middleware.jwt.skipVerify           | Skip verify JWT token                                                   | boolean  | false                  |
| fiber.middleware.jwt.signerEntry          | SignerEntry name                                                        | string   | ""                     |
| fiber.middleware.jwt.symmetric.algorithm  | Symmetric algorithm, options: HS256, HS384, HS512                       | string   | ""                     |
| fiber.middleware.jwt.symmetric.token      | Symmetric token                                                         | string   | ""                     |
| fiber.middleware.jwt.symmetric.tokenPath  | Symmetric token path                                                    | string   | ""                     |
| fiber.middleware.jwt.asymmetric.algorithm | Asymmetric algorithm, options: RS256, RS384, RS512, ES256, ES384, ES512 | string   | ""                     |
| fiber.middleware.jwt.tokenLookup          | JWT Token format，see example bellow                                     | string   | "header:Authorization" |
| fiber.middleware.jwt.authScheme           | Auth Scheme                                                             | string   | Bearer                 |

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
fiber:
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
#        ignore: [ "" ]
#        skipVerify: false
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
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.

package main

import (
  "context"
  "fmt"
  "github.com/gofiber/fiber/v2"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-fiber/boot"
  "net/http"
)

// @title Swagger Example API
// @version 1.0
// @description This is a sample rk-demo server.
// @termsOfService http://swagger.io/terms/

// @securityDefinitions.basic BasicAuth

// @contact.name API Support
// @contact.url http://www.swagger.io/support
// @contact.email support@swagger.io

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html
func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Register handler
  entry := rkfiber.GetFiberEntry("greeter")
  entry.App.Get("/v1/greeter", Greeter)
  // This is required!!!
  entry.RefreshFiberRoutes()

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter
// @Id 1
// @Tags Hello
// @version 1.0
// @Param name query string true "name"
// @produce application/json
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(ctx *fiber.Ctx) error {
  ctx.Response().SetStatusCode(http.StatusOK)
  return ctx.JSON(&GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.Validate
- Valid JWT Token

```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.EpM5XBzTJZ4J8AfoJEcJrjth8pfH28LWdjLo90sYb9g"
{"Message":"Hello rk-dev!"}
```

- Invalid JWT Token
```bash
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
![](../../../../img/user-guide/cheers.png)