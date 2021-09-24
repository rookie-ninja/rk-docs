---
title: "启动多个服务"
linkTitle: "启动多个服务"
weight: 7
description: >
  如何在启动器中，启动多个 GRPC 服务？
---

## 概述
通过启动器，用户可以在一个进程里面，启动多个 GrpcEntry。

用户还可以同时启动 Gin 服务和 GRPC 服务。

## 快速开始
```yaml
grpc:
  - name: alice
    port: 1949
    enabled: true
    commonService:
      enabled: true
  - name: bob
    port: 2008
    enabled: true
    commonService:
      enabled: true
```

### 1.访问 Entry
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	"github.com/rookie-ninja/rk-gin/interceptor/context"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()
    
    // Get alice
	boot.GetGrpcEntry("alice")
    // Get bob
	boot.GetGrpcEntry("bob")

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)