---
title: "正常关闭"
linkTitle: "正常关闭"
weight: 8
description: >
  当收到关闭信号时，如何能够让进程优雅地关闭？
---

## 概述
我们需要了解启动器是如何关闭进程的。

1. 用户添加可以添加自定义的关闭函数，来关闭内部资源。
1. 当从外部收到关闭信号时，启动器会按照顺序一次调用用户添加的函数。

![](/bootstrapper/user-guide/gin-golang/advanced/shutdown-hook.png)

## 快速开始
- 安装

```shell script
$ go get github.com/rookie-ninja/rk-boot/gin
```

```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-boot/gin"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()
    
    // Add shutdown hook function
	boot.AddShutdownHookFunc("shutdown-hook", func() {
		fmt.Println("shutting down")
	})

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
```shell script
...
shutting down
...
```

## _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)