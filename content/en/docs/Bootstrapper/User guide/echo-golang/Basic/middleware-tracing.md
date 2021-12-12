---
title: "Middleware tracing"
linkTitle: "Middleware tracing"
weight: 9
description: >
  Enable RPC tracing interceptor/middleware for server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-echo
```

## General options
> These are general options to start a echo server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| echo.name | The name of echo server | string | N/A |
| echo.port | The port of echo server | integer | nil, server won't start |
| echo.enabled | Enable echo entry | bool | false |
| echo.description | Description of echo entry. | string | "" |

## Tracing options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| echo.interceptors.tracingTelemetry.enabled | Enable tracing interceptor | boolean | false |
| echo.interceptors.tracingTelemetry.exporter.file.enabled | Enable file exporter | boolean | false |
| echo.interceptors.tracingTelemetry.exporter.file.outputPath | Export tracing info to files | string | stdout |
| echo.interceptors.tracingTelemetry.exporter.jaeger.agent.enabled | Export tracing info to jaeger agent | boolean | false |
| echo.interceptors.tracingTelemetry.exporter.jaeger.agent.host | As name described | string | localhost |
| echo.interceptors.tracingTelemetry.exporter.jaeger.agent.port | As name described | int | 6831 |
| echo.interceptors.tracingTelemetry.exporter.jaeger.collector.enabled | Export tracing info to jaeger collector | boolean | false |
| echo.interceptors.tracingTelemetry.exporter.jaeger.collector.endpoint | As name described | string | http://localhost:16368/api/trace |
| echo.interceptors.tracingTelemetry.exporter.jaeger.collector.username | As name described | string | "" |
| echo.interceptors.tracingTelemetry.exporter.jaeger.collector.password | As name described | string | "" |

## Quick start
### 1.Create boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true          # Enable common service for testing
    interceptors:
      tracingTelemetry:
        enabled: true        # Enable tracing interceptor/middleware
        exporter:
          file:
            enabled: true
            outputPath: "stdout"
```

### 2.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
    _ "github.com/rookie-ninja/rk-echo/boot"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 3.Validate
> Send request

```shell script
$ curl -X GET localhost:8080/rk/v1/healthy
{"healthy":true}
```

```json
[
        {
                "SpanContext": {
                        "TraceID": "4b59250ca328450045efe841a443e77d",
                        "SpanID": "4d3545e0c0da05c3",
                        "TraceFlags": "01",
                        "TraceState": null,
                        "Remote": false
                },
                "Parent": {
                        "TraceID": "00000000000000000000000000000000",
                        "SpanID": "0000000000000000",
                        "TraceFlags": "00",
                        "TraceState": null,
                        "Remote": true
                },
                "SpanKind": 2,
                "Name": "/rk/v1/healthy",
                "StartTime": "2021-07-06T03:06:50.871097+08:00",
                "EndTime": "2021-07-06T03:06:50.871167677+08:00",
                ....
                "InstrumentationLibrary": {
                        "Name": "greeter",
                        "Version": "semver:0.20.0"
                }
        }
]
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.Export tracing log to file
> There will be delay for the couple of seconds before flushing to log file.

```yaml
---
echo:
  - name: greeter
    ...
    interceptors:
      tracingTelemetry:
        enabled: true                       # Enable tracing interceptor/middleware
        exporter:
          file:
            enabled: true
            outputPath: "logs/tracing.log"  # Log to files
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.Export to jaeger

> Start [jaeger-all-in-one](https://www.jaegertracing.io/docs/1.23/getting-started/) for testing
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
echo:
  - name: greeter
    ...
    interceptors:
      tracingTelemetry:
        enabled: true                                          # Enable tracing interceptor/middleware
        exporter:
          jaeger:
            agent:
              enabled: true                                    # Export to jaeger agent
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
> http://localhost:16686/

![jaeger](/bootstrapper/user-guide/echo-golang/basic/echo-jaeger-inter.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)