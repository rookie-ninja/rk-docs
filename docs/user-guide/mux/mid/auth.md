Enable auth middleware.

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-mux
```

## Options
| options                     | description                        | type     | default |
|-----------------------------|------------------------------------|----------|---------|
| mux.middleware.auth.enabled | Enable auth middleware             | boolean  | false   |
| mux.middleware.auth.ignore  | Ignore by path                     | []string | []      |
| mux.middleware.auth.basic   | Basic Auth info，format：<user:pass> | []string | []      |
| mux.middleware.auth.apiKey  | X-API-Key                          | []string | []      |

## Quick start
### 1.Create boot.yaml
```yaml
---
mux:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      auth:
        enabled: true
        basic: ["user:pass"]
#        ignore: [""]
#        apiKey:
#          - "keys"
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

### 3.Validate
```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
# This is RK style error code if unauthorized
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"Missing authorization, provide one of bellow auth header:[Basic Auth]",
        "details":[]
    }
}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 4.X-API-Key
```yaml
---
mux:
  - name: greeter
    ...
    middleware:
      auth:
        enabled: true
        apiKey: ["token"]
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 5.Ignore
```yaml
---
mux:
  - name: greeter
    ...
    middleware:
      auth:
        enabled: true
        basic: ["user:pass"]
        ignorePrefix: ["/v1/greeter"]
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)