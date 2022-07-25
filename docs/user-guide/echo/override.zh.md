当使用 rk-boot 的时候，如何通过 flag 覆盖 boot.yaml 里的参数？

## 概述
rk-boot 支持通过命令行和环境变量参数来覆盖 boot.yaml 里的值。

- 覆盖 boot.yaml 里的参数 (通过 \-\-rkset)

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-echo
```

### 2.创建 boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.创建 main.go

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

### 5.通过 flag 覆盖 Port
如果是 List，请使用【echo[0].port】格式访问。

```bash
$ go run main.go --rkset echo[0].port=8081
2022-04-17T00:13:52.166+0800    INFO    entry/util.go:332       Found flag to override, applying...     {"flags": ["gin[0].port=8081"]}
2022-04-17T00:13:52.167+0800    INFO    boot/echo_entry.go:656   Bootstrap EchoEntry      {"eventId": "19a961f4-b866-4047-9741-aa0a2d00055b", "entryName": "greeter", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T00:13:52.167194+08:00
startTime=2022-04-17T00:13:52.167146+08:00
elapsedNano=47911
timezone=CST
ids={"eventId":"19a961f4-b866-4047-9741-aa0a2d00055b"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"EchoEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"ginPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

> 发送请求到：8081
```bash
$ curl "localhost:8081/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)

### 6.通过环境变量覆盖 Port
覆盖 boot.yaml 里参数的时候，需要带上 【RK_】 作为前缀，此外，如果是 List 使用 【_X_】 访问。

例如：RK_GIN_0_PORT

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
  os.Setenv("RK_ECHO_0_PORT", "8081")
  
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

```bash
$ go run main.go
2022-04-17T00:20:38.452+0800    INFO    entry/util.go:299       Found ENV to override, applying...      {"env": ["RK_GIN_0_PORT=8081 => gin[0].port=8081"]}
2022-04-17T00:20:38.453+0800    INFO    boot/echo_entry.go:656   Bootstrap EchoEntry      {"eventId": "06f271e8-7eb8-4c1a-abef-f710afe18021", "entryName": "greeter", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T00:20:38.453386+08:00
startTime=2022-04-17T00:20:38.453345+08:00
elapsedNano=41474
timezone=CST
ids={"eventId":"06f271e8-7eb8-4c1a-abef-f710afe18021"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"EchoEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"ginPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```