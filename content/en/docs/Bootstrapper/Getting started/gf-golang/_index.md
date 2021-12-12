---
title: "GoFrame-golang"
linkTitle: "GoFrame-golang"
weight: 3
description: >
  Create a [GoFrame](https://github.com/gogf/gf) server with RK bootstrapper.
---

## Overview
This is an example demonstrate how to configure boot.yaml file to start a GoFrame server with bellow functionality.

> Full demo: https://github.com/rookie-ninja/rk-demo/gf/getting-started

- [GoFrame server](https://github.com/gogf/gf)
- [Swagger UI](https://swagger.io/tools/swagger-ui/)
- CommonService API
- RK TV Web UI
- User defined API

## Create server
### 1.Install dependency
```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-gf
```

### 2.Create boot.yaml
```yaml
---
gf:
  - name: greeter       # Name of gf entry
    port: 8080          # Port of gf entry
    enabled: true       # Enable gf entry
    sw:
      enabled: true     # Enable swagger UI
    commonService:
      enabled: true     # Enable common service
    tv:
      enabled:  true    # Enable RK TV
```

### 3.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-gf/boot"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 4.Start server
```go
$ go run main.go
...
2021-12-06T01:33:09.663+0800    INFO    boot/gf_entry.go:848    Bootstrapping GfEntry.  {"eventId": "ea7026c0-51e4-435b-bca7-06584949405c", "entryName": "greeter", "entryType": "GfEntry", "port": 8080}
------------------------------------------------------------------------
endTime=2021-12-06T01:33:09.663146+08:00
startTime=2021-12-06T01:33:09.660935+08:00
elapsedNano=2211886
timezone=CST
ids={"eventId":"ea7026c0-51e4-435b-bca7-06584949405c"}
app={"appName":"rk-demo","appVersion":"master-878d8ab","entryName":"greeter","entryType":"GfEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"entryName":"greeter","entryType":"GfEntry","port":8080}
error={}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=bootstrap
resCode=OK
eventStatus=Ended
EOE
```

### 5.Validate
> **Swagger:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/gf-golang/gf-sw.png)

> **TV:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/gf-golang/gf-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## Add an API
We will define a Greeter API with the path of "/v1/greeter". 

### 1.Register API
> We need **gf.Server** in order to register **HandlerFunc**. 
>
> In GfEntry, **ghttp.Server** will be initialized by bootstrapper with the name of **Server**.

```go
package main

import (
	"context"
	"fmt"
	"github.com/labstack/echo/v4"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-gf/boot"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler before bootstrap!
	gfEntry := boot.GetEntry("greeter").(*rkgf.GfEntry)
	gfEntry.Server.BindHandler("/v1/greeter", Greeter)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

// GoFrame handler
func Greeter(ctx *ghttp.Request) {
	ctx.Response.WriteHeader(http.StatusOK)
	err := ctx.Response.WriteJson(&GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name")),
	})

	if err != nil {
		panic(err)
	}
}

// Response.
type GreeterResponse struct {
	Message string
}
```

### 2.Validate
```shell script
$ curl "http://localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## Support Swagger UI
We will use [swag](https://github.com/swaggo/swag) to generate swagger UI config files.

### 1.Install swag
> **Option 1:** 
>
> From swag the official repository:
```shell script
$ go get -u github.com/swaggo/swag/cmd/swag
```

> **Option 2:** 
>
> With [RK CMD](https://github.com/rookie-ninja/rk)
```shell script
# Install RK CMD
$ go get -u github.com/rookie-ninja/rk/cmd/rk

# Install swag with rk
$ rk install swag
```

### 2.Add comments
```go
package main

import (
	"context"
	"fmt"
	"github.com/gogf/gf/v2/net/ghttp"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// @title RK Swagger for GoFrame
// @version 1.0
// @description This is a greeter service with rk-boot.

// Application entrance.
func main() {...}

// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(ctx *ghttp.Request) {...}
```

### 3.Generate swagger config
```shell script
$ swag init

# swagger.json, swagger.yaml, docs.go files will be generated under ./docs folder.
$ tree
.
├── boot.yaml
├── docs
│   ├── docs.go
│   ├── swagger.json
│   └── swagger.yaml
├── go.mod
├── go.sum
└── main.go

1 directory, 7 files
```

### 4.Add path to boot.yaml
In order to make rk-boot finds out available swagger config files, we need to add **gf.sw.jsonPath** in boot.yaml file.
```yaml
---
gf:
  - name: greeter
    ...
    sw:
      enabled: true
      jsonPath: "docs"  # Boot will look for swagger config files from this folder
    ...
```

### 5.Validate
> **Swagger:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/gf-golang/gf-sw-api.png)

> **TV:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/gf-golang/gf-tv-api.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)