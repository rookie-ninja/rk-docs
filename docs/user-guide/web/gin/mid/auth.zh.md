启动权限中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

## 权限选项
| 名字                          | 描述                           | 类型       | 默认值   |
|-----------------------------|------------------------------|----------|-------|
| gin.middleware.auth.enabled | 启动权限中间件                      | boolean  | false |
| gin.middleware.auth.ignore  | 局部选项，忽略 API 路径               | []string | []    |
| gin.middleware.auth.basic   | Basic Auth 信息，格式：<user:pass> | []string | []    |
| gin.middleware.auth.apiKey  | X-API-Key                    | []string | []    |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      auth:
        enabled: true
        basic: ["user:pass"]
#        ignore: [""]
#        apiKey:
#          - "keys"
```

### 2.创建 main.go
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

### 3.验证
```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
# This is RK style error code if unauthorized
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"Missing authorization, provide one of bellow auth header:[Basic Auth]",
        "details":[]
    }
}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 4.X-API-Key 类型授权
```yaml
---
gin:
  - name: greeter
    ...
    middleware:
      auth:
        enabled: true
        apiKey: ["token"]
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5.忽略请求路径
```yaml
---
gin:
  - name: greeter
    ...
    middleware:
      auth:
        enabled: true
        basic: ["user:pass"]
        ignorePrefix: ["/v1/greeter"]
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)