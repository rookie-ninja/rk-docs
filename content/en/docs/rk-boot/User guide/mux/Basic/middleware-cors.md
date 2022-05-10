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
go get github.com/rookie-ninja/rk-mux
```

## CORS options
| options                              | description              | type     | default              |
|--------------------------------------|--------------------------|----------|----------------------|
| mux.middleware.cors.enabled          | Enable CORS middleware   | boolean  | false                |
| mux.middleware.cors.ignore           | Ignore by path           | []string | []                   |
| mux.middleware.cors.allowOrigins     | Allowed origin list      | []string | *                    |
| mux.middleware.cors.allowMethods     | Allowed http method list | []string | All http methods     |
| mux.middleware.cors.allowHeaders     | Allowed http header list | []string | Headers from request |
| mux.middleware.cors.allowCredentials | Allowed credential list  | bool     | false                |
| mux.middleware.cors.exposeHeaders    | Exposed list of headers  | []string | ""                   |
| mux.middleware.cors.maxAge           | Max age                  | int      | 0                    |

## Quick start
### 1.Create boot.yaml
```yaml
---
mux:
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
  _ "embed"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-mux/boot"
  "github.com/rookie-ninja/rk-mux/middleware"
  "net/http"
)

// @title RK Swagger for Mux
// @version 1.0
// @description This is a greeter service with rk-boot.
func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Get MuxEntry
  muxEntry := rkmux.GetMuxEntry("greeter")
  // Use *mux.Router adding handler.
  muxEntry.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(writer http.ResponseWriter, req *http.Request) {
  rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
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
Set mux.middleware.cors.allowOrigins to http://localhost:8080

```yaml
---
mux:
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