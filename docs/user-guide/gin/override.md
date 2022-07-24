Override values in boot.yaml with flags and environment variable

## Overview
User can override values in boot.yaml with flags and environment variable

- Override value in boot.yaml with (\-\-rkset)

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-gin/v2
```

### 2.Create boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.Create main.go

```go
package main

import (
  "context"
  "fmt"
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gin/v2/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgin.GetGinEntry("greeter")
  entry.Router.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *gin.Context) {
  ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 4.Change Port with flag
Follows format of【gin[0].port】if value was in List.

```bash
$ go run main.go --rkset gin[0].port=8081
2022-04-17T00:13:52.166+0800    INFO    entry/util.go:332       Found flag to override, applying...     {"flags": ["gin[0].port=8081"]}
2022-04-17T00:13:52.167+0800    INFO    boot/gin_entry.go:656   Bootstrap GinEntry      {"eventId": "19a961f4-b866-4047-9741-aa0a2d00055b", "entryName": "greeter", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T00:13:52.167194+08:00
startTime=2022-04-17T00:13:52.167146+08:00
elapsedNano=47911
timezone=CST
ids={"eventId":"19a961f4-b866-4047-9741-aa0a2d00055b"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GinEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"ginPort":8081}
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
```bash
$ curl "localhost:8081/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)

### 6.Override Port with environment variable
【RK_】should be included as prefix while using environment variables.

Example：RK_GIN_0_PORT

```go
package main

import (
  "context"
  "fmt"
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gin/v2/boot"
  "net/http"
)

func main() {
  os.Setenv("RK_GIN_0_PORT", "8081")
  
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgin.GetGinEntry("greeter")
  entry.Router.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *gin.Context) {
  ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

```bash
$ go run main.go
2022-04-17T00:20:38.452+0800    INFO    entry/util.go:299       Found ENV to override, applying...      {"env": ["RK_GIN_0_PORT=8081 => gin[0].port=8081"]}
2022-04-17T00:20:38.453+0800    INFO    boot/gin_entry.go:656   Bootstrap GinEntry      {"eventId": "06f271e8-7eb8-4c1a-abef-f710afe18021", "entryName": "greeter", "entryType": "GinEntry"}
------------------------------------------------------------------------
endTime=2022-04-17T00:20:38.453386+08:00
startTime=2022-04-17T00:20:38.453345+08:00
elapsedNano=41474
timezone=CST
ids={"eventId":"06f271e8-7eb8-4c1a-abef-f710afe18021"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"GinEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.101.5","os":"darwin"}
payloads={"ginPort":8081}
counters={}
pairs={}
timing={}
remoteAddr=localhost
operation=Bootstrap
resCode=OK
eventStatus=Ended
EOE
```