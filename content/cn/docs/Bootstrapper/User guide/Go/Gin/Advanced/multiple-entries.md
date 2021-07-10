---
title: "Multiple entries"
linkTitle: "Multiple entries"
weight: 7
description: >
  How to start multiple Gin server with different port in one process?
---

## Overview
With bootstrapper, user can start multiple GinEntry at the same time. Event for multiple different entries like GRPC.

## Quick start
```yaml
gin:
  - name: alice
    port: 8080
    commonService:
      enabled: true
  - name: bob
    port: 8081
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
	boot.GetGinEntry("alice")
    // Get bob
	boot.GetGinEntry("bob")

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)