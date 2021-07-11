---
title: "Custom routes"
linkTitle: "Custom routes"
weight: 11
description: >
  How to add custom routes in grpc-gateway without grpc?
---

## Overview
[grpc-gateway](https://grpc-ecosystem.github.io/grpc-gateway/docs/operations/inject_router/) already support custom routing.

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
```

### 2.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Get grpc entry with name
	grpcEntry := boot.GetGrpcEntry("greeter")
    
    // !!!!!!
    // This codes should be located after Bootstrap()
	grpcEntry.GwEntry.GwMux.HandlePath("GET", "/custom", func(w http.ResponseWriter, r *http.Request, pathParams map[string]string) {
		w.Write([]byte("Custom routes!"))
	})

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 3.Validate
```shell script
$ curl "localhost:8080/custom"
Custom routes!
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)