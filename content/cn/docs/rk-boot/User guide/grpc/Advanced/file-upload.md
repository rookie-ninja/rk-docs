---
title: "文件上传"
linkTitle: "文件上传"
weight: 12
description: >
  如何让启动器支持文件上传的 Restful API？
---

## 概述
GRPC 没有给我们提供一个简单文件上传 API，或者说通过 grpc-gateway on GRPC，我们无法直接实现简单文件上传的功能。

[grpc-gateway](https://grpc-ecosystem.github.io/grpc-gateway/docs/mapping/binary_file_uploads/) 已经支持了该功能。

## 快速开始
- 安装

```shell script
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### 1.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enabled: true
    enableRkGwOption: true
```

### 2.创建 main.go
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

### 3.验证
```shell script
$ curl -X POST -F "attachment=@xxx.txt" localhost:8080/v1/files
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
