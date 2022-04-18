---
title: "Interceptor timeout"
linkTitle: "Interceptor timeout"
weight: 11
description: >
  Enable timeout interceptor/middleware for the server.
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

## Timeout options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.timeout.enabled | Enable timeout interceptor | boolean | false |
| grpc.interceptors.timeout.timeoutMs | Global timeout in milliseconds. | int | 5000 |
| grpc.interceptors.timeout.paths.path | Full path | string | "" |
| grpc.interceptors.timeout.paths.timeoutMs | Timeout in milliseconds by full path | int | 5000 |

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
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
          - path: "/rk.api.v1.RkCommonService/Gc"   # Optional, default: ""
            timeoutMs: 1                            # Optional, default: 5000
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
$ grpcurl -plaintext localhost:8080 rk.api.v1.RkCommonService.Gc
ERROR:
  Code: Canceled
  Message: Request timed out!
  Details:
  1)	{"@type":"type.googleapis.com/rk.api.v1.ErrorDetail","code":1,"message":"[from-grpc] Request timed out!","status":"Canceled"}
```

```shell script
$ curl -X GET localhost:8080/rk/v1/gc
{
    "error":{
        "code":408,
        "status":"Request Timeout",
        "message":"Request timed out!",
        "details":[
            {
                "code":1,
                "status":"Canceled",
                "message":"[from-grpc] Request timed out!"
            }
        ]
    }
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)


