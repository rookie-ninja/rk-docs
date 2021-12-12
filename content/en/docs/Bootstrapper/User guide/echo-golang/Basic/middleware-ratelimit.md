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

## Rate limit options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| echo.interceptors.rateLimit.enabled | Enable rate limit interceptor | boolean | false |
| echo.interceptors.rateLimit.algorithm | Provide algorithm, tokenBucket and leakyBucket are available options | string | tokenBucket |
| echo.interceptors.rateLimit.reqPerSec | Request per second globally | int | 0 |
| echo.interceptors.rateLimit.paths.path | Full path | string | "" |
| echo.interceptors.rateLimit.paths.reqPerSec | Request per second by full path | int | 0 |

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
      rateLimit:
        enabled: true
        algorithm: "leakyBucket"
        reqPerSec: 100
        paths:
          - path: "/rk/v1/healthy"
            reqPerSec: 0
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
{
    "error":{
        "code":429,
        "status":"Too Many Requests",
        "message":"",
        "details":[
            "slow down your request"
        ]
    }
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)


