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
go get github.com/rookie-ninja/rk-zero
```

## Options
| options                      | description            | type     | default |
|------------------------------|------------------------|----------|---------|
| zero.middleware.meta.enabled | Enable meta middleware | boolean  | false   |
| zero.middleware.meta.prefix  | X-<Prefix>-XXX         | string   | RK      |
| zero.middleware.meta.ignore  | Ignore by path         | []string | []      |

## Quick start
### 1.Create boot.yaml
```yaml
---
zero:
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
package main

import (
  "context"
  "encoding/json"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-zero/boot"
  "github.com/zeromicro/go-zero/rest"
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
  zeroEntry := rkzero.GetZeroEntry("greeter")
  zeroEntry.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: Greeter,
  })

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
func Greeter(writer http.ResponseWriter, request *http.Request) {
  writer.WriteHeader(http.StatusOK)
  resp := &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", request.URL.Query().Get("name")),
  }
  bytes, _ := json.Marshal(resp)
  writer.Write(bytes)
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
func Greeter(writer http.ResponseWriter, request *http.Request) {
    // Override request id
    rkzeroctx.SetHeaderToClient(writer, rkmid.HeaderRequestId, "request-id-override")
    // We expect new request id attached to logger
    rkzeroctx.GetLogger(request, writer).Info("Received request")

    writer.WriteHeader(http.StatusOK)
    resp := &GreeterResponse{
        Message: fmt.Sprintf("Hello %s!", request.URL.Query().Get("name")),
    }
    bytes, _ := json.Marshal(resp)
    writer.Write(bytes)
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
2022-05-09T12:37:09.705+0800    INFO    boot/zero_entry.go:761  Bootstrap zeroEntry     {"eventId": "088bff18-ab88-406f-ba4e-20e51775038a", "entryName": "greeter", "entryType": "ZeroEntry"}
------------------------------------------------------------------------
endTime=2022-05-09T12:37:09.705544+08:00
startTime=2022-05-09T12:37:09.705463+08:00
elapsedNano=81090
timezone=CST
ids={"eventId":"088bff18-ab88-406f-ba4e-20e51775038a"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"zeroPort":8080}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5.Override header prefix
```yaml
---
zero:
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