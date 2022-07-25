Enable meta middleware

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
go get github.com/rookie-ninja/rk-echo
```

## Options
| options                     | description                        | type     | default |
|-----------------------------|------------------------|----------|-------|
| echo.middleware.meta.enabled | Enable meta middleware | boolean  | false |
| echo.middleware.meta.prefix  | X-<Prefix>-XXX         | string   | RK    |
| echo.middleware.meta.ignore  | Ignore by path         | []string | []    |

## Quick start
### 1.Create boot.yaml
```yaml
---
echo:
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
  "fmt"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-echo/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  echoEntry := rkecho.GetEchoEntry("greeter")
  echoEntry.Echo.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx echo.Context) error {
  return ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.QueryParam("name")),
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
![](../../../img/user-guide/cheers.png)

### 4.Override requestId
```go
func Greeter(ctx echo.Context) error {
    // Override request id
    rkechoctx.SetHeaderToClient(ctx, rkmid.HeaderRequestId, "request-id-override")
    // We expect new request id attached to logger
    rkechoctx.GetLogger(ctx).Info("Received request")

	return ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
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
2022-04-15T23:21:22.452+0800    INFO    echo/main.go:59  Received request        {"requestId": "request-id-override"}
------------------------------------------------------------------------
endTime=2022-04-15T23:21:22.452678+08:00
startTime=2022-04-15T23:21:22.452528+08:00
elapsedNano=149138
timezone=CST
ids={"eventId":"request-id-override","requestId":"request-id-override"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"EchoEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=localhost:54763
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 5.Override header prefix
```yaml
---
echo:
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
![](../../../img/user-guide/cheers.png)