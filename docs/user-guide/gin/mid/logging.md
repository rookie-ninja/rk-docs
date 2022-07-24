Enable API middleware logging。

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

## Options
| options                     | description                        | type     | default |
|------------------------------------------|-----------------------------|----------|---------|
| gin.middleware.logging.enabled           | Enable logging middleware   | boolean  | false   |
| gin.middleware.logging.ignore            | Ignore by path              | []string | []      |
| gin.middleware.logging.loggerEncoding    | Logger format：json, console | string   | console |
| gin.middleware.logging.loggerOutputPaths | Logger output path          | []string | stdout  |
| gin.middleware.logging.eventEncoding     | Event format：json, console  | string   | console |
| gin.middleware.logging.eventOutputPaths  | Event output path           | []string | stdout  |

## Concept
1. [Event](https://github.com/rookie-ninja/rk-query)
2. [Logger](https://github.com/uber-go/zap)

### Logger
rk-boot use [zap](https://github.com/uber-go/zap) as underlying logger instance，[lumberjack](https://github.com/natefinch/lumberjack) as log rotation.

> Example
>
> ```bash
> 2021-07-05T23:35:17.104+0800    INFO    boot/gin_entry.go:631   Bootstrapping GinEntry. {"eventId": "08d11a34-472a-4785-a98c-144a3417d7f0", "entryName": "greeter", "entryType": "GinEntry", "port": 8080, "interceptorsCount": 1, "swEnabled": false, "tlsEnabled": false, "commonServiceEnabled": false, "tvEnabled": false, "promPath": "/metrics", "promPort": 8080}
> ```

### Event
rk-boot will record RPC metadata into **Event**.

| Fields      | Description                                                                                                 |
|-------------|-------------------------------------------------------------------------------------------------------------|
| endTime     | End time                                                                                                    |
| startTime   | Start time                                                                                                  |
| elapsedNano | Event elapsed（Nanoseconds）                                                                                  |
| timezone    | Timezone                                                                                                    |
| ids         | Includes eventId, requestId and traceId                                                                     |
| app         | Includes [appName, appVersion](https://github.com/rookie-ninja/rk-entry#appinfoentry), entryName, entryType |
| env         | Includes arch, domain, hostname, localIP, os 字段。这些字段来自系统环境变量（DOMAIN）。 "*" 代表环境变量为空                          |
| payloads    | Includes RPC metadata                                                                                       |
| error       | Includes errors                                                                                             |
| counters    | Set through event.SetCounter()                                                                              |
| pairs       | Set through event.AddPair()                                                                                 |
| timing      | Set through event.StartTimer() and event.EndTimer()                                                         |
| remoteAddr  | RPC remote address                                                                                          |
| operation   | RPC name                                                                                                    |
| resCode     | RPC return code                                                                                             |
| eventStatus | Ended or InProgress                                                                                         |

> Example
>
> ```bash
> ------------------------------------------------------------------------
> endTime=2021-06-25T01:30:45.144023+08:00
> startTime=2021-06-25T01:30:45.143767+08:00
> elapsedNano=255948
> timezone=CST
> ids={"eventId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","requestId":"3332e575-43d8-4bfe-84dd-45b5fc5fb104","traceId":"65b9aa7a9705268bba492fdf4a0e5652"}
> app={"appName":"rk-gin","appVersion":"master-xxx","entryName":"greeter","entryType":"GinEntry"}
> env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
> payloads={"apiMethod":"GET","apiPath":"/rk/v1/healthy","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
> error={}
> counters={}
> pairs={}
> timing={}
> remoteAddr=localhost:60718
> operation=/rk/v1/healthy
> resCode=200
> eventStatus=Ended
> EOE
> ```

## Quick start
### 1.Create boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      logging:
        enabled: true                                     # Optional, default: false
#        ignore: [""]                                      # Optional, default: []
#        loggerEncoding: "console"                         # Optional, default: "console"
#        loggerOutputPaths: ["logs/app.log"]               # Optional, default: ["stdout"]
#        eventEncoding: "console"                          # Optional, default: "console"
#        eventOutputPaths: ["logs/event.log"]              # Optional, default: ["stdout"]
```

### 2.Create main.go
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

### 3.Validation
> Send request

```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

> Logs

```bash
------------------------------------------------------------------------
endTime=2022-04-15T19:11:36.096411+08:00
startTime=2022-04-15T19:11:36.096318+08:00
elapsedNano=92426
timezone=CST
ids={"eventId":"d7d49c0e-6aaa-40b9-a374-28e21425cabf"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"GinEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.101","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=localhost:51346
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 4. JSON format
```yaml
---
gin:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerEncoding: "json"
        eventEncoding: "json"
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 5. Log output path
> By default, log will be rotated every 1GB and compressed by lumberjack.

```yaml
---
gin:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerOutputPaths: ["logs/app.log"]
        eventOutputPaths: ["logs/event.log"]
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 6. Log instance
rk-boot will add RequestId into logger instance for every RPC calls.

```go
func Greeter(ctx *gin.Context) {
	rkginctx.GetLogger(ctx).Info("Received request")

	ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
	})
}
```

```shell script
2022-04-15T19:17:01.380+0800    INFO    gin/main.go:55  Received request        {"requestId": "446c1138-1b20-48f7-a479-7b3e8a26de40"}
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)

### 7. Change Event
rk-boot will create Event for every RPC call.

```go
func Greeter(ctx *gin.Context) {
	event := rkginctx.GetEvent(ctx)
	event.AddPair("key", "value")

	ctx.JSON(http.StatusOK, &GreeterResponse{
		Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
	})
}
```

```shell script
------------------------------------------------------------------------
endTime=2022-04-15T19:17:47.088106+08:00
startTime=2022-04-15T19:17:47.08801+08:00
elapsedNano=96182
timezone=CST
ids={"eventId":"efbb5b4c-a77b-403d-b7d5-ec81662669e6","requestId":"efbb5b4c-a77b-403d-b7d5-ec81662669e6"}
app={"appName":"","appVersion":"","entryName":"greeter","entryType":"GinEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.101","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=localhost:51505
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
