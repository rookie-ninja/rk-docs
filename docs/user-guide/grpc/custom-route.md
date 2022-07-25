How to define custom route in grpc-gateway?

## Overview
User can add custom HTTP handler into grpc-gateway.

## Quick start
- Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
```

### 2.Create main.go
```go
package main

import (
  "context"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "net/http"
)

func main() {
  boot := rkboot.NewBoot()

  // Bootstrap
  boot.Bootstrap(context.TODO())

  // register grpc
  entry := rkgrpc.GetGrpcEntry("greeter")

  // !!!!!!
  // This codes should be located after Bootstrap()
  entry.GwMux.HandlePath("GET", "/custom", func(w http.ResponseWriter, r *http.Request, pathParams map[string]string) {
    w.Write([]byte("Custom routes!"))
  })

  // Wait for shutdown sig
  boot.WaitForShutdownSig(context.TODO())
}
```

### 3.Validate
```bash
$ curl "localhost:8080/custom"
Custom routes!
```

### _**Cheers**_
![](../../img/user-guide/cheers.png)