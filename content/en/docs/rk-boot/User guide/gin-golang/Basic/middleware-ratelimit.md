---
title: "Middleware rate limit"
linkTitle: "Middleware rate limit"
weight: 10
description: >
  Enable rate limit interceptor/middleware for the server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot/gin
```


## General options
> These are general options to start a gin server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.name | The name of gin server | string | N/A |
| gin.port | The port of gin server | integer | nil, server won't start |
| gin.enabled | Enable gin entry | bool | false |
| gin.description | Description of gin entry. | string | "" |

## Rate limit options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.interceptors.rateLimit.enabled | Enable rate limit interceptor | boolean | false |
| gin.interceptors.rateLimit.algorithm | Provide algorithm, tokenBucket and leakyBucket are available options | string | tokenBucket |
| gin.interceptors.rateLimit.reqPerSec | Request per second globally | int | 0 |
| gin.interceptors.rateLimit.paths.path | Full path | string | "" |
| gin.interceptors.rateLimit.paths.reqPerSec | Request per second by full path | int | 0 |

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
	_ "github.com/rookie-ninja/rk-boot/gin"
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


