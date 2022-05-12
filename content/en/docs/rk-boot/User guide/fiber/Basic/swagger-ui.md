---
title: "Swagger UI"
linkTitle: "Swagger UI"
weight: 2
description: >
  Enable Swagger UI.
---

## Prerequisite
We will use [swag](https://github.com/swaggo/swag) to generate swagger UI config files.

> Through [swag](https://github.com/swaggo/swag)
```shell script
$ go get -u github.com/swaggo/swag/cmd/swag
```

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## Swagger options
| name             | description                                     | type     | default value |
|------------------|-------------------------------------------------|----------|---------------|
| fiber.sw.enabled  | Enable Swagger UI                               | boolean  | false         |
| fiber.sw.path     | Path of Swagger Web UI                          | string   | sw            |
| fiber.sw.jsonPath | Path Swagger config（swagger.json）file           | string   | ""            |
| fiber.sw.headers  | Headers returned by server, format: [key:value] | []string | []            |

## Quick start
### 1.Create boot.yaml
> rk-boot will search docs/, api/gen/v1/ folder for swagger JSON file
>
> If swagger JSON located at other location，**fiber.sw.jsonPath** needs to be configured

```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
#      jsonPath: ""
#      path: "sw"
#      headers: []
```

### 2.Create main.go
> In order to create Swagger JSON file with swag，comments needs to be added in codes
>
> Please refer to [swag](https://github.com/swaggo/swag)

```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.

package main

import (
  "context"
  "fmt"
  "github.com/gofiber/fiber/v2"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-fiber/boot"
  "net/http"
)

// @title Swagger Example API
// @version 1.0
// @description This is a sample rk-demo server.
// @termsOfService http://swagger.io/terms/

// @securityDefinitions.basic BasicAuth

// @contact.name API Support
// @contact.url http://www.swagger.io/support
// @contact.email support@swagger.io

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html
func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // Register handler
  entry := rkfiber.GetFiberEntry("greeter")
  entry.App.Get("/v1/greeter", Greeter)
  // This is required!!!
  entry.RefreshFiberRoutes()

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter
// @Id 1
// @Tags Hello
// @version 1.0
// @Param name query string true "name"
// @produce application/json
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(ctx *fiber.Ctx) error {
  ctx.Response().SetStatusCode(http.StatusOK)
  return ctx.JSON(&GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.Generate swagger config file
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

![](/rk-boot/example/sw.png)

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)