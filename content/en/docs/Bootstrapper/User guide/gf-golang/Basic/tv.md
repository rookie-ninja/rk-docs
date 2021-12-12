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
go get github.com/rookie-ninja/rk-gf
```

## General options
> These are general options to start a GoFrame server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.name | The name of GoFrame server | string | N/A |
| gf.port | The port of GoFrame server | integer | nil, server won't start |
| gf.enabled | Enable GoFrame entry | bool | false |
| gf.description | Description of GoFrame entry. | string | "" |

## TV options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.tv.enabled | Enable RK TV | boolean | false |

## Quick start
### 1.Create boot.yaml
```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    tv:
      enabled: true     # Enable TV
```

### 2.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
    _ "github.com/rookie-ninja/rk-gf/boot"
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


![tv](/bootstrapper/getting-started/gf-golang/gf-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)