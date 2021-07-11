---
title: "RK TV"
linkTitle: "RK TV"
weight: 4
description: >
  启动 RK TV。
---

## 概述
RK TV 是一个 Web 界面，用户可以通过 RK TV 获取服务以及进程的信息，API 信息，请求监控等。

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 Gin 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.name | Gin 服务名称 | string | N/A |
| gin.port | Gin 服务端口 | integer | nil, server won't start |
| gin.description | Gin 服务的描述 | string | "" |

## TV 选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.tv.enabled | 启动 RK TV | boolean | false |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    tv:
      enabled: true     # Enable TV
```

### 2.创建 main.go
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

### 3.验证
> 验证
>
> [http://localhost:8080/rk/v1/tv](http://localhost:8080/rk/v1/tv)


![tv](/bootstrapper/getting-started/go/gin/gin-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)