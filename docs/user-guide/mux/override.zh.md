当使用 rk-boot 的时候，如何通过 flag 覆盖 boot.yaml 里的参数？

## 概述
rk-boot 支持通过命令行和环境变量参数来覆盖 boot.yaml 里的值。

- 覆盖 boot.yaml 里的参数 (通过 \-\-rkset)

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-mux
```

### 2.创建 boot.yaml
```yaml
---
mux:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.创建 main.go

```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
  "context"
  _ "embed"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-mux/boot"
  "github.com/rookie-ninja/rk-mux/middleware"
  "net/http"
)

// @title RK Swagger for Mux
// @version 1.0
// @description This is a greeter service with rk-boot.
func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Get MuxEntry
  muxEntry := rkmux.GetMuxEntry("greeter")
  // Use *mux.Router adding handler.
  muxEntry.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(writer http.ResponseWriter, req *http.Request) {
  rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 5.通过 flag 覆盖 Port
如果是 List，请使用【mux[0].port】格式访问。

```bash
$ go run main.go --rkset mux[0].port=8081
2022-05-10T12:24:50.605+0800    INFO    entry/util.go:332       Found flag to override, applying...     {"flags": ["mux[0].port=8081"]}
2022-05-10T12:24:50.605+0800    INFO    boot/mux_entry.go:687   Bootstrap muxEntry      {"eventId": "d6931ca3-e16a-45ba-ad37-30a90f8bbd17", "entryName": "greeter", "entryType": "MuxEntry"}
------------------------------------------------------------------------
endTime=2022-05-10T12:24:50.605613+08:00
startTime=2022-05-10T12:24:50.605571+08:00
elapsedNano=42090
timezone=CST
ids={"eventId":"d6931ca3-e16a-45ba-ad37-30a90f8bbd17"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"MuxEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"muxPort":8081}
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

例如：RK_MUX_0_PORT

```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
  "context"
  _ "embed"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-mux/boot"
  "github.com/rookie-ninja/rk-mux/middleware"
  "net/http"
  "os"
)

// @title RK Swagger for Mux
// @version 1.0
// @description This is a greeter service with rk-boot.
func main() {
  os.Setenv("RK_MUX_0_PORT", "8081")

  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Get MuxEntry
  muxEntry := rkmux.GetMuxEntry("greeter")
  // Use *mux.Router adding handler.
  muxEntry.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(writer http.ResponseWriter, req *http.Request) {
  rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

```bash
$ go run main.go
2022-05-10T12:25:27.143+0800    INFO    entry/util.go:299       Found ENV to override, applying...      {"env": ["RK_MUX_0_PORT=8081 => mux[0].port=8081"]}
2022-05-10T12:25:27.144+0800    INFO    boot/mux_entry.go:687   Bootstrap muxEntry      {"eventId": "7813fc2c-bb03-42f5-8aa7-e8aea479fe7c", "entryName": "greeter", "entryType": "MuxEntry"}
------------------------------------------------------------------------
endTime=2022-05-10T12:25:27.144887+08:00
startTime=2022-05-10T12:25:27.144788+08:00
elapsedNano=98900
timezone=CST
ids={"eventId":"7813fc2c-bb03-42f5-8aa7-e8aea479fe7c"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"MuxEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"muxPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```