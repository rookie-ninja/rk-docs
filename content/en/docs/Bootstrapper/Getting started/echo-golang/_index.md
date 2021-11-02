---
title: "Echo-golang"
linkTitle: "Echo-golang"
weight: 3
description: >
  Create a [Echo](https://github.com/labstack/echo) server with RK bootstrapper.
---

## Overview
This is an example demonstrate how to configure boot.yaml file to start a Echo server with bellow functionality.

> Full demo: https://github.com/rookie-ninja/rk-demo/echo/getting-started

- [Echo server](https://github.com/labstack/echo)
- [Swagger UI](https://swagger.io/tools/swagger-ui/)
- CommonService API
- RK TV Web UI
- User defined API

## Create server
### 1.Install dependency
```shell script
$ go get github.com/rookie-ninja/rk-boot
```

### 2.Create boot.yaml
```yaml
---
echo:
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
2021-11-02T05:13:03.960+0800    INFO    boot/echo_entry.go:693  Bootstrapping EchoEntry.        {"eventId": "4ca7e0de-a5ff-4d94-91fa-dc5f75c599fb", "entryName": "greeter", "entryType": "EchoEntry", "port": 8080}
------------------------------------------------------------------------
endTime=2021-11-02T05:13:03.960021+08:00
startTime=2021-11-02T05:13:03.958277+08:00
elapsedNano=1743749
timezone=CST
ids={"eventId":"4ca7e0de-a5ff-4d94-91fa-dc5f75c599fb"}
app={"appName":"gin-demo","appVersion":"master-8ad197d","entryName":"greeter","entryType":"EchoEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.6","os":"darwin","realm":"*","region":"*"}
payloads={"entryName":"greeter","entryType":"EchoEntry","port":8080}
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

![](/bootstrapper/getting-started/gin-golang/gin-sw.png)

> **TV:** [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![](/bootstrapper/getting-started/gin-golang/gin-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## Add an API
We will define a Greeter API with the path of "/v1/greeter". 

### 1.Register API
> We need **echo.Echo** in order to register **HandlerFunc**. 
>
> In EchoEntry, **echo.Echo** will be initialized by bootstrapper with the name of **Echo**.

```go
package main

import (
	"context"
	"fmt"
	"github.com/labstack/echo/v4"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
	boot.GetEchoEntry("greeter").Echo.GET("/v1/greeter", Greeter)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

// Echo handler
func Greeter(ctx echo.Context) error {
	return ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.QueryParam("name")),
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

// @title RK Swagger for Echo
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
func Greeter(ctx echo.Context) error {...}
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
In order to make rk-boot finds out available swagger config files, we need to add **echo.sw.jsonPath** in boot.yaml file.
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