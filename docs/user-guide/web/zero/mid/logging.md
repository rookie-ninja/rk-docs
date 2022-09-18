Enable API middleware logging

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-zero
```

## Options
| options                                   | description                 | type     | default |
|-------------------------------------------|-----------------------------|----------|---------|
| zero.middleware.logging.enabled           | Enable logging middleware   | boolean  | false   |
| zero.middleware.logging.ignore            | Ignore by path              | []string | []      |
| zero.middleware.logging.loggerEncoding    | Logger format：json, console | string   | console |
| zero.middleware.logging.loggerOutputPaths | Logger output path          | []string | stdout  |
| zero.middleware.logging.eventEncoding     | Event format：json, console  | string   | console |
| zero.middleware.logging.eventOutputPaths  | Event output path           | []string | stdout  |

## Concept
1. [Event](https://github.com/rookie-ninja/rk-query)
2. [Logger](https://github.com/uber-go/zap)

### Logger
rk-boot use [zap](https://github.com/uber-go/zap) as underlying logger instance，[lumberjack](https://github.com/natefinch/lumberjack) as log rotation.

> Example
>
> ```bash
> 2022-05-09T12:21:42.016+0800    INFO    boot/zero_entry.go:761  Bootstrap zeroEntry     {"eventId": "07b3e5b2-98f2-4ba1-924c-dc283be0a163", "entryName": "greeter", "entryType": "ZeroEntry"}
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
> app={"appName":"rk-zero","appVersion":"master-xxx","entryName":"greeter","entryType":"ZeroEntry"}
> env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"10.8.0.2","os":"darwin","realm":"*","region":"*"}
> payloads={"apiMethod":"GET","apiPath":"/rk/v1/alive","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
> error={}
> counters={}
> pairs={}
> timing={}
> remoteAddr=localhost:60718
> operation=/rk/v1/alive
> resCode=200
> eventStatus=Ended
> EOE
> ```

## Quick start
### 1.Create boot.yaml
```yaml
---
zero:
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
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

> Log

```bash
------------------------------------------------------------------------
endTime=2022-05-09T12:24:49.109407+08:00
startTime=2022-05-09T12:24:49.109139+08:00
elapsedNano=268061
timezone=CST
ids={"eventId":"e2d16bee-f465-41ae-8e81-51dea2159482","traceId":"5449f93e7f245ffb70ca0a3591a353d7"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={}
timing={}
remoteAddr=localhost:63043
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 4. JSON format
```yaml
---
zero:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerEncoding: "json"
        eventEncoding: "json"
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5. Log output path
> By default, log will be rotated every 1GB and compressed by lumberjack.

```yaml
---
zero:
  - name: greeter
    ...
    middleware:
      logging:
        ...
        loggerOutputPaths: ["logs/app.log"]
        eventOutputPaths: ["logs/event.log"]
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 6. Log instance

```go
func Greeter(writer http.ResponseWriter, request *http.Request) {
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
2022-05-09T12:27:54.219+0800    INFO    zero/main.go:56 Received request        {"traceId": "511cb15a9509b0cdd2515249529cdd6a"}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 7. Change Event
rk-boot will create Event for every RPC call.

```go
func Greeter(writer http.ResponseWriter, request *http.Request) {
    event := rkzeroctx.GetEvent(request)
    event.AddPair("key", "value")

    writer.WriteHeader(http.StatusOK)
    resp := &GreeterResponse{
        Message: fmt.Sprintf("Hello %s!", request.URL.Query().Get("name")),
    }
    bytes, _ := json.Marshal(resp)
    writer.Write(bytes)
}
```

```bash
------------------------------------------------------------------------
endTime=2022-05-09T12:30:41.578903+08:00
startTime=2022-05-09T12:30:41.578769+08:00
elapsedNano=133925
timezone=CST
ids={"eventId":"fd5915c0-e480-4a48-8f4d-7a21373736de","traceId":"dc0013ee35340f4b0601f31659fcb9d1"}
app={"appName":"rk","appVersion":"local","entryName":"greeter","entryType":"ZeroEntry"}
env={"arch":"amd64","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin"}
payloads={"apiMethod":"GET","apiPath":"/v1/greeter","apiProtocol":"HTTP/1.1","apiQuery":"name=rk-dev","userAgent":"curl/7.64.1"}
counters={}
pairs={"key":"value"}
timing={}
remoteAddr=localhost:63137
operation=/v1/greeter
resCode=200
eventStatus=Ended
EOE
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)
