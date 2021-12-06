---
title: "Static file handler with Web UI"
linkTitle: "Static file handler with Web UI"
weight: 11
description: >
  How to list download static files via Web UI?
---

## Overview
rk-boot provide an easy way to start a Web UI of downloading static files from server.

Currently, rk-boot support bellow source of static files located. User can implement own http.FileSystem to support other remote repository like S3.
- local
- [pkger](https://github.com/markbates/pkger)

## Quick start
```yaml
---
gf:
  - name: greeter                     # Required
    port: 8080                        # Required
    enabled: true                     # Required
    static:
      enabled: true                   # Optional, default: false
      path: "/rk/v1/static"           # Optional, default: /rk/v1/static
      sourceType: local               # Required, options: pkger, local
      sourcePath: "."                 # Required, full path of source directory
```

### 1.Access via static file handler path
> [http://localhost:8080/rk/v1/static](http://localhost:8080/rk/v1/static)

![](/bootstrapper/user-guide/gf-golang/advanced/static-file-handler.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## Read from pkger (embeded static files)
[pkger](https://github.com/markbates/pkger) is a tool for embedding static files into Go binaries.

### 1.Download pkger
```shell script
go get github.com/markbates/pkger/cmd/pkger
```

### 2.Add pkger.Include() in main.go
We will embed files in the current directory and move pkger.go file into internal/ folder.

```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
	"context"
	"github.com/markbates/pkger"
	"github.com/rookie-ninja/rk-boot"
	// Must be present in order to make pkger load embedded files into memory.
	_ "github.com/rookie-ninja/rk-demo/internal"
)

func init() {
	// This is used while running pkger CLI
	pkger.Include("./")
}

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 3.Generate pkger.go
```shell script
pkger -o internal
```

### 4.Configure boot.yaml
pkger will use module name as package, so we need to specify sourcePath has the prefix of current module

```yaml
---
gf:
  - name: greeter                                             # Required
    port: 8080                                                # Required
    enabled: true                                             # Required
    static:
      enabled: true                                           # Optional, default: false
      path: "/rk/v1/static"                                   # Optional, default: /rk/v1/static
      sourceType: pkger                                       # Required, options: pkger, local
      sourcePath: "github.com/rookie-ninja/rk-demo:/"         # Required, full path of source directory
```

### 5.Directory hierarchy
```
.
├── boot.yaml
├── go.mod
├── go.sum
├── internal
│   └── pkged.go
└── main.go

1 directory, 5 files
```

### 6.Validate
> [http://localhost:8080/rk/v1/static](http://localhost:8080/rk/v1/static)

![](/bootstrapper/user-guide/gf-golang/advanced/static-file-handler-pkger.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)





