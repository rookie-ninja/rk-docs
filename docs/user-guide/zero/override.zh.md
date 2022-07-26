当使用 rk-boot 的时候，如何通过 flag 覆盖 boot.yaml 里的参数？

## 概述
rk-boot 支持通过命令行和环境变量参数来覆盖 boot.yaml 里的值。

- 覆盖 boot.yaml 里的参数 (通过 \-\-rkset)

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-zero
```

### 2.创建 boot.yaml
```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.创建 main.go

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

### 5.通过 flag 覆盖 Port
如果是 List，请使用【zero[0].port】格式访问。

```bash
$ go run main.go --rkset zero[0].port=8081
2022-05-09T13:05:52.632+0800    INFO    entry/util.go:332       Found flag to override, applying...     {"flags": ["zero[0].port=8081"]}
2022-05-09T13:05:52.632+0800    INFO    boot/zero_entry.go:761  Bootstrap zeroEntry     {"eventId": "dd06a3d9-f437-49d4-ac4f-05ebceb257f0", "entryName": "greeter", "entryType": "ZeroEntry"}
------------------------------------------------------------------------
endTime=2022-05-09T13:05:52.633023+08:00
startTime=2022-05-09T13:05:52.632981+08:00
elapsedNano=41924
timezone=CST
ids={"eventId":"dd06a3d9-f437-49d4-ac4f-05ebceb257f0"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"zeroPort":8081}
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

例如：RK_ZERO_0_PORT

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
  "os"
)

func main() {
  os.Setenv("RK_ZERO_0_PORT", "8081")

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

```bash
$ go run main.go
2022-05-09T13:06:42.437+0800    INFO    entry/util.go:299       Found ENV to override, applying...      {"env": ["RK_ZERO_0_PORT=8081 => zero[0].port=8081"]}
2022-05-09T13:06:42.438+0800    INFO    boot/zero_entry.go:761  Bootstrap zeroEntry     {"eventId": "2d5bd046-a0bb-4887-a252-449691d7fde8", "entryName": "greeter", "entryType": "ZeroEntry"}
------------------------------------------------------------------------
endTime=2022-05-09T13:06:42.438376+08:00
startTime=2022-05-09T13:06:42.438324+08:00
elapsedNano=52141
timezone=CST
ids={"eventId":"2d5bd046-a0bb-4887-a252-449691d7fde8"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"zeroPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```