---
title: "PPROF"
linkTitle: "PPROF"
weight: 4
description: >
  启动 pprof UI。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## 选项
| 名字                | 描述              | 类型      | 默认值   |
|-------------------|-----------------|---------|-------|
| echo.pprof.enabled | 启动 pprof web UI | boolean | false |
| echo.pprof.path    | pprof web 界面路径  | string  | pprof |

## 快速开始
### 1.创建 boot.yaml

```yaml
---
echo:
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
    "github.com/labstack/echo/v4"
    "github.com/rookie-ninja/rk-boot/v2"
    "github.com/rookie-ninja/rk-echo/boot"
    "net/http"
)

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
    echoEntry := rkecho.GetEchoEntry("greeter")
    echoEntry.Echo.GET("/v1/greeter", Greeter)

	// Bootstrap
	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx echo.Context) error {
  return ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.QueryParam("name")),
  })
}

type GreeterResponse struct {
	Message string
}
```

### 3.验证
> **PPROF:** [http://localhost:8080/pprof](http://localhost:8080/pprof)

![](/rk-boot/user-guide/gin/basic/gin-pprof.png)

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
