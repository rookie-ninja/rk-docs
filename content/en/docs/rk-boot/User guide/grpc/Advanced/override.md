---
title: "Override bootstrapper"
linkTitle: "Override bootstrapper"
weight: 9
description: >
  Is there any way to override boot.yaml or values in boot.yaml at start time?
---

## Overview
Bootstrapper support two kinds of ways to override bootstrapper configs.
- Override config file (by \-\-rkboot)
- Override values in config file (by \-\-rkset)

## Quick start
- Install

```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-grpc
```

### 1.Override bootstrapper config file
In order to override bootstrapper file path, **\-\-rkboot** needs to be passed.

> boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 1949
    enabled: true
    enableReflection: true
    commonService:
      enabled: true
```

> boot-override.yaml
```yaml
---
grpc:
  - name: greeter
    port: 2008
    enabled: true
    enableReflection: true
    commonService:
      enabled: true
```

> main.go
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

> Start server with \-\-rkboot args
```shell script
$ go build main.go
$ ./main --rkboot boot-override.yaml
```

> Send request to 1949
```shell script
$ grpcurl -plaintext localhost:1949 rk.api.v1.RkCommonService.Healthy
Failed to dial target host "localhost:1949": dial tcp [::1]:1949: connect: connection refused
```

> Send request to 2008
```shell script
$ grpcurl -plaintext localhost:2008 rk.api.v1.RkCommonService.Healthy
{
    "healthy": true
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 2.Override values in boot.yaml
In order to override bootstrapper file path, **\-\-rkset** needs to be passed.

Use **comma** to separate multiple overrides.

| Type | Example |
| ---- | ---- |
| Map | app.description="This is description" |
| List | grpc[0].name="alice",grpc[0].port=8081 |

> boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 1949
    enabled: true
    enableReflection: true
    commonService:
      enabled: true
```

> main.go
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

> Start server with \-\-rkset args
```shell script
$ go build main.go
$ ./main --rkset "grpc[0].port=2008"
```

> Send request to 1949
```shell script
$ grpcurl -plaintext localhost:1949 rk.api.v1.RkCommonService.Healthy
Failed to dial target host "localhost:1949": dial tcp [::1]:1949: connect: connection refused
```

> Send request to 2008
```shell script
$ grpcurl -plaintext localhost:2008 rk.api.v1.RkCommonService.Healthy
{
    "healthy": true
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)