启动 gzip 中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## gzip 选项
| 名字                          | 描述                                                                                    | 类型       | 默认值                |
|-----------------------------|---------------------------------------------------------------------------------------|----------|--------------------|
| echo.middleware.gzip.enabled | 启动 gzip 压缩/解压缩中间件                                                                     | boolean  | false              |
| echo.middleware.gzip.ignore  | 局部选项，忽略 API 路径                                                                        | []string | []                 |
| echo.middleware.gzip.level   | 压缩比例, 选项为 noCompression, bestSpeed, bestCompression, defaultCompression, huffmanOnly. | string   | defaultCompression |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
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
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-echo/boot"
  "net/http"
  "io"
  "strings"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  echoEntry := rkecho.GetEchoEntry("greeter")
  echoEntry.Echo.POST("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx echo.Context) error {
  buf := new(strings.Builder)
  io.Copy(buf, ctx.Request.Body)
  
  return ctx.JSON(http.StatusOK, &GreeterResponse{
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
![](../../../img/user-guide/cheers.png)