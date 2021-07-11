---
title: "RK TV"
linkTitle: "RK TV"
weight: 4
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
> These are general options to start a gin server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.name | The name of gin server | string | N/A |
| gin.port | The port of gin server | integer | nil, server won't start |
| gin.description | Description of gin entry. | string | "" |

## TV options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gin.tv.enabled | Enable RK TV | boolean | false |

## Quick start
### 1.Create boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    tv:
      enabled: true     # Enable TV
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


![tv](/bootstrapper/getting-started/go/gin/gin-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)