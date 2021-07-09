---
title: "Shutdown hook"
linkTitle: "Shutdown hook"
weight: 8
description: >
  How to add shutdown hook function while receiving shutdown signal?
---

## Overview
We need to understand how bootstrapper stop process.

1. Add shutdown hook functions by user
1. Call functions as soon as receive signal from outside

![](/bootstrapper/user-guide/go/gin/advanced/shutdown-hook.png)

## Getting started
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
    
    // Add shutdown hook function
	boot.AddShutdownHookFunc("shutdown-hook", func() {
		fmt.Println("shutting down")
	})

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
```shell script
...
shutting down
...
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)