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
- 安装

```shell script
$ go get github.com/rookie-ninja/rk-boot
$ go get github.com/rookie-ninja/rk-echo
```

### 1.覆盖启动器文件
覆盖启动器的参数文件，我们需要 **\-\-rkboot** 作为参数。

> boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true
```

> boot-override.yaml
```yaml
---
echo:
  - name: greeter
    port: 8081
    enabled: true
    commonService:
      enabled: true
```

> main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-echo/boot"
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

> 发送请求到：8080
```shell script
$ curl localhost:8080/rk/v1/healthy
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

> 发送请求到：8081
```shell script
$ curl localhost:8081/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 2.覆盖 boot.yaml 里的参数
覆盖启动器 boot.yaml 里的参数，我们需要 **\-\-rkset** 作为参数。

使用**逗号**（注意，是英文逗号）来覆盖多个参数。

| 类型 | 例子 |
| ---- | ---- |
| Map | app.description="This is description" |
| List | echo[0].name="alice",echo[0].port=8081 |

> boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true
```

> main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-echo/boot"
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
$ ./main --rkset "echo[0].port=8081"
```

> 发送请求到：8080
```shell script
$ curl localhost:8080/rk/v1/healthy
curl: (7) Failed to connect to localhost port 8080: Connection refused
```

> 发送请求到：8081
```shell script
$ curl localhost:8081/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)