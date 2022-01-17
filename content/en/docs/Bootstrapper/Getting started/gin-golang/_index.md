---
title: "Gin-golang"
linkTitle: "Gin-golang"
weight: 1
description: >
  Create a [Gin](https://github.com/gin-gonic/gin) server with RK bootstrapper.
---

## Overview
This is an example demonstrate how to configure boot.yaml file to start a Gin server with bellow functionality.

> Full demo: https://github.com/rookie-ninja/rk-demo/gin/getting-started

- [Gin server](https://github.com/gin-gonic/gin)
- [Swagger UI](https://swagger.io/tools/swagger-ui/)
- CommonService API
- RK TV Web UI
- User defined API

## Create server
### 1.Install dependency
```shell script
$ go get github.com/rookie-ninja/rk-boot/gin
```

### 2.Create boot.yaml
```yaml
---
gin:
  - name: greeter       # Name of gin entry
    port: 8080          # Port of gin entry
    enabled: true       # Enable gin entry
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
	_ "github.com/rookie-ninja/rk-boot/gin"
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
2022-01-17T14:34:15.758+0800    INFO    boot/gin_entry.go:596   Bootstrap ginEntry      {"eventId": "ac0502b5-e52f-4bf9-a95b-7efb1326d6e1", "entryName": "greeter"}
------------------------------------------------------------------------
endTime=2022-01-17T14:34:15.760234+08:00
startTime=2022-01-17T14:34:15.75864+08:00
elapsedNano=1593960
timezone=CST
ids={"eventId":"ac0502b5-e52f-4bf9-a95b-7efb1326d6e1"}
app={"appName":"rk","appVersion":"","entryName":"greeter","entryType":"Gin"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
payloads={"commonServiceEnabled":true,"commonServicePathPrefix":"/rk/v1/","ginPort":8080,"swEnabled":true,"swPath":"/sw/","tvEnabled":true,"tvPath":"/rk/v1/tv/"}
error={}
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
> **Swagger:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/gin-golang/gin-sw.png)

> **TV:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/gin-golang/gin-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## Add an API
We will define a Greeter API with the path of "/v1/greeter". 

### 1.Register API
> We need **gin.Engine** in order to register **HandlerFunc**. 
>
> In GinEntry, **gin.Engine** will be initialized by bootstrapper with the name of **Router**.

```go
package main

import (
	"context"
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-boot/gin"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler before bootstrap!
	ginEntry := rkbootgin.GetGinEntry("greeter")
	ginEntry.Router.GET("/v1/greeter", Greeter)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

// Gin handler function
func Greeter(ctx *gin.Context) {
	ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
	})
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
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// @title RK Swagger for Gin
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
func Greeter(ctx *gin.Context) {...}
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
In order to make rk-boot finds out available swagger config files, we need to add **gin.sw.jsonPath** in boot.yaml file.
```yaml
---
gin:
  - name: greeter
    ...
    sw:
      enabled: true
      jsonPath: "docs"  # Boot will look for swagger config files from this folder
    ...
```

### 5.Validate
> **Swagger:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/gin-golang/gin-sw-api.png)

> **TV:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/gin-golang/gin-tv-api.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)