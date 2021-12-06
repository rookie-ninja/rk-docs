---
title: "Multiple entries"
linkTitle: "Multiple entries"
weight: 7
description: >
  How to start multiple GoFrame server with different port in one process?
---

## Overview
With bootstrapper, user can start multiple GfEntry at the same time. Event for multiple different entries like GRPC.

## Quick start
```yaml
gf:
  - name: alice
    port: 8080
    enabled: true
    commonService:
      enabled: true
  - name: bob
    port: 8081
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
	"github.com/rookie-ninja/rk-echo/interceptor/context"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()
    
    // Get alice
	boot.GetGfEntry("alice")
    // Get bob
	boot.GetGfEntry("bob")

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)