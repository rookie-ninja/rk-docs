---
title: "Middleware trace"
linkTitle: "Middleware trace"
weight: 9
description: >
  Enable tracing middleware
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## Options
| options                                                  | description                  | type     | default                          |
|----------------------------------------------------------|------------------------------|----------|----------------------------------|
| gf.middleware.trace.enabled                              | Enable tracing middleware    | boolean  | false                            |
| gf.middleware.trace.ignore                             | Ignore by path               | []string | []                               |
| gf.middleware.trace.exporter.file.enabled              | Enable file exporter         | boolean  | false                            |
| gf.middleware.trace.exporter.file.outputPath           | Path of file exporter        | string   | stdout                           |
| gf.middleware.trace.exporter.jaeger.agent.enabled      | Enable jaeger agent exporter | boolean  | false                            |
| gf.middleware.trace.exporter.jaeger.agent.host         | Hostname of jaeger agent     | string   | localhost                        |
| gf.middleware.trace.exporter.jaeger.agent.port         | Port of jaeger agent         | int      | 6831                             |
| gf.middleware.trace.exporter.jaeger.collector.enabled  | Enable jaeger collector      | boolean  | false                            |
| gf.middleware.trace.exporter.jaeger.collector.endpoint | Hostname of jaeger collector | string   | http://localhost:16368/api/trace |
| gf.middleware.trace.exporter.jaeger.collector.username | Username of jaeger collector | string   | ""                               |
| gf.middleware.trace.exporter.jaeger.collector.password | Password of jaeger collector | string   | ""                               |

## Quick start
### 1.Create boot.yaml
```yaml
---
gf:
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
#          jaeger:
#            agent:
#              enabled: false
#              host: ""
#              port: 0
#            collector:
#              enabled: true
#              endpoint: ""
#              username: ""
#              password: ""

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
        "Value": "GfEntry"
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
![](/rk-boot/user-guide/cheers.png)

### 4.Export to file
> There will be delays in seconds

```yaml
---
gf:
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
![](/rk-boot/user-guide/cheers.png)

### 5.Export to jaeger

> Start [jaeger-all-in-one](https://www.jaegertracing.io/docs/1.23/getting-started/) locally
> ```shell script
> $ docker run -d --name jaeger \
>     -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
>     -p 5775:5775/udp \
>     -p 6831:6831/udp \
>     -p 6832:6832/udp \
>     -p 5778:5778 \
>     -p 16686:16686 \
>     -p 14268:14268 \
>     -p 14250:14250 \
>     -p 9411:9411 \
>     jaegertracing/all-in-one:1.23
> ```

```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      trace:
        enabled: true
        exporter:
          jaeger:
            agent:
              enabled: true
#              host: ""                                        # Optional, default: localhost
#              port: 0                                         # Optional, default: 6831
#            collector:
#              enabled: true                                   # Optional, default: false
#              endpoint: ""                                    # Optional, default: http://localhost:14268/api/traces
#              username: ""                                    # Optional, default: ""
#              password: ""                                    # Optional, default: ""
```

> Jaeger:
>
> [http://localhost:16686/](http://localhost:16686/)

![jaeger](/rk-boot/user-guide/gin/basic/gin-trace.png)

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)