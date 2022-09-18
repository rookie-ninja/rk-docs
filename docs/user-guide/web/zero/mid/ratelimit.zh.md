启动限流中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-zero
```

## 选项
| 名字                                       | 描述                   | 类型      | 默认值         |
|------------------------------------------|----------------------|---------|-------------|
| zero.middleware.rateLimit.enabled         | 启动限流中间件              | boolean | false       |
| zero.middleware.rateLimit.ignore            | 局部选项，忽略 API 路径       | []string | []      |
| zero.middleware.rateLimit.algorithm       | 限流算法， 支持 leakyBucket | string  | leakyBucket |
| zero.middleware.rateLimit.reqPerSec       | 全局限流值                | int     | 1000000     |
| zero.middleware.rateLimit.paths.path      | 访问路径                 | string  | ""          |
| zero.middleware.rateLimit.paths.reqPerSec | 基于访问路径的限流值           | int     | 1000000     |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
zero:
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

### 2.创建 main.go
```go
package main

import (
  "context"
  "encoding/json"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-zero/boot"
  "github.com/zeromicro/go-zero/rest"
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

  // Register handler
  zeroEntry := rkzero.GetZeroEntry("greeter")
  zeroEntry.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: Greeter,
  })

  // Bootstrap
  boot.Bootstrap(context.TODO())

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
func Greeter(writer http.ResponseWriter, request *http.Request) {
  writer.WriteHeader(http.StatusOK)
  resp := &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", request.URL.Query().Get("name")),
  }
  bytes, _ := json.Marshal(resp)
  writer.Write(bytes)
}

type GreeterResponse struct {
  Message string
}
```

### 3.验证
> 发送请求

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