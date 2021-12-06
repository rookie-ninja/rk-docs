---
title: "Middleware auth"
linkTitle: "Middleware auth"
weight: 10
description: >
  Enable RPC auth interceptor/middleware for server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
```

## General options
> These are general options to start a GoFrame server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.name | The name of GoFrame server | string | N/A |
| gf.port | The port of GoFrame server | integer | nil, server won't start |
| gf.enabled | Enable GoFrame entry | bool | false |
| gf.description | Description of GoFrame entry. | string | "" |

## Auth options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.auth.enabled | Enable auth interceptor | boolean | false |
| gf.interceptors.auth.basic | Basic auth credentials as scheme of <user:pass> | []string | [] |
| gf.interceptors.auth.apiKey | API key auth | []string | [] |
| gf.interceptors.auth.ignorePrefix | The paths of prefix that will be ignored by interceptor | []string | [] |

## Quick start
### 1.Create boot.yaml
```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true          # Enable common service for testing
    interceptors:
      auth:
        enabled: true        # Enable auth interceptor/middleware
        basic: ["user:pass"] # Enable basic auth
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
```shell script
$ curl  -X GET localhost:8080/rk/v1/healthy
# This is RK style error code if unauthorized
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"Missing authorization, provide one of bellow auth header:[Basic Auth]",
        "details":[]
    }
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.With X-API-Key auth type
```yaml
---
gf:
  - name: greeter
    ...
    interceptors:
      auth:
        enabled: true        # Enable auth interceptor/middleware
        apiKey: ["token"]    # Enable X-API-Key auth
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 5.Ignore paths
```yaml
---
gf:
  - name: greeter
    ...
    interceptors:
      auth:
        enabled: true                     # Enable auth interceptor/middleware
        basic: ["user:pass"]              # Enable basic auth
        ignorePrefix: ["/rk/v1/healthy"]  # Ignoring path with prefix
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)