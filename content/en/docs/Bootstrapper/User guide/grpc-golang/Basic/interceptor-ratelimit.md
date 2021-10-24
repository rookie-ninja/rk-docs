---
title: "Middleware rate limit"
linkTitle: "Middleware rate limit"
weight: 10
description: >
  Enable rate limit interceptor/middleware for the server.
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

## Rate limit options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.rateLimit.enabled | Enable rate limit interceptor | boolean | false |
| grpc.interceptors.rateLimit.algorithm | Provide algorithm, tokenBucket and leakyBucket are available options | string | tokenBucket |
| grpc.interceptors.rateLimit.reqPerSec | Request per second globally | int | 0 |
| grpc.interceptors.rateLimit.paths.path | gRPC full name | string | "" |
| grpc.interceptors.rateLimit.paths.reqPerSec | Request per second by gRPC full method name | int | 0 |

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true          # Enable common service for testing
    interceptors:
      rateLimit:
        enabled: false
        algorithm: "leakyBucket"
        reqPerSec: 0
        paths:
          - path: "/rk.api.v1.RkCommonService/Healthy"
            reqPerSec: 0
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
$ grpcurl -plaintext localhost:1949 rk.api.v1.RkCommonService.Healthy
Error invoking method "rk.api.v1.RkCommonService.Healthy": rpc error: code = ResourceExhausted desc = failed to query for service descriptor "rk.api.v1.RkCommonService": Slow down your request
```

```shell script
$ curl -X GET localhost:8080/rk/v1/healthy
{
    "error":{
        "code":429,
        "status":"Too Many Requests",
        "message":"Slow down your request.",
        "details":[
            {
                "code":8,
                "status":"ResourceExhausted",
                "message":"[from-grpc] Slow down your request."
            }
        ]
    }
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)


