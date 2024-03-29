Enable Swagger UI.

## Prerequisite
We will use [swag](https://github.com/swaggo/swag) to generate swagger UI config files.

> Through [swag](https://github.com/swaggo/swag)
```bash
$ go get -u github.com/swaggo/swag/cmd/swag
```

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## Swagger options
| name             | description                                     | type     | default value |
|------------------|-------------------------------------------------|----------|---------------|
| gf.sw.enabled    | Enable Swagger UI                               | boolean  | false         |
| gf.sw.path     | Path of Swagger Web UI                          | string   | sw            |
| gf.sw.jsonPath | Path Swagger config（swagger.json）file           | []string | []            |
| gf.sw.headers  | Headers returned by server, format: [key:value] | []string | []            |

## Quick start
### 1.Create boot.yaml
> rk-boot will search docs/, api/gen/v1/ folder for swagger JSON file
>
> If swagger JSON located at other location，**gf.sw.jsonPath** needs to be configured

```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    sw:
      enabled: true
#      jsonPath: []
#      path: "sw"
#      headers: []
```

### 2.Create main.go
> In order to create Swagger JSON file with swag，comments needs to be added in codes
>
> Please refer to [swag](https://github.com/swaggo/swag)

```go
package main

import (
  "context"
  "fmt"
  "github.com/gogf/gf/v2/net/ghttp"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gf/boot"
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

  // Register handler
  entry := rkgf.GetGfEntry("greeter")
  entry.Server.BindHandler("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

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

### 3.Generate swagger config file
```bash
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

![](../../../img/example/sw.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)