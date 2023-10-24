---
title: "Shutdown hook"
linkTitle: "Shutdown hook"
weight: 8
description: >
  Graceful shutdown.
---

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### 2.Create boot.yaml
```yaml
grpc:
  - name: greeter
    port: 8080
#   gwPort: 8081                  # Optional, default: gateway port will be the same as grpc port if not provided
    enabled: true
```

### 3.Create main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  _ "github.com/rookie-ninja/rk-grpc/v2/boot"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Add shutdown hook function
  boot.AddShutdownHookFunc("shutdown-hook", func() {
    fmt.Println("shutting down")
  })

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}
```

### 4.Start and ctrl-c
```bash
...
shutting down
...
```

## _**Cheers**_
![](../../../img/user-guide/cheers.png)