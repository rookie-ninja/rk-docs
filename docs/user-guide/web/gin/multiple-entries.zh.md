如何在 rk-boot 中，启动多个服务？

## 概述
通过 rk-boot，用户可以在一个进程里面，启动多个 Entry。

用户还可以同时启动 Gin 服务和 gRPC 等服务。

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
```

### 2.创建 boot.yaml
```yaml
gin:
  - name: alice
    port: 8080
    enabled: true
  - name: bob
    port: 8081
    enabled: true
```

### 3.创建 main.go
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

### 4.验证
```bash
$ curl  "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}

$ curl  "localhost:8081/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)