Start multiple entries in one process.

## Overview
User can start multiple Entry in one single process.

Furthermore, use can even start Gin and gRPC in one single process.

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
```

### 2.Create boot.yaml
```yaml
gin:
  - name: alice
    port: 8080
    enabled: true
  - name: bob
    port: 8081
    enabled: true
```

### 3.Create main.go
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
  alice := rkgin.GetGinEntry("alice")
  alice.Router.GET("/v1/greeter", Greeter)

  bob := rkgin.GetGinEntry("bob")
  bob.Router.GET("/v1/greeter", Greeter)

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

### 4.Validate
```bash
$ curl  "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}

$ curl  "localhost:8081/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)