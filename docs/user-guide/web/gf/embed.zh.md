rk-boot 中，可以使用 embed.FS 给 rk-boot 提供 boot.yaml 等静态文件。

## 概述
目前，用户可以把 Swagger UI 参数文件, API Docs 参数文件和 boot.yaml 文件通过 embed.FS 传递给 rk-boot.

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gf
```

### 2.创建 boot.yaml
```yaml
gf:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.创建 main.go
```go
package main

import (
	"context"
	"embed"
	"fmt"
    "github.com/gogf/gf/v2/net/ghttp"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-entry/v2/entry"
	"github.com/rookie-ninja/rk-gf/boot"
	"net/http"
)

//go:embed boot.yaml
var bootRaw []byte

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot(rkboot.WithBootConfigRaw(bootRaw))

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

### 4.启动 main.go
```bash
$ go run main.go
2022-04-17T00:47:34.597+0800    INFO    boot/gf_entry.go:656   Bootstrap GfEntry      {"eventId": "fe0b0e26-1271-4294-81c8-4f675e308335", "entryName": "greeter", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T00:47:34.597345+08:00
startTime=2022-04-17T00:47:34.597248+08:00
elapsedNano=96828
timezone=CST
ids={"eventId":"fe0b0e26-1271-4294-81c8-4f675e308335"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GfEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"ginPort":8080}
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
gf:
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
	"embed"
	"fmt"
	"github.com/gogf/gf/v2/net/ghttp"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-entry/v2/entry"
	"github.com/rookie-ninja/rk-gf/boot"
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
