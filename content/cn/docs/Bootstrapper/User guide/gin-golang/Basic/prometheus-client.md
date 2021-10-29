---
title: "Prometheus 客户端"
linkTitle: "Prometheus 客户端"
weight: 5
description: >
  启动 Prometheus 客户端。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 Gin 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.name | Gin 服务名称 | string | N/A |
| gin.port | Gin 服务端口 | integer | nil, 服务不会启动 |
| gin.enabled | Gin 服务启动开关 | bool | false |
| gin.description | Gin 服务的描述 | string | "" |

## Prometheus 选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.prom.enabled | 启动 prometheus | boolean | false |
| gin.prom.path | Prometheus Web 路径 | string | /metrics |
| gin.prom.pusher.enabled | 启动 prometheus pusher | bool | false |
| gin.prom.pusher.jobName | JobName 将会以标签的形式添加到监控指标，并推送到远程 pushgateway | string | "" |
| gin.prom.pusher.remoteAddress | Pushgateway 远程地址, http://x.x.x.x 或者 x.x.x.x | string | "" |
| gin.prom.pusher.intervalMs | 推送间隔（毫秒） | string | 1000 |
| gin.prom.pusher.basicAuth | 远程 Pushgateway 的 Basic auth。 格式：[user:pass] | string | "" |
| gin.prom.pusher.cert.ref | rkentry.CertEntry 的引用，请参考高级指南 | string | "" |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    prom:
      enabled: true     # Enable prometheus client
#      path: "metrics"   # Default value is "metrics", set path as needed.
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
> 验证
>
> [http://localhost:8080/metrics](http://localhost:8080/metrics)

![prom](/bootstrapper/user-guide/gin-golang/basic/gin-prom.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.Prometheus 客户端中添加监控
> 我们需要先了解 Prometheus 中的如下概念。

![prom](/bootstrapper/user-guide/gin-golang/basic/gin-prom-arch.png)

| 名字 | 详情 |
| ---- | ---- |
| [MetricsSet](https://github.com/rookie-ninja/rk-prom/blob/master/metrics_set.go) | RK 自定义的结构，通过 MetricsSet 注册 Prometheus 的 Counter，Gauge，Histogram 和 Summary |
| [Prometheus Registerer](https://github.com/prometheus/client_golang/blob/master/prometheus/registry.go) | Prometheus 会通过 Registrerer 来管理 Counter，Gauge，Histogram 和 Summary  |
| [Prometheus Counter](https://prometheus.io/docs/concepts/metric_types/#counter) | Counter 是一个累积度量，表示单个单调增加的计数器，其值只能增加或重置为零 |
| [Prometheus Gauge](https://prometheus.io/docs/concepts/metric_types/#gauge) | Gauge 值可以随意加减 |
| [Prometheus Histogram](https://prometheus.io/docs/concepts/metric_types/#histogram) | Histogram 进行采样（通常是请求持续时间或响应大小之类的内容）并将它们计算在可配置的桶中，同时还提供所有观测值的总和 |
| [Prometheus Summary](https://prometheus.io/docs/concepts/metric_types/#summary) | 与 Histogram 类似，摘要样本观察（通常是请求持续时间和响应大小之类的东西） |
| Prometheus Namespace | Prometheus 监控名格式： namespace_subSystem_metricsName |
| Prometheus SubSystem | Prometheus 监控名格式： namespace_subSystem_metricsName |

```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-prom"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Create a metrics set into prometheus.Registerer
	set := rkprom.NewMetricsSet("rk", "demo", boot.GetGinEntry("greeter").PromEntry.Registerer)

	// Register counter, gauge, histogram, summary
	set.RegisterCounter("my_counter", "label")
	set.RegisterGauge("my_gauge", "label")
	set.RegisterHistogram("my_histogram", []float64{}, "label")
	set.RegisterSummary("my_summary", rkprom.SummaryObjectives, "label")

	// Increase counter, gauge, histogram, summary with label value
	set.GetCounterWithValues("my_counter", "value").Inc()
	set.GetGaugeWithValues("my_gauge", "value").Add(1.0)
	set.GetHistogramWithValues("my_histogram", "value").Observe(0.1)
	set.GetSummaryWithValues("my_summary", "value").Observe(0.1)

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 5.验证
> 验证
>
> [http://localhost:8080/metrics](http://localhost:8080/metrics)

![prom](/bootstrapper/user-guide/gin-golang/basic/gin-prom-value.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 6.推送到 Pushgateway
在 boot.yaml 中启动 pusher。
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    prom:
      enabled: true                         # Enable prometheus client
      pusher:
        enabled : true                      # Enable backend job push metrics to remote pushgateway
        jobName: "demo"                     # Name of current push job
        remoteAddress: "localhost:9091"     # Remote address of pushgateway
        intervalMs: 2000                    # Push interval in milliseconds
#        basicAuth: "user:pass"             # Basic auth of pushgateway
#        cert:
#          ref: "ref"                       # Cert reference defined in CertEntry. Please see advanced user guide for details.
```

> 在本地启动 pushgateway
```shell script
$ docker run prom/pushgateway -p 9091:9091
```

> 在本地 pushgateway 中验证
>
> [http://localhost:9091](http://localhost:909)

![pushgateway](/bootstrapper/user-guide/gin-golang/basic/gin-prom-pusher.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)