启动 Prometheus 中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## 选项
| 名字                          | 描述                | 类型       | 默认值   |
|-----------------------------|-------------------|----------|-------|
| echo.middleware.prom.enabled | 启动 Prometheus 中间件 | boolean  | false |
| echo.middleware.prom.ignore  | 局部选项，忽略 API 路径    | []string | []    |

## 概念
Prometheus 中间件会默认记录如下监控。

| 监控项         | 数据类型    | 详情             |
|-------------|---------|----------------|
| elapsedNano | Summary | RPC 耗时         |
| resCode     | Counter | 基于 RPC 返回码的计数器 |

上述三项监控，都有如下的标签。

| 标签         | 详情                                                                                                |
|------------|---------------------------------------------------------------------------------------------------|
| entryName  | Entry 名称                                                                                          |
| entryType  | Entry 类型                                                                                          |
| domain     | 环境变量: DOMAIN, eg: prod                                                                            |
| instance   | 本地 Hostname                                                                                       |
| restMethod | eg: GET                                                                                           |
| restPath   | eg: /rk/v1/alive                                                                                  |
| resCode    | 返回码, eg: 200                                                                                      |

例子

```bash
rk_prom_elapsedNano{domain="*",entryName="greeter",entryType="EchoEntry",instance="lark.local",resCode="200",restMethod="GET",restPath="/v1/greeter",quantile="0.5"} 88645
...
rk_prom_resCode{domain="*",entryName="greeter",entryType="EchoEntry",instance="lark.local",resCode="200",restMethod="GET",restPath="/v1/greeter"} 1
```

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    prom:
      enabled : true
    middleware:
      prom:
        enabled: true
#        ignore: [""]
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
> 发送请求

```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

> Prometheus 客户端:
>
> http://localhost:8080/metrics

![prom-inter](../../../img/user-guide/gin/basic/gin-prom-inter.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
