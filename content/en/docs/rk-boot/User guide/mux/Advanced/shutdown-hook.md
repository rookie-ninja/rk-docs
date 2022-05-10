---
title: "Shutdown hook"
linkTitle: "Shutdown hook"
weight: 8
description: >
  Graceful shutdown.
---

## Quick start
### 1.Install

```shell script
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-mux
```

### 2.Create boot.yaml
```yaml
mux:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.Create main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  _ "github.com/rookie-ninja/rk-mux/boot"
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
```shell
...
shutting down
...
```

## _**Cheers**_
![](/rk-boot/user-guide/cheers.png)