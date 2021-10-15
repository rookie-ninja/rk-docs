---
title: "Interceptor tracing"
linkTitle: "Interceptor tracing"
weight: 10
description: >
  Enable RPC tracing interceptor/middleware for server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
```

## General options
> These are general options to start a grpc server with rk-boot

| name | description | type | default value | Required |
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | The name of grpc server | string | "", server won't start | Required |
| grpc.port | The port of grpc server | integer | 0, server won't start | Required |
| grpc.enabled | Enable grpc entry | bool | false | Required |
| grpc.description | Description of grpc entry. | string | "" | Optional |

## Tracing options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.tracingTelemetry.enabled | Enable tracing interceptor | boolean | false |
| grpc.interceptors.tracingTelemetry.exporter.file.enabled | Enable file exporter | boolean | RK |
| grpc.interceptors.tracingTelemetry.exporter.file.outputPath | Export tracing info to files | string | stdout |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.agent.enabled | Export tracing info to jaeger agent | boolean | false |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.agent.host | As name described | string | localhost |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.agent.port | As name described | int | 6831 |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collector.enabled | Export tracing info to jaeger collector | boolean | false |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collector.endpoint | As name described | string | http://localhost:16368/api/trace |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collector.username | As name described | string | "" |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collector.password | As name described | string | "" |

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 8080                      # Port of grpc entry
    enabled: true                   # Enable grpc entry
    commonService:
      enabled: true                 # Enable common service for testing
    interceptors:
      tracingTelemetry:
        enabled: true
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
$ grpcurl -plaintext localhost:8080 rk.api.v1.RkCommonService.Healthy
{
    "healthy": true
}
```

> By default, tracing information will be exported to stdout

```json
[
        {
                "SpanContext": {
                        "TraceID": "479bd29a93042619372da50163ae75e3",
                        "SpanID": "3b8e3ceba6741c23",
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
                "Name": "/rk.api.v1.RkCommonService/Healthy",
                "StartTime": "2021-07-10T00:42:56.302578+08:00",
                "EndTime": "2021-07-10T00:42:56.30263999+08:00",
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
grpc:
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

![jaeger](/bootstrapper/user-guide/grpc-golang/basic/grpc-jaeger-inter.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)