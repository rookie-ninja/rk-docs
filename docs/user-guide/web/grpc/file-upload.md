Implement file upload API with grpc-gateway

## Overview
User can implement file upload API with grpc-gateway.

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
#   gwPort: 8081                  # Optional, default: gateway port will be the same as grpc port if not provided
    enabled: true
    enableRkGwOption: true
```

### 2.Create main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-grpc/v2/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Bootstrap
  boot.Bootstrap(context.Background())

  // Get grpc entry with name
  grpcEntry := rkgrpc.GetGrpcEntry("greeter")

  // Attachment upload from http/s handled manually
  grpcEntry.GwMux.HandlePath("POST", "/v1/files", handleBinaryFileUpload)

  // Wait for shutdown sig
  boot.WaitForShutdownSig(context.Background())
}

func handleBinaryFileUpload(w http.ResponseWriter, req *http.Request, params map[string]string) {
  err := req.ParseForm()
  if err != nil {
    http.Error(w, fmt.Sprintf("failed to parse form: %s", err.Error()), http.StatusBadRequest)
    return
  }

  f, header, err := req.FormFile("attachment")
  if err != nil {
    http.Error(w, fmt.Sprintf("failed to get file 'attachment': %s", err.Error()), http.StatusBadRequest)
    return
  }
  defer f.Close()

  fmt.Println(header)

  //
  // Now do something with the io.Reader in `f`, i.e. read it into a buffer or stream it to a gRPC client side stream.
  // Also `header` will contain the filename, size etc of the original file.
  //
}
```

### 3.Validate
```bash
$ curl -X POST -F "attachment=@xxx.txt" localhost:8080/v1/files
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
