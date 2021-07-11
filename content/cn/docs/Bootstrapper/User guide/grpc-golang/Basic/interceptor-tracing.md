---
title: "调用链拦截器"
linkTitle: "调用链拦截器"
weight: 10
description: >
  启动调用链拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 GRPC 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 | 必要与否
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | GRPC 服务名称 | string | "", server won't start | Required |
| grpc.port | GRPC 服务端口 | integer | 0, server won't start | Required |
| grpc.description | GRPC 服务的描述 | string | "" | Optional |
| grpc.reflection | 启动 GRPC 反射功能 | boolean | false |

## 调用链选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.tracingTelemetry.enabled | 启动调用链拦截器 | boolean | false |
| grpc.interceptors.tracingTelemetry.exporter.file.enabled | 启动文件输出| boolean | RK |
| grpc.interceptors.tracingTelemetry.exporter.file.outputPath | 输出文件路径 | string | stdout |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.enabled | 启动 jaeger 输出 | boolean | false |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collectorEndpoint | jaeger 收集器地址 | string | localhost:16368/api/trace |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collectorUsername | jaeger 收集器用户名 | string | "" |
| grpc.interceptors.tracingTelemetry.exporter.jaeger.collectorPassword | jaeger 收集器密码 | string | "" |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 1949                      # Port of grpc entry
    reflection: true
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
$ grpcurl -plaintext localhost:1949 rk.api.v1.RkCommonService.Healthy
{
    "healthy": true
}
```

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

### 4.输出到文件
> 输出到文件的时候，会有几秒的延迟。

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

### 5.输出到 jaeger
因为 [openTelemetry](https://opentelemetry.io/) 对于 jaeger agent 的支持还有问题，我们使用 jaeger collector 来推送数据。

> 本地启动 [jaeger-all-in-one](https://www.jaegertracing.io/docs/1.23/getting-started/)
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

![jaeger](/bootstrapper/user-guide/grpc-golang/basic/grpc-jaeger-inter.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)