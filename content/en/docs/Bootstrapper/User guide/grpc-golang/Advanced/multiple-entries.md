---
title: "Multiple entries"
linkTitle: "Multiple entries"
weight: 7
description: >
  How to start multiple Grpc server with different port in one process?
---

## Overview
With bootstrapper, user can start multiple GrpcEntry at the same time. Event for multiple different entries like Gin.

## Quick start
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

### 1.Access entries
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