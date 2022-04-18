---
title: "RK TV"
linkTitle: "RK TV"
weight: 5
description: >
  Enable RK TV for server.
---

## Overview
RK TV is a web UI contains service information including APIs, process info, metrics etc.

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

## TV options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.tv.enabled | Enable RK TV | boolean | false |

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 8080                      # Port of grpc entry
    enabled: true                   # Enable grpc entry
    tv:
      enabled: true                 # Enable TV
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
> Validate
>
> [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![tv](/bootstrapper/getting-started/grpc-golang/grpc-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
