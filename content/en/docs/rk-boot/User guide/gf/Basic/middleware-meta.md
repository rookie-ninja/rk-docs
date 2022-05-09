---
title: "Middleware meta"
linkTitle: "Middleware meta"
weight: 8
description: >
  Enable meta middleware.
---

## Overview
Meta middleware will inject bellow information into response header.

| Header key               | Description                     |
|--------------------------|---------------------------------|
| X-Request-Id             | Auto generated request ID       |
| X-[Prefix]-App-Domain    | ENV value of DOMAIN             |
| X-[Prefix]-App-Unix-Time | Current Unix time               |
| X-[Prefix]-Received-Time | Unix time when request received |

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## Options
| options                     | description                        | type     | default |
|-----------------------------|------------------------|----------|-------|
| gf.middleware.meta.enabled  | Enable meta middleware | boolean  | false |
| gf.middleware.meta.prefix | X-<Prefix>-XXX         | string   | RK    |
| gf.middleware.meta.ignore | Ignore by path         | []string | []    |

## Quick start
### 1.Create boot.yaml
```yaml
---
gf:
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
  "github.com/gogf/gf/v2/net/ghttp"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gf/boot"
  "net/http"
)

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

### 3.Validate
> Send request

```shell script
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
![](/rk-boot/user-guide/cheers.png)

### 4.Override requestId
```go
func Greeter(ctx *ghttp.Request) {
    // Override request id
    rkgfctx.SetHeaderToClient(ctx, rkmid.HeaderRequestId, "request-id-override")
    // We expect new request id attached to logger
    rkgfctx.GetLogger(ctx).Info("Received request")

    ctx.Response.WriteHeader(http.StatusOK)
    ctx.Response.WriteJson(&GreeterResponse{
        Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name").String()),
    })
}
```

```shell script
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

```shell script
2022-04-15T23:21:22.452+0800    INFO    gf/main.go:59  Received request        {"requestId": "request-id-override"}
------------------------------------------------------------------------
endTime=2022-04-15T23:21:22.452678+08:00
startTime=2022-04-15T23:21:22.452528+08:00
elapsedNano=149138
timezone=CST
ids={"eventId":"request-id-override","requestId":"request-id-override"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GfEntry"}
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
![](/rk-boot/user-guide/cheers.png)

### 5.Override header prefix
```yaml
---
gf:
  - name: greeter
    ...
    middleware:
      meta:
        enabled: true
        prefix: "Override"
```

```shell script
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
![](/rk-boot/user-guide/cheers.png)