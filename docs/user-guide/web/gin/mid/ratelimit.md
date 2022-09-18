Enable ratelimit middleware.

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

## Option
| options                     | description                        | type     | default |
|------------------------------------------|-------------------------------|----------|-------------|
| gin.middleware.rateLimit.enabled         | Enable ratelimit middleware   | boolean  | false       |
| gin.middleware.rateLimit.ignore  | Ignore by path                | []string | []    |
| gin.middleware.rateLimit.algorithm       | leakyBucket is supported only | string   | leakyBucket |
| gin.middleware.rateLimit.reqPerSec       | global limit                  | int      | 1000000     |
| gin.middleware.rateLimit.paths.path      | API path                      | string   | ""          |
| gin.middleware.rateLimit.paths.reqPerSec | limit value of path           | int      | 1000000     |

## Quick start
### 1.Start boot.yaml
```yaml
---
gin:
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
package main

import (
  "context"
  "fmt"
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gin/v2/boot"
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
  ctx.JSON(http.StatusOK, &GreeterResponse{
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