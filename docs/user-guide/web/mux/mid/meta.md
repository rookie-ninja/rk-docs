Enable meta middleware.

## Overview
Meta middleware will inject bellow information into response header.

| Header key               | Description                     |
|--------------------------|---------------------------------|
| X-Request-Id             | Auto generated request ID       |
| X-[Prefix]-App-Domain    | ENV value of DOMAIN             |
| X-[Prefix]-App-Unix-Time | Current Unix time               |
| X-[Prefix]-Received-Time | Unix time when request received |

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-mux
```

## Options
| options                     | description            | type     | default |
|-----------------------------|------------------------|----------|---------|
| mux.middleware.meta.enabled | Enable meta middleware | boolean  | false   |
| mux.middleware.meta.prefix  | X-<Prefix>-XXX         | string   | RK      |
| mux.middleware.meta.ignore  | Ignore by path         | []string | []      |

## Quick start
### 1.Create boot.yaml
```yaml
---
mux:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      meta:
        enabled: true                                      # Optional, default: false
#        ignore: [""]                                      # Optional, default: []
#        prefix: ""                                        # Optional, default: "RK"
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
$ curl -vs -X GET "localhost:8080/v1/greeter?name=rk-dev"
...
< X-Request-Id: dad72f8c-cd35-43b6-9414-493edf7e0d10
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-15T23:14:52.495+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-15T23:14:52.495+08:00
...
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 4.Override requestId
```go
func Greeter(writer http.ResponseWriter, req *http.Request) {
    // Override request id
    rkmuxctx.SetHeaderToClient(writer, rkmid.HeaderRequestId, "request-id-override")
    // We expect new request id attached to logger
    rkmuxctx.GetLogger(req, writer).Info("Received request")

    rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
        Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
    })
}
```

```bash
$ curl -vs -X GET "localhost:8080/v1/greeter?name=rk-dev"
...
< X-Request-Id: request-id-override
< X-Rk-App-Domain: *
< X-Rk-App-Name: rk
< X-Rk-App-Unix-Time: 2022-04-15T23:20:37.763436+08:00
< X-Rk-App-Version: local
< X-Rk-Received-Time: 2022-04-15T23:20:37.763436+08:00
...
{"Message":"Hello rk-dev!"}
```

> Bellow logs will be available if logging middleware was enabled.

```bash
2022-05-10T12:08:41.285+0800    INFO    mux/main.go:49  Received request        {"requestId": "request-id-override"}
------------------------------------------------------------------------
endTime=2022-05-10T12:08:41.285905+08:00
startTime=2022-05-10T12:08:41.285796+08:00
elapsedNano=108859
timezone=CST
ids={"eventId":"request-id-override","requestId":"request-id-override"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"MuxEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=127.0.0.1:55657
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5.Override header prefix
```yaml
---
mux:
  - name: greeter
    ...
    middleware:
      meta:
        enabled: true
        prefix: "Override"
```

```bash
$ curl -X GET -vs "localhost:8080/v1/greeter?name=rk-dev"
...
< X-Override-App-Domain: *
< X-Override-App-Name: rk
< X-Override-App-Unix-Time: 2022-04-15T23:22:18.087064+08:00
< X-Override-App-Version: local
< X-Override-Received-Time: 2022-04-15T23:22:18.087064+08:00
< X-Request-Id: request-id-override
...
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)