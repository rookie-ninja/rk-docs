---
title: "浏览下载静态文件"
linkTitle: "浏览下载静态文件"
weight: 11
description: >
  如何通过网页浏览和下载静态文件?
---

## 概述
rk-boot 提供了一个方便的方法，让用户快速实现通过网页浏览和下载静态文件的功能。

目前，rk-boot 支持如下文件源。如果用户希望支持更多的文件源，可以通过实现 http.FileSystem 接口来实现。
- 文帝文件系统
- [pkger](https://github.com/markbates/pkger)

## 快速开始
```yaml
---
gin:
  - name: greeter                     # Required
    port: 8080                        # Required
    enabled: true                     # Required
    static:
      enabled: true                   # Optional, default: false
      path: "/rk/v1/static"           # Optional, default: /rk/v1/static
      sourceType: local               # Required, options: pkger, local
      sourcePath: "."                 # Required, full path of source directory
```

### 1.访问自定义 Path
> [http://localhost:8080/rk/v1/static](http://localhost:8080/rk/v1/static)

![](/bootstrapper/user-guide/gin-golang/advanced/static-file-handler.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

## 从 pkger 读取文件 (嵌入式静态文件)
[pkger](https://github.com/markbates/pkger) 是一个可以把静态文件，嵌入到 .go 文件的工具。

### 1.下载 pkger
```shell script
go get github.com/markbates/pkger/cmd/pkger
```

### 2.在 main.go 中添加 pkger.Include() 
我们将会把当前文件夹里的所有文件，嵌入到 pkger.go 文件中，并放在 internal/pkger.go。

请注意 import 中的 [_ "github.com/rookie-ninja/rk-demo/internal"]，一定要这么引入。

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

### 3.生成 pkger.go
```shell script
pkger -o internal
```

### 4.配置 boot.yaml
pkger 会使用 module 来区分不同的 package，所以，sourcePath 里，我们添加了相应 module 的前缀。

```yaml
---
gin:
  - name: greeter                                             # Required
    port: 8080                                                # Required
    enabled: true                                             # Required
    static:
      enabled: true                                           # Optional, default: false
      path: "/rk/v1/static"                                   # Optional, default: /rk/v1/static
      sourceType: pkger                                       # Required, options: pkger, local
      sourcePath: "github.com/rookie-ninja/rk-demo:/"         # Required, full path of source directory
```

### 5.文件夹结构
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

### 6.验证
> [http://localhost:8080/rk/v1/static](http://localhost:8080/rk/v1/static)

![](/bootstrapper/user-guide/gin-golang/advanced/static-file-handler-pkger.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)





