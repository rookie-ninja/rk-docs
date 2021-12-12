---
title: "Middleware metrics"
linkTitle: "Middleware metrics"
weight: 7
description: >
  Enable RPC metrics interceptor/middleware for server.
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

## Metrics options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.interceptors.metricsProm.enabled | Enable metrics interceptor | boolean | false |

## Concept
By default, metrics interceptor/middleware will record bellow metrics.
| Metrics name | Metrics type | Description |
| ---- | ---- | ---- |
| elapsedNano | Summary | The time elapsed for RPC.|
| resCode | Counter | The counter of RPC with resCode. |
| errors | Counter | The counter of RPC with errors if occurs. |

All of three metrics have the same labels as bellow:
| Label name | Description |
| ---- | ---- |
| entryName | Gin entry name |
| entryType | Gin entry type |
| realm | OS environment variable: REALM, eg: rk |
| region | OS environment variable: REGION, eg: beijing |
| az | OS environment variable: AZ, eg: beijing-1 |
| domain | OS environment variable: DOMAIN, eg: prod |
| instance | Hostname |
| appVersion | Retrieved from [AppInfoEntry](https://github.com/rookie-ninja/rk-entry#appinfoentry) |
| appName | Retrieved from [AppInfoEntry](https://github.com/rookie-ninja/rk-entry#appinfoentry) |
| restMethod | Restful API method, eg: GET |
| restPath | Restful API path, eg: /rk/v1/healthy |
| resCode | Response code, eg: 200 |

## Quick start
### 1.Create boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    prom:
      enabled : true         # Enable prometheus client in order to export metrics
    commonService:
      enabled: true          # Enable common service for testing
    interceptors:
      metricsProm:
        enabled: true        # Enable metrics interceptor/middleware
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
> Send request

```shell script
$ curl -X GET localhost:8080/rk/v1/healthy
{"healthy":true}
```

> Prometheus client:
>
> http://localhost:8080/metrics

![prom-inter](/bootstrapper/user-guide/gin-golang/basic/gin-prom-inter.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
