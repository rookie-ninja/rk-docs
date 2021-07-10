---
title: "RK TV"
linkTitle: "RK TV"
weight: 5
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
> 启动器包含了如下通用选项，这些选项是启动 GRPC 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 | 必要与否
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | GRPC 服务名称 | string | "", server won't start | Required |
| grpc.port | GRPC 服务端口 | integer | 0, server won't start | Required |
| grpc.description | GRPC 服务的描述 | string | "" | Optional |
| grpc.reflection | 启动 GRPC 反射功能 | boolean | false |

## TV 选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.tv.enabled | 启动 RK TV | boolean | false |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 1949                      # Port of grpc entry
    gw:
      enabled: true                 # Enable grpc-gateway, https://github.com/grpc-ecosystem/grpc-gateway
      port: 8080                    # Port of grpc-gateway
      tv:
        enabled: true               # Enable TV
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

![tv](/bootstrapper/getting-started/go/grpc/grpc-tv.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
