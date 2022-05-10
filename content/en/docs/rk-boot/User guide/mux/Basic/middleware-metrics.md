---
title: "Middleware prometheus"
linkTitle: "Middleware prometheus"
weight: 7
description: >
  Enable Prometheus middleware
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-mux
```

## Options
| options                      | description                  | type     | default |
|------------------------------|------------------------------|----------|---------|
| mux.middleware.prom.enabled | Enable Prometheus middleware | boolean  | false   |
| mux.middleware.prom.ignore  | Ignore by path               | []string | []      |

## Concept
Prometheus middleware will record bellow information

| Fields      | Type    | Description              |
|-------------|---------|--------------------------|
| elapsedNano | Summary | RPC elapsed time in nano |
| resCode     | Counter | Counter of response code |

Labels as bellow:

| Label      | Description                   |
|------------|-------------------------------|
| entryName  | Entry name                    |
| entryType  | Entry type                    |
| domain     | ENV value of DOMAIN, eg: prod |
| instance   | Hostname                      |
| restMethod | eg: GET                       |
| restPath   | eg: /rk/v1/alive              |
| resCode    | Response code, eg: 200        |

Example

```shell
rk_prom_elapsedNano{domain="*",entryName="greeter",entryType="MuxEntry",instance="lark.local",resCode="200",restMethod="GET",restPath="/v1/greeter",quantile="0.5"} 88645
...
rk_prom_resCode{domain="*",entryName="greeter",entryType="MuxEntry",instance="lark.local",resCode="200",restMethod="GET",restPath="/v1/greeter"} 1
```

## Quick start
### 1.Create boot.yaml
```yaml
---
mux:
  - name: greeter
    port: 8080
    enabled: true
    prom:
      enabled : true
    middleware:
      prom:
        enabled: true
#        ignore: [""]
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

```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

> Prometheus client:
>
> http://localhost:8080/metrics

![prom-inter](/rk-boot/user-guide/gin/basic/gin-prom-inter.png)

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
