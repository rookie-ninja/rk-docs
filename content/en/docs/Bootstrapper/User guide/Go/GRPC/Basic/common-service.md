---
title: "Common Service"
linkTitle: "Common Service"
weight: 4
description: >
  Enable common service for server.
---

## Overview
Common service contains builtin API commonly used by user.

| grpc-gateway Path | GRPC Method | Description |
| ---- | ---- | ---- |
| /rk/v1/apis | rk.api.v1.RkcCommonService.Apis | List APIs in current GrpcEntry. |
| /rk/v1/certs | rk.api.v1.RkcCommonService.Certs | List CertEntry. |
| /rk/v1/configs | rk.api.v1.RkcCommonService.Configs | List ConfigEntry. |
| /rk/v1/deps | rk.api.v1.RkcCommonService.Deps | List dependencies related application, entire contents of go.mod file would be returned. |
| /rk/v1/entries | rk.api.v1.RkcCommonService.Entries | List all Entries. |
| /rk/v1/gc | rk.api.v1.RkcCommonService.Gc | Trigger GC |
| /rk/v1/healthy | rk.api.v1.RkcCommonService.Healthy | Get application healthy status. |
| /rk/v1/info | rk.api.v1.RkcCommonService.Info | Get application and process info. |
| /rk/v1/license | rk.api.v1.RkcCommonService.License | Get license related application, entire contents of LICENSE file would be returned. |
| /rk/v1/logs | rk.api.v1.RkcCommonService.Logs | List logger related entries. |
| /rk/v1/git | rk.api.v1.RkcCommonService.Git | Get git information. |
| /rk/v1/readme | rk.api.v1.RkcCommonService.Readme | Get contents of README file. |
| /rk/v1/req | rk.api.v1.RkcCommonService.Req | List prometheus metrics of requests. |
| /rk/v1/sys | rk.api.v1.RkcCommonService.Sys | Get OS stat. |
| /rk/v1/tv | rk.api.v1.RkcCommonService.Tv | Get HTML page of /tv. |

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
| grpc.reflection | Enable grpc server reflection | boolean | false |

## CommonService options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.commonService.enabled | Enable embedded common service | boolean | false |

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 1949
    reflection: true    # Enable reflection in order to use grpcurl for validation
    commonService:
      enabled: true     # Enable common service
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
> Install grpcurl from official site.
> https://github.com/fullstorydev/grpcurl

```shell script
# List grpc services at port 1949 without TLS
# Expect RkCommonService since we enabled common services.
$ grpcurl -plaintext localhost:1949 list                           
grpc.reflection.v1alpha.ServerReflection
rk.api.v1.RkCommonService

# List grpc methods in rk.api.v1.RkCommonService
$ grpcurl -plaintext localhost:1949 list rk.api.v1.RkCommonService            
rk.api.v1.RkCommonService.Apis
rk.api.v1.RkCommonService.Certs
rk.api.v1.RkCommonService.Configs
rk.api.v1.RkCommonService.Deps
rk.api.v1.RkCommonService.Entries
rk.api.v1.RkCommonService.Gc
rk.api.v1.RkCommonService.GwErrorMapping
rk.api.v1.RkCommonService.Healthy
rk.api.v1.RkCommonService.Info
rk.api.v1.RkCommonService.License
rk.api.v1.RkCommonService.Logs
rk.api.v1.RkCommonService.Readme
rk.api.v1.RkCommonService.Req
rk.api.v1.RkCommonService.Sys
rk.api.v1.RkCommonService.Git

# Send request to rk.api.v1.RkCommonService.Healthy
$ grpcurl -plaintext localhost:1949 rk.api.v1.RkCommonService.Healthy
{
    "healthy": true
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.Enable swagger UI
> In order to visualise common service in swagger UI, we need to enable swagger in boot.yaml as bellow.

```yaml
---
grpc:
  - name: greeter
    port: 1949
    reflection: true    # Enable reflection in order to use grpcurl for validation
    commonService:
      enabled: true     # Enable common service
    gw:
      enabled: true
      port: 8080
      sw:
        enabled: true
```

> Validate
>
> [http://localhost:8080/sw](http://localhost:8080/sw)

![sw-common](/bootstrapper/getting-started/go/grpc/grpc-sw.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
