Enable ratelimit middleware.

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-mux
```

## Option
| options                                   | description                   | type     | default     |
|-------------------------------------------|-------------------------------|----------|-------------|
| mux.middleware.rateLimit.enabled         | Enable ratelimit middleware   | boolean  | false       |
| mux.middleware.rateLimit.ignore          | Ignore by path                | []string | []          |
| mux.middleware.rateLimit.algorithm       | leakyBucket is supported only | string   | leakyBucket |
| mux.middleware.rateLimit.reqPerSec       | global limit                  | int      | 1000000     |
| mux.middleware.rateLimit.paths.path      | API path                      | string   | ""          |
| mux.middleware.rateLimit.paths.reqPerSec | limit value of path           | int      | 1000000     |

## Quick start
### 1.Create boot.yaml
```yaml
---
mux:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      rateLimit:
        enabled: true
        paths:
          - path: "/v1/greeter"
            reqPerSec: 0
#        algorithm: "leakyBucket"
#        reqPerSec: 100
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
> Send request

```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{
    "error":{
        "code":429,
        "status":"Too Many Requests",
        "message":"",
        "details":[
            "slow down your request"
        ]
    }
}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)