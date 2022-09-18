rk-boot 中，可以使用 embed.FS 给 rk-boot 提供 boot.yaml 等静态文件。

## 概述
目前，用户可以把 Swagger UI 参数文件, API Docs 参数文件和 boot.yaml 文件通过 embed.FS 传递给 rk-boot.

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-mux
```

### 2.创建 boot.yaml
```yaml
mux:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.创建 main.go
```go
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

//go:embed boot.yaml
var bootRaw []byte

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot(rkboot.WithBootConfigRaw(bootRaw))

  // Get MuxEntry
  muxEntry := rkmux.GetMuxEntry("greeter")
  // Use *mux.Router adding handler.
  muxEntry.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(writer http.ResponseWriter, req *http.Request) {
  rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 4.启动 main.go
```bash
$ go run main.go
2022-05-10T12:08:39.562+0800    INFO    boot/mux_entry.go:687   Bootstrap muxEntry      {"eventId": "471962da-2fdb-4154-8bc6-a31a8a4637cd", "entryName": "greeter", "entryType": "MuxEntry"}
------------------------------------------------------------------------
endTime=2022-05-10T12:08:39.562674+08:00
startTime=2022-05-10T12:08:39.562581+08:00
elapsedNano=93582
timezone=CST
ids={"eventId":"471962da-2fdb-4154-8bc6-a31a8a4637cd"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"MuxEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin"}
payloads={"muxPort":8080}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

### 5.验证
```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}%
```

## Swagger UI 与 Docs UI
我们同时还可以让 rk-boot 使用 embedFS 读取 swagger 相关的参数文件。

### 1.已存在的 Swagger JSON 文件
```bash
├── docs
│   ├── swagger.json
│   └── swagger.yaml
```

### 2.创建 boot.yaml
```yaml
mux:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
    docs:
      enabled: true
```

### 3.创建 main.go

```go
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

//go:embed boot.yaml
var bootRaw []byte

//go:embed ../../..
var docsFS embed.FS

//go:embed docs
var swFS embed.FS

func init() {
	rkentry.GlobalAppCtx.AddEmbedFS(rkentry.SWEntryType, "greeter", &docsFS)
	rkentry.GlobalAppCtx.AddEmbedFS(rkentry.DocsEntryType, "greeter", &swFS)
}

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot(rkboot.WithBootConfigRaw(bootRaw))

	// Get MuxEntry
	muxEntry := rkmux.GetMuxEntry("greeter")
	// Use *mux.Router adding handler.
	muxEntry.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

	// Bootstrap
	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}

func Greeter(writer http.ResponseWriter, req *http.Request) {
	rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
	})
}

type GreeterResponse struct {
	Message string
}
```
