---
title: "Static file download"
linkTitle: "Static file download"
weight: 11
description: >
  Enable a web UI for static file downloading
---

## Overview
rk-boot provide an easy way to start a web UI for downloading static files.

rk-boot support download from bellow location. User can also implement http.FileSystem to extend it.

- local file system
- embed.FS

## Quick start
### 1.Install

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-grpc/v2
```

### 2.Create.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
#   gwPort: 8081                  # Optional, default: gateway port will be the same as grpc port if not provided
    enabled: true
    static:
      enabled: true
      path: "/static"
      sourceType: local
      sourcePath: "."
```

### 3.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot/v2"
	_ "github.com/rookie-ninja/rk-grpc/v2/boot"
)

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}
```

### 4.Validate
> [http://localhost:8080/static](http://localhost:8080/static)

![](../../../img/user-guide/gin/advanced/static-file-handler.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
