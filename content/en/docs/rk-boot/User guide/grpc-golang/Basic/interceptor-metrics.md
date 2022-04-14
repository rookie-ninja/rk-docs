---
title: "Interceptor metrics"
linkTitle: "Interceptor metrics"
weight: 8
description: >
  Enable RPC metrics interceptor/middleware for server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-grpc
```

## General options
> These are general options to start a grpc server with rk-boot

| name | description | type | default value | Required |
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | The name of grpc server | string | "", server won't start | Required |
| grpc.port | The port of grpc server | integer | 0, server won't start | Required |
| grpc.enabled | Enable grpc entry | bool | false | Required |
| grpc.description | Description of grpc entry. | string | "" | Optional |
| grpc.enableReflection | Enable grpc server reflection | boolean | false | Optional |
| grpc.enableRkGwOption | Enable RK style gateway server options. | boolean | false | Optional |
| grpc.noRecvMsgSizeLimit | Disable grpc server side receive message size limit | boolean | false | Optional |
| grpc.gwMappingFilePaths | The grpc gateway mapping file path | []string | [] | Optional |

## Logging options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.metricsProm.enabled | Enable metrics interceptor | boolean | false |

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
| entryName | gRPC entry name |
| entryType | gRPC entry type |
| realm | OS environment variable: REALM, eg: rk |
| region | OS environment variable: REGION, eg: beijing |
| az | OS environment variable: AZ, eg: beijing-1 |
| domain | OS environment variable: DOMAIN, eg: prod |
| instance | Hostname |
| appVersion | Retrieved from [AppInfoEntry](https://github.com/rookie-ninja/rk-entry#appinfoentry) |
| appName | Retrieved from [AppInfoEntry](https://github.com/rookie-ninja/rk-entry#appinfoentry) |
| restMethod | Restful API method, eg: GET |
| restPath | Restful API path, eg: /rk/v1/healthy |
| type | Entry type, eg: grpc |
| resCode | Response code, eg: OK |

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
    prom:
      enabled: true                 # Enable prometheus client
    interceptors:
      metricsProm:
        enabled: true
```

### 2.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-grpc/boot"
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

![prom-inter](/bootstrapper/user-guide/grpc-golang/basic/grpc-prom-inter.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

