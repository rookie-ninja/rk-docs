---
title: "Override boot.yaml"
linkTitle: "Override boot.yaml"
weight: 9
description: >
  Override values in boot.yaml with flags and environment variable
---

## Overview
User can override values in boot.yaml with flags and environment variable

- Override value in boot.yaml with (\-\-rkset)

## Quick start
### 1.Install

```shell script
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-zero
```

### 2.Create boot.yaml
```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
```

### 4.Create main.go

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

### 5.Change Port with flag
Follows format of【zero[0].port】if value was in List.

```shell
$ go run main.go --rkset zero[0].port=8081
2022-05-09T13:05:52.632+0800    INFO    entry/util.go:332       Found flag to override, applying...     {"flags": ["zero[0].port=8081"]}
2022-05-09T13:05:52.632+0800    INFO    boot/zero_entry.go:761  Bootstrap zeroEntry     {"eventId": "dd06a3d9-f437-49d4-ac4f-05ebceb257f0", "entryName": "greeter", "entryType": "ZeroEntry"}
------------------------------------------------------------------------
endTime=2022-05-09T13:05:52.633023+08:00
startTime=2022-05-09T13:05:52.632981+08:00
elapsedNano=41924
timezone=CST
ids={"eventId":"dd06a3d9-f437-49d4-ac4f-05ebceb257f0"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"zeroPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```

> Send request to：8081
```shell script
$ curl "localhost:8081/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)

### 6.Override Port with environment variable
【RK_】should be included as prefix while using environment variables.

Example：RK_ZERO_0_PORT

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
  "os"
)

func main() {
  os.Setenv("RK_ZERO_0_PORT", "8081")

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

```shell
$ go run main.go
2022-05-09T13:06:42.437+0800    INFO    entry/util.go:299       Found ENV to override, applying...      {"env": ["RK_ZERO_0_PORT=8081 => zero[0].port=8081"]}
2022-05-09T13:06:42.438+0800    INFO    boot/zero_entry.go:761  Bootstrap zeroEntry     {"eventId": "2d5bd046-a0bb-4887-a252-449691d7fde8", "entryName": "greeter", "entryType": "ZeroEntry"}
------------------------------------------------------------------------
endTime=2022-05-09T13:06:42.438376+08:00
startTime=2022-05-09T13:06:42.438324+08:00
elapsedNano=52141
timezone=CST
ids={"eventId":"2d5bd046-a0bb-4887-a252-449691d7fde8"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"zeroPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```