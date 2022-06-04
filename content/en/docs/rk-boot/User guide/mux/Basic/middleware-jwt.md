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
go get github.com/rookie-ninja/rk-mux
```

## Options
| options                                 | description                                                             | type     | default                |
|-----------------------------------------|-------------------------------------------------------------------------|----------|------------------------|
| mux.middleware.jwt.enabled              | Enable JWT middleware                                                   | boolean  | false                  |
| mux.middleware.jwt.ignore               | Ignore by path                                                          | []string | []                     |
| mux.middleware.jwt.skipVerify           | Skip verify JWT token                                                   | boolean  | false                  |
| mux.middleware.jwt.signerEntry          | SignerEntry name                                                        | string   | ""                     |
| mux.middleware.jwt.symmetric.algorithm  | Symmetric algorithm, options: HS256, HS384, HS512                       | string   | ""                     |
| mux.middleware.jwt.symmetric.token      | Symmetric token                                                         | string   | ""                     |
| mux.middleware.jwt.symmetric.tokenPath  | Symmetric token path                                                    | string   | ""                     |
| mux.middleware.jwt.asymmetric.algorithm | Asymmetric algorithm, options: RS256, RS384, RS512, ES256, ES384, ES512 | string   | ""                     |
| mux.middleware.jwt.tokenLookup          | JWT Token format，see example bellow                                     | string   | "header:Authorization" |
| mux.middleware.jwt.authScheme           | Auth Scheme                                                             | string   | Bearer                 |

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
mux:
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