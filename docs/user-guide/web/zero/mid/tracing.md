Enable tracing middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-zero
```

## Options
| options                                                  | description                  | type     | default                          |
|----------------------------------------------------------|------------------------------|----------|----------------------------------|
| zero.middleware.trace.enabled                            | Enable tracing middleware    | boolean  | false                            |
| zero.middleware.trace.ignore                             | Ignore by path               | []string | []                               |
| zero.middleware.trace.exporter.file.enabled              | Enable file exporter         | boolean  | false                            |
| zero.middleware.trace.exporter.file.outputPath           | Path of file exporter        | string   | stdout                           |
| zero.middleware.trace.exporter.otlp.enabled              | Enable otlp exporter   | boolean  | false                            |
| zero.middleware.trace.exporter.otlp.endpoint             | Hostname of OTLP            | string   | localhost                        |

## Quick start
### 1.Create boot.yaml
```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      trace:
        enabled: true
        exporter:
          file:
            enabled: true
            outputPath: "stdout"
#        ignore: [""]
#          otlp:
#            enabled: true
#            endpoint: ""
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
$ curl -X GET "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

```json
{
  "Name": "/v1/greeter",
  "SpanContext": {
    "TraceID": "507aaae07729a9f7cae7ec25ee7f0ede",
    "SpanID": "ea556bd79f2f2732",
    "TraceFlags": "01",
    "TraceState": "",
    "Remote": false
  },
  "Parent": {
    "TraceID": "00000000000000000000000000000000",
    "SpanID": "0000000000000000",
    "TraceFlags": "00",
    "TraceState": "",
    "Remote": true
  },
  "SpanKind": 2,
  "StartTime": "2022-04-15T23:28:34.054496+08:00",
  "EndTime": "2022-04-15T23:28:34.054594239+08:00",
  "Attributes": [
    ...
    {
      "Key": "http.status_code",
      "Value": {
        "Type": "INT64",
        "Value": 200
      }
    }
  ],
  ...
  "Resource": [
    {
      "Key": "service.entryName",
      "Value": {
        "Type": "STRING",
        "Value": "greeter"
      }
    },
    {
      "Key": "service.entryType",
      "Value": {
        "Type": "STRING",
        "Value": "ZeroEntry"
      }
    },
  ],
  "InstrumentationLibrary": {
    "Name": "greeter",
    "Version": "semver:1.4.0",
    "SchemaURL": ""
  }
}
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 4.Export to file
> There will be delays in seconds

```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      trace:
        enabled: true
        exporter:
          file:
            enabled: true
            outputPath: "logs/tracing.log"
```

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)

### 5.Export to jaeger

> Start [jaeger-all-in-one](https://www.jaegertracing.io/docs/1.50/getting-started/) locally
> ```bash
> $ docker run --rm --name jaeger \
>     -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
>     -p 6831:6831/udp \
>     -p 6832:6832/udp \
>     -p 5778:5778 \
>     -p 16686:16686 \
>     -p 4317:4317 \
>     -p 4318:4318 \
>     -p 14250:14250 \
>     -p 14268:14268 \
>     -p 14269:14269 \
>     -p 9411:9411 \
>     jaegertracing/all-in-one:1.50
> ```

```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      trace:
        enabled: true
        exporter:
          otpl:
            enabled: true
            endpoint: "localhost:4317"
```

> Jaeger:
>
> [http://localhost:16686/](http://localhost:16686/)

![jaeger](../../../../img/user-guide/gin/basic/gin-trace.png)

### _**Cheers**_
![](../../../../img/user-guide/cheers.png)