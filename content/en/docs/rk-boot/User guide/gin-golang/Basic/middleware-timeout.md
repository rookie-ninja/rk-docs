---
title: "Middleware timeout"
linkTitle: "Middleware timeout"
weight: 11
description: >
  Enable timeout interceptor/middleware for the server.
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

## Timeout options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.interceptors.timeout.enabled | Enable timeout interceptor | boolean | false |
| gin.interceptors.timeout.timeoutMs | Global timeout in milliseconds. | int | 5000 |
| gin.interceptors.timeout.paths.path | Full path | string | "" |
| gin.interceptors.timeout.paths.timeoutMs | Timeout in milliseconds by full path | int | 5000 |

## Quick start
### 1.Create boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true                                 # Enable common service for testing
    interceptors:
      timeout:
        enabled: true                               # Optional, default: false
        timeoutMs: 5000                             # Optional, default: 5000
        paths: 
          - path: "/rk/v1/gc"                       # Optional, default: ""
            timeoutMs: 1                            # Optional, default: 5000
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
$ curl -X GET localhost:8080/rk/v1/gc
{
    "error":{
        "code":408,
        "status":"Request Timeout",
        "message":"Request timed out!",
        "details":[]
    }
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)


