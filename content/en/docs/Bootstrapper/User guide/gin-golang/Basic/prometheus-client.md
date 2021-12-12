---
title: "Prometheus client"
linkTitle: "Prometheus client"
weight: 5
description: >
  Enable prometheus client for server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-gin
```


## General options
> These are general options to start a gin server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.name | The name of gin server | string | N/A |
| gin.port | The port of gin server | integer | nil, server won't start |
| gin.enabled | Enable gin entry | bool | false |
| gin.description | Description of gin entry. | string | "" |

## Prometheus options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.prom.enabled | Enable prometheus | boolean | false |
| gin.prom.path | Path of prometheus | string | /metrics |
| gin.prom.pusher.enabled | Enable prometheus pusher | bool | false |
| gin.prom.pusher.jobName | Job name would be attached as label while pushing to remote pushgateway | string | "" |
| gin.prom.pusher.remoteAddress | PushGateWay address, could be form of http://x.x.x.x or x.x.x.x | string | "" |
| gin.prom.pusher.intervalMs | Push interval in milliseconds | string | 1000 |
| gin.prom.pusher.basicAuth | Basic auth used to interact with remote pushgateway, form of [user:pass] | string | "" |
| gin.prom.pusher.cert.ref | Reference of rkentry.CertEntry | string | "" |

## Quick start
### 1.Create boot.yaml
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

### 2.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-gin/boot"
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
> Validate
>
> [http://localhost:8080/metrics](http://localhost:8080/metrics)

![prom](/bootstrapper/user-guide/gin-golang/basic/gin-prom.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.Add value to prometheus client
> In order to add custom metrics into current prometheus client, we need to clarify bellow concept.

![prom](/bootstrapper/user-guide/gin-golang/basic/gin-prom-arch.png)

| Name | Description |
| ---- | ---- |
| [MetricsSet](https://github.com/rookie-ninja/rk-prom/blob/master/metrics_set.go) | RK defined data structure could be used register Counter, Gauge, Histogram and Summary |
| [Prometheus Registerer](https://github.com/prometheus/client_golang/blob/master/prometheus/registry.go) | User need to register MetricsSet into registerer |
| [Prometheus Counter](https://prometheus.io/docs/concepts/metric_types/#counter) | A counter is a cumulative metric that represents a single monotonically increasing counter whose value can only increase or be reset to zero on restart. |
| [Prometheus Gauge](https://prometheus.io/docs/concepts/metric_types/#gauge) | A gauge is a metric that represents a single numerical value that can arbitrarily go up and down. |
| [Prometheus Histogram](https://prometheus.io/docs/concepts/metric_types/#histogram) | A histogram samples observations (usually things like request durations or response sizes) and counts them in configurable buckets. It also provides a sum of all observed values. |
| [Prometheus Summary](https://prometheus.io/docs/concepts/metric_types/#summary) | Similar to a histogram, a summary samples observations (usually things like request durations and response sizes). |
| Prometheus Namespace | Prometheus metrics was consist of namespace_subSystem_metricsName |
| Prometheus SubSystem | Prometheus metrics was consist of namespace_subSystem_metricsName |

```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-gin/boot" 
	"github.com/rookie-ninja/rk-prom"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Create a metrics set into prometheus.Registerer
	ginEntry := boot.GetEntry("greeter").(*rkgin.GinEntry)
	set := rkprom.NewMetricsSet("rk", "demo", ginEntry.PromEntry.Registerer)

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

### 5.Validate metrics
> Validate
>
> [http://localhost:8080/metrics](http://localhost:8080/metrics)

![prom](/bootstrapper/user-guide/gin-golang/basic/gin-prom-value.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 6.Push metrics to Pushgateway
Turn on pusher in boot.yaml.
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

> Start pushgateway locally for testing
```shell script
$ docker run prom/pushgateway -p 9091:9091
```

> Validate metrics at local pushgateway
>
> [http://localhost:9091](http://localhost:909)

![pushgateway](/bootstrapper/user-guide/gin-golang/basic/gin-prom-pusher.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)