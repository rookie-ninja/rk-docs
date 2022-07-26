---
User can use embedFS to provide static files to rk-boot.

## Overview
User can provide Swagger UI config file, API Docs config file and boot.yaml file to rk-boot with embedFS.

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-mux
```

### 2.Create boot.yaml
```yaml
mux:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.Create main.go
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

### 4.Start main.go
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
mux:
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
  _ "embed"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-mux/boot"
  "github.com/rookie-ninja/rk-mux/middleware"
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
