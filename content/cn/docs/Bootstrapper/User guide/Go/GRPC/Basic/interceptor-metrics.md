---
title: "监控拦截器"
linkTitle: "监控拦截器"
weight: 8
description: >
  启动监控拦截器。
---

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

## 监控选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.metricsProm.enabled | 启动监控拦截器 | boolean | false |

## 概念
监控拦截器会默认记录如下监控。

| 监控项 | 数据类型 | 详情 |
| ---- | ---- | ---- |
| elapsedNano | Summary | RPC 耗时 |
| resCode | Counter | 基于 RPC 返回码的计数器 |
| errors | Counter | 基于 RPC 错误的计数器 |

上述三项监控，都有如下的标签。

| 标签 | 详情 |
| ---- | ---- |
| entryName | Gin entry 名字 |
| entryType | Gin entry 类型 |
| realm | 环境变量: REALM, eg: rk |
| region | 环境变量: REGION, eg: beijing |
| az | 环境变量: AZ, eg: beijing-1 |
| domain | 环境变量: DOMAIN, eg: prod |
| instance | 本地 Hostname |
| appVersion | 从 [AppInfoEntry](https://github.com/rookie-ninja/rk-entry#appinfoentry) 获取 |
| appName | 从 [AppInfoEntry](https://github.com/rookie-ninja/rk-entry#appinfoentry) 获取 |
| restMethod | 如果启动了 grpc-gateway，并且请求是以 http 形式发过来的，则会记录当中。 eg: GET |
| restPath | 如果启动了 grpc-gateway，并且请求是以 http 形式发过来的，则会记录当中。 eg: /rk/v1/healthy |
| grpcService | GRPC 服务名称 ｜
| grpcMethod | GRPC 方法名称
| grpcType | GRPC 类型。eg: UnaryServer |
| resCode | 返回码, eg: OK |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter                   # Name of grpc entry
    port: 1949                      # Port of grpc entry
    commonService:
      enabled: true                 # Enable common service for testing
    gw:
      enabled: true                 # Enable grpc-gateway, https://github.com/grpc-ecosystem/grpc-gateway
      port: 8080                    # Port of grpc-gateway
      prom:
        enabled: true               # Enable prometheus client
    interceptors:
      metricsProm:
        enabled: true
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
> 发送请求

```shell script
$ curl -X GET localhost:8080/rk/v1/healthy
{"healthy":true}
```

> Prometheus 客户端:
>
> http://localhost:8080/metrics

![prom-inter](/bootstrapper/user-guide/go/grpc/basic/grpc-prom-inter.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

