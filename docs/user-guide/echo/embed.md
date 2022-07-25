User can use embedFS to provide static files to rk-boot.

## Overview
User can provide Swagger UI config file, API Docs config file and boot.yaml file to rk-boot with embedFS.

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-echo
```

### 2.Create boot.yaml
```yaml
echo:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.Create main.go
```go
package main

import (
	"context"
	"embed"
	"fmt"
    "github.com/labstack/echo/v4"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-entry/v2/entry"
	"github.com/rookie-ninja/rk-echo/boot"
	"net/http"
)

//go:embed boot.yaml
var bootRaw []byte

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot(rkboot.WithBootConfigRaw(bootRaw))

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

### 4.Start main.go
```bash
$ go run main.go
2022-04-17T00:47:34.597+0800    INFO    boot/echo_entry.go:656   Bootstrap GinEntry      {"eventId": "fe0b0e26-1271-4294-81c8-4f675e308335", "entryName": "greeter", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T00:47:34.597345+08:00
startTime=2022-04-17T00:47:34.597248+08:00
elapsedNano=96828
timezone=CST
ids={"eventId":"fe0b0e26-1271-4294-81c8-4f675e308335"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"EchoEntry"}
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

### 5.Validate
```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}%
```

## Swagger UI & Docs UI
### 1.Swagger JSON in local FS
```bash
├── docs
│   ├── swagger.json
│   └── swagger.yaml
```

### 2.Create boot.yaml
```yaml
echo:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
    docs:
      enabled: true
```

### 3.Create main.go
```go
package main

import (
	"context"
	"embed"
	"fmt"
    "github.com/labstack/echo/v4"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-entry/v2/entry"
	"github.com/rookie-ninja/rk-echo/boot"
	"net/http"
)

//go:embed boot.yaml
var bootRaw []byte

//go:embed docs
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
