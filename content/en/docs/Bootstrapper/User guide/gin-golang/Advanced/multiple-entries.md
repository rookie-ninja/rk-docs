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
- Install

```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-gin
```

```yaml
gin:
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
	"github.com/rookie-ninja/rk-gin/boot"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()
    
    // Get alice
	boot.GetEntry("alice").(*rkgin.GinEntry)
    // Get bob
    boot.GetEntry("bob").(*rkgin.GinEntry)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)