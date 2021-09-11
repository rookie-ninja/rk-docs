---
title: "File uploads"
linkTitle: "File uploads"
weight: 12
description: >
  How to upload file with grpc-gateway?
---

## Overview
[grpc-gateway](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/binary_file_uploads/) already support file uploads.

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 8080                      # Port of grpc entry
```

### 2.Create main.go
```go
package main

import (
	"context"
	"fmt"
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
```shell script
$ curl -X POST -F "attachment=@xxx.txt" localhost:8080/v1/files
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)