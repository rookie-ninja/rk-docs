---
title: "调用链拦截器"
linkTitle: "调用链拦截器"
weight: 9
description: >
  启动调用链拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 Echo 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.name | Echo 服务名称 | string | N/A |
| echo.port | Echo 服务端口 | integer | nil, 服务不会启动 |
| echo.enabled | Echo 服务启动开关 | bool | false |
| echo.description | Echo 服务的描述 | string | "" |

## 调用链选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.interceptors.tracingTelemetry.enabled | 启动调用链拦截器 | boolean | false |
| echo.interceptors.tracingTelemetry.exporter.file.enabled | 启动文件输出| boolean | false |
| echo.interceptors.tracingTelemetry.exporter.file.outputPath | 输出文件路径 | string | stdout |
| echo.interceptors.tracingTelemetry.exporter.jaeger.agent.enabled | jaeger agent 作为数据输出 | boolean | false |
| echo.interceptors.tracingTelemetry.exporter.jaeger.agent.host | jaeger agent 地址 | string | localhost |
| echo.interceptors.tracingTelemetry.exporter.jaeger.agent.port | jaeger agent 端口 | int | 6831 |
| echo.interceptors.tracingTelemetry.exporter.jaeger.collector.enabled | jaeger collector 作为数据输出 | boolean | false |
| echo.interceptors.tracingTelemetry.exporter.jaeger.collector.endpoint | jaeger collector 地址 | string | http://localhost:16368/api/trace |
| echo.interceptors.tracingTelemetry.exporter.jaeger.collector.username | jaeger collector 用户名 | string | "" |
| echo.interceptors.tracingTelemetry.exporter.jaeger.collector.password | jaeger collector 密码 | string | "" |

## 快速开始
### 1.创建 boot.yaml
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

### 2.创建 main.go
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

### 3.验证
> 发送请求

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

### 4.输出到文件
> 输出到文件的时候，会有几秒的延迟。

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

### 5.输出到 jaeger

> 本地启动 [jaeger-all-in-one](https://www.jaegertracing.io/docs/1.23/getting-started/)
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