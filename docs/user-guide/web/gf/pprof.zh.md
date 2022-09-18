启动 pprof UI。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## 选项
| 名字                | 描述              | 类型      | 默认值   |
|-------------------|-----------------|---------|-------|
| gf.pprof.enabled | 启动 pprof web UI | boolean | false |
| gf.pprof.path    | pprof web 界面路径  | string  | pprof |

## 快速开始
### 1.创建 boot.yaml

```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    pprof:
      enabled: true
#      path: ""
```

### 2.创建 main.go
```go
package main

import (
	"context"
	"fmt"
    "github.com/gogf/gf/v2/net/ghttp"
    "github.com/rookie-ninja/rk-boot/v2"
    "github.com/rookie-ninja/rk-gf/boot"
    "net/http"
)

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
    entry := rkgf.GetGfEntry("greeter")
    entry.Server.BindHandler("/v1/greeter", Greeter)

	// Bootstrap
	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *ghttp.Request) {
  ctx.Response.WriteHeader(http.StatusOK)
  ctx.Response.WriteJson(&GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name").String()),
  })
}

type GreeterResponse struct {
	Message string
}
```

### 3.验证
> **PPROF:** [http://localhost:8080/pprof](http://localhost:8080/pprof)

![](../../../img/user-guide/gin/basic/gin-pprof.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
