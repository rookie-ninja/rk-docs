Enable ratelimit middleware.

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## Option
| options                                   | description                   | type     | default     |
|-------------------------------------------|-------------------------------|----------|-------------|
| fiber.middleware.rateLimit.enabled         | Enable ratelimit middleware   | boolean  | false       |
| fiber.middleware.rateLimit.ignore          | Ignore by path                | []string | []          |
| fiber.middleware.rateLimit.algorithm       | leakyBucket is supported only | string   | leakyBucket |
| fiber.middleware.rateLimit.reqPerSec       | global limit                  | int      | 1000000     |
| fiber.middleware.rateLimit.paths.path      | API path                      | string   | ""          |
| fiber.middleware.rateLimit.paths.reqPerSec | limit value of path           | int      | 1000000     |

## Quick start
### 1.Create boot.yaml
```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      rateLimit:
        enabled: true
        paths:
          - path: "/v1/greeter"
            reqPerSec: 0
#        algorithm: "leakyBucket"
#        reqPerSec: 100
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
> Send request

```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{
    "error":{
        "code":429,
        "status":"Too Many Requests",
        "message":"",
        "details":[
            "slow down your request"
        ]
    }
}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)