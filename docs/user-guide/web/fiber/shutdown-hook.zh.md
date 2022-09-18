当收到关闭信号时，如何能够让进程优雅地关闭？

## 概述
我们需要了解启动器是如何关闭进程的。

1. 用户添加可以添加自定义的关闭函数，关闭内部资源。
1. 当从外部收到关闭信号时，启动器会按照顺序一次调用用户添加的函数。

## 快速开始
### 1.安装

```bash
$ go get github.com/rookie-ninja/rk-boot/v2
$ go get github.com/rookie-ninja/rk-fiber
```

### 2.创建 boot.yaml
```yaml
fiber:
  - name: greeter
    port: 8080
    enabled: true
```

### 3.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  _ "github.com/rookie-ninja/rk-fiber/boot"
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

### 4.启动并 ctrl-c
```bash
...
shutting down
...
```

## _**Cheers**_
![](../../../img/user-guide/cheers.png)