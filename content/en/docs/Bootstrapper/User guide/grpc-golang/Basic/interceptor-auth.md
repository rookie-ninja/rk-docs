---
title: "Interceptor auth"
linkTitle: "Interceptor auth"
weight: 11
description: >
  Enable RPC auth interceptor/middleware for server.
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
| grpc.description | Description of grpc entry. | string | "" | Optional |

## Auth options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.auth.enabled | Enable auth interceptor | boolean | false |
| grpc.interceptors.auth.basic | Basic auth credentials as scheme of <user:pass> | []string | [] |
| grpc.interceptors.auth.apiKey | API key auth | []string | [] |
| grpc.interceptors.auth.ignorePrefix | The paths of prefix that will be ignored by interceptor | []string | [] |

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enableRkGwOption: true
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
        "details":[
            {
                "code":16,
                "status":"Unauthenticated",
                "message":"[from-grpc] Missing authorization, provide one of bellow auth header:[Basic Auth]"
            }
        ]
    }
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.With X-API-Key auth type
```yaml
---
grpc:
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
grpc:
  - name: greeter
    ...
    interceptors:
      auth:
        enabled: true                                         # Enable auth interceptor/middleware
        basic: ["user:pass"]                                  # Enable basic auth
        ignorePrefix: ["/rk.api.v1.RkCommonService/Healthy"]  # Ignoring path with prefix
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)