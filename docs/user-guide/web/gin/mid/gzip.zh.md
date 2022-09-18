启动 gzip 中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

## gzip 选项
| 名字                          | 描述                                                                                    | 类型       | 默认值                |
|-----------------------------|---------------------------------------------------------------------------------------|----------|--------------------|
| gin.middleware.gzip.enabled | 启动 gzip 压缩/解压缩中间件                                                                     | boolean  | false              |
| gin.middleware.gzip.ignore  | 局部选项，忽略 API 路径                                                                        | []string | []                 |
| gin.middleware.gzip.level   | 压缩比例, 选项为 noCompression, bestSpeed, bestCompression, defaultCompression, huffmanOnly. | string   | defaultCompression |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      gzip:
        enabled: true
        level: bestSpeed
#        ignore: [""]
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
  "io"
  "net/http"
  "strings"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgin.GetGinEntry("greeter")
  entry.Router.POST("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *gin.Context) {
  buf := new(strings.Builder)
  io.Copy(buf, ctx.Request.Body)

  ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Received %s!", buf.String()),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.验证
> 发送请求

```bash
$ echo 'this is message' | gzip | curl --compressed --data-binary @- -H "Content-Encoding: gzip" -H "Accept-Encoding: gzip" localhost:8080/v1/greeter
{"Message":"Received this is message\n!"}%
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)