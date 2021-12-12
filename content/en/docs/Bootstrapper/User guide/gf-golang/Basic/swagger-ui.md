---
title: "Swagger UI"
linkTitle: "Swagger UI"
weight: 2
description: >
  Enable swagger UI for server.
---

## Prerequisite
We will use [swag](https://github.com/swaggo/swag) to generate swagger UI config files.

> **Option 1:** With [RK CMD](https://github.com/rookie-ninja/rk)
```shell script
# Install RK CMD
$ go get -u github.com/rookie-ninja/rk/cmd/rk

# Install swag with rk
$ rk install swag
```

> **Option 2:** From swag the official repository:
```shell script
$ go get -u github.com/swaggo/swag/cmd/swag
```

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-gf
```

## General options
> These are general options to start a GoFrame server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.name | The name of GoFrame server | string | N/A |
| gf.port | The port of GoFrame server | integer | nil, server won't start |
| gf.enabled | Enable GoFrame entry | bool | false |
| gf.description | Description of GoFrame entry. | string | "" |

## Swagger options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.sw.enabled | Enable swagger service over GoFrame server | boolean | false |
| gf.sw.path | The path access swagger service from web | string | /sw |
| gf.sw.jsonPath | Where the swagger.json files are stored locally | string | "" |
| gf.sw.headers | Headers would be sent to caller as scheme of [key:value] | []string | [] |

## Quick start
### 1.Create boot.yaml
In order to make rk-boot finds out available swagger config files, we need to add **gf.sw.jsonPath** in boot.yaml file.
```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
      jsonPath: "docs"
#      path: "sw"        # Default value is "sw", change it as needed
#      headers: []       # Headers that will be set while accessing swagger UI main page.
```

### 2.Create main.go
> In order to generate swagger config with swag, we need to add comments as bellow.

```go
package main

import (
	"context"
	"fmt"
	"github.com/gogf/gf/v2/net/ghttp"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-gf/boot"
	"net/http"
)

// @title RK Swagger for GoFrame
// @version 1.0
// @description This is a greeter service with rk-boot.

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
	gfEntry := boot.GetEntry("greeter").(*rkgf.GfEntry)
	gfEntry.Server.BindHandler("/v1/greeter", Greeter)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}


// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
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

### 4.Validate
> **Swagger:** [http://localhost:8080/sw](http://localhost:8080/sw)

![](/bootstrapper/getting-started/gf-golang/gf-sw-api.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)