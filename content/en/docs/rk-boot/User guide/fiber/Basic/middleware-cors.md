---
title: "Middleware CORS"
linkTitle: "Middleware CORS"
weight: 15
description: >
  Enable CORS middleware
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-fiber
```

## CORS options
| options                               | description                        | type     | default |
|---------------------------------------|--------------------------|----------|----------------------|
| fiber.middleware.cors.enabled         | Enable CORS middleware   | boolean  | false                |
| fiber.middleware.cors.ignore          | Ignore by path           | []string | []                   |
| fiber.middleware.cors.allowOrigins    | Allowed origin list      | []string | *                    |
| fiber.middleware.cors.allowMethods    | Allowed http method list | []string | All http methods     |
| fiber.middleware.cors.allowHeaders        | Allowed http header list | []string | Headers from request |
| fiber.middleware.cors.allowCredentials | Allowed credential list  | bool     | false                |
| fiber.middleware.cors.exposeHeaders    | Exposed list of headers  | []string | ""                   |
| fiber.middleware.cors.maxAge           | Max age                  | int      | 0                    |

## Quick start
### 1.Create boot.yaml
```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      cors:
        enabled: true
        allowOrigins:
          - "http://localhost:*"
#        ignore: [""]
#        allowCredentials: false
#        allowHeaders: []
#        allowMethods: []
#        exposeHeaders: []
#        maxAge: 0
```

### 2.Create main.go
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

### 3.Create cors.html
```html
<!DOCTYPE html>
<html>
<body>

<h1>CORS Test</h1>

<p>Call http://localhost:8080/v1/greeter</p>

<script type="text/javascript">
    window.onload = function() {
        var apiUrl = 'http://localhost:8080/v1/greeter';
        fetch(apiUrl).then(response => response.json()).then(data => {
            document.getElementById("res").innerHTML = data["Message"]
        }).catch(err => {
            document.getElementById("res").innerHTML = err
        });
    };
</script>

<h4>Response: </h4>
<p id="res"></p>

</body>
</html>
```

### 4.Directory hierarchy
```shell script
.
├── boot.yaml
├── cors.html
├── go.mod
├── go.sum
└── main.go

0 directories, 5 files
```

### 5.Validate
Open cors.html

![](/rk-boot/user-guide/gin/basic/cors-success.png)

### 6.Blocked CORS
Set fiber.middleware.cors.allowOrigins to http://localhost:8080

```yaml
---
fiber:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      cors:
        enabled: true
        allowOrigins:
          - "http://localhost:8080"
```

![](/rk-boot/user-guide/gin/basic/cors-fail.png)

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)