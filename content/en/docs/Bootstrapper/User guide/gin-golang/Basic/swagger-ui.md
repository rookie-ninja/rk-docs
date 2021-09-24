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
```

## General options
> These are general options to start a gin server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.name | The name of gin server | string | N/A |
| gin.port | The port of gin server | integer | nil, server won't start |
| gin.enabled | Enable gin entry | bool | false |
| gin.description | Description of gin entry. | string | "" |

## Swagger options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.sw.enabled | Enable swagger service over gin server | boolean | false |
| gin.sw.path | The path access swagger service from web | string | /sw |
| gin.sw.jsonPath | Where the swagger.json files are stored locally | string | "" |
| gin.sw.headers | Headers would be sent to caller as scheme of [key:value] | []string | [] |

## Quick start
### 1.Create boot.yaml
In order to make rk-boot finds out available swagger config files, we need to add **gin.sw.jsonPath** in boot.yaml file.
```yaml
---
gin:
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
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// @title RK Swagger for Gin
// @version 1.0
// @description This is a greeter service with rk-boot.

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
	boot.GetGinEntry("greeter").Router.GET("/v1/greeter", Greeter)

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

![](/bootstrapper/getting-started/gin-golang/gin-sw-api.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)