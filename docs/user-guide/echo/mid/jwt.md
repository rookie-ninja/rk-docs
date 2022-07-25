Enable JWT middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## Options
| options                                  | description                                                             | type     | default                |
|------------------------------------------|-------------------------------------------------------------------------|----------|------------------------|
| echo.middleware.jwt.enabled              | Enable JWT middleware                                                   | boolean  | false                  |
| echo.middleware.jwt.ignore               | Ignore by path                                                          | []string | []                     |
| echo.middleware.jwt.skipVerify           | Skip verify JWT token                                                   | boolean  | false                  |
| echo.middleware.jwt.signerEntry          | SignerEntry name                                                        | string   | ""                     |
| echo.middleware.jwt.symmetric.algorithm  | Symmetric algorithm, options: HS256, HS384, HS512                       | string   | ""                     |
| echo.middleware.jwt.symmetric.token      | Symmetric token                                                         | string   | ""                     |
| echo.middleware.jwt.symmetric.tokenPath  | Symmetric token path                                                    | string   | ""                     |
| echo.middleware.jwt.asymmetric.algorithm | Asymmetric algorithm, options: RS256, RS384, RS512, ES256, ES384, ES512 | string   | ""                     |
| echo.middleware.jwt.tokenLookup          | JWT Token format，see example bellow                                     | string   | "header:Authorization" |
| echo.middleware.jwt.authScheme           | Auth Scheme                                                             | string   | Bearer                 |

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
echo:
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
package main

import (
  "context"
  "fmt"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-echo/boot"
  "github.com/rookie-ninja/rk-echo/middleware/context"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  echoEntry := rkecho.GetEchoEntry("greeter")
  echoEntry.Echo.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx echo.Context) error {
  // Get JWT token
  rkechoctx.GetJwtToken(ctx)
  
  return ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.QueryParam("name")),
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
![](../../../img/user-guide/cheers.png)