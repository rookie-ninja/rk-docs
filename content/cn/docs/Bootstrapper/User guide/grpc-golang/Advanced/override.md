---
title: "覆盖启动器参数"
linkTitle: "覆盖启动器参数"
weight: 9
description: >
  当使用启动器的时候，如何通过命令行参数来覆盖 boot.yaml 里的参数？
---

## 概述
启动器支持两种命令行参数来覆盖 boot.yaml 里的参数。

- 覆盖启动器文件：(通过 \-\-rkboot)
- 覆盖 boot.yaml 里的参数 (通过 \-\-rkset)

## 快速开始
### 1.覆盖启动器文件
覆盖启动器的参数文件，我们需要 **\-\-rkboot** 作为参数。

> boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 1949
    enabled: true
    enableReflection: true
    commonService:
      enabled: true
```

> boot-override.yaml
```yaml
---
grpc:
  - name: greeter
    port: 2008
    enabled: true
    enableReflection: true
    commonService:
      enabled: true
```

> main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
)

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

> 启动：\-\-rkboot
```shell script
$ go build main.go
$ ./main --rkboot boot-override.yaml
```

> 发送请求到：1949
```shell script
$ grpcurl -plaintext localhost:1949 rk.api.v1.RkCommonService.Healthy
Failed to dial target host "localhost:1949": dial tcp [::1]:1949: connect: connection refused
```

> 发送请求到：2008
```shell script
$ grpcurl -plaintext localhost:2008 rk.api.v1.RkCommonService.Healthy
{
    "healthy": true
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 2.覆盖 boot.yaml 里的参数
覆盖启动器 boot.yaml 里的参数，我们需要 **\-\-rkset** 作为参数。

使用**逗号**（注意，是英文逗号）来覆盖多个参数。

| 类型 | 例子 |
| ---- | ---- |
| Map | app.description="This is description" |
| List | gin[0].name="alice",gin[0].port=8081 |

> boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 1949
    enabled: true
    enableReflection: true
    commonService:
      enabled: true
```

> main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
)

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

> 启动：\-\-rkset
```shell script
$ go build main.go
$ ./main --rkset "grpc[0].port=2008"
```

> 发送请求到：1949
```shell script
$ grpcurl -plaintext localhost:1949 rk.api.v1.RkCommonService.Healthy
Failed to dial target host "localhost:1949": dial tcp [::1]:1949: connect: connection refused
```

> 发送请求到：2008
```shell script
$ grpcurl -plaintext localhost:2008 rk.api.v1.RkCommonService.Healthy
{
    "healthy": true
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)