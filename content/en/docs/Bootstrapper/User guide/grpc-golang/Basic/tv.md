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
```

## General options
> These are general options to start a grpc server with rk-boot

| name | description | type | default value | Required |
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | The name of grpc server | string | "", server won't start | Required |
| grpc.port | The port of grpc server | integer | 0, server won't start | Required |
| grpc.description | Description of grpc entry. | string | "" | Optional |
| grpc.reflection | Enable grpc server reflection | boolean | false |

## TV options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.gw.tv.enabled | Enable RK TV | boolean | false |

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 1949                      # Port of grpc entry
    gw:
      enabled: true                 # Enable grpc-gateway, https://github.com/grpc-ecosystem/grpc-gateway
      port: 8080                    # Port of grpc-gateway
      tv:
        enabled: true               # Enable TV
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
> Validate
>
> [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)

![tv](/bootstrapper/getting-started/go/grpc/grpc-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
