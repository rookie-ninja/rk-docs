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
```

## General options
> These are general options to start a gin server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.name | The name of gin server | string | N/A |
| gin.port | The port of gin server | integer | nil, server won't start |
| gin.enabled | Enable gin entry | bool | false |
| gin.description | Description of gin entry. | string | "" |

## Tracing options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.interceptors.tracingTelemetry.enabled | Enable tracing interceptor | boolean | false |
| gin.interceptors.tracingTelemetry.exporter.file.enabled | Enable file exporter | boolean | RK |
| gin.interceptors.tracingTelemetry.exporter.file.outputPath | Export tracing info to files | string | stdout |
| gin.interceptors.tracingTelemetry.exporter.jaeger.enabled | Export tracing info jaeger | boolean | false |
| gin.interceptors.tracingTelemetry.exporter.jaeger.collectorEndpoint | As name described | string | localhost:16368/api/trace |
| gin.interceptors.tracingTelemetry.exporter.jaeger.collectorUsername | As name described | string | "" |
| gin.interceptors.tracingTelemetry.exporter.jaeger.collectorPassword | As name described | string | "" |

## Quick start
### 1.Create boot.yaml
```yaml
---
gin:
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
grpc:
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
Currently, because [openTelemetry](https://opentelemetry.io/) for jaeger agent exporter is not stable yet, we will use jaeger collector instead.

> Start [jaeger-all-in-one](https://www.jaegertracing.io/docs/1.23/getting-started/) for testing
> ```shell script
> $ docker run -d --name jaeger \
      -e COLLECTOR_ZIPKIN_HOST_PORT=:9411 \
      -p 5775:5775/udp \
      -p 6831:6831/udp \
      -p 6832:6832/udp \
      -p 5778:5778 \
      -p 16686:16686 \
      -p 14268:14268 \
      -p 14250:14250 \
      -p 9411:9411 \
      jaegertracing/all-in-one:1.23
> ```

```yaml
---
grpc:
  - name: greeter
    ...
    interceptors:
      tracingTelemetry:
        enabled: true                                          # Enable tracing interceptor/middleware
        exporter:
          jaeger:
            enabled: true                                      # Export to jaeger
#            collectorEndpoint: "localhost:16368/api/trace"
#            collectorUsername: ""
#            collectorPassword: ""
```

> Jaeger:
> 
> http://localhost:16686/

![jaeger](/bootstrapper/user-guide/gin-golang/basic/gin-jaeger-inter.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)