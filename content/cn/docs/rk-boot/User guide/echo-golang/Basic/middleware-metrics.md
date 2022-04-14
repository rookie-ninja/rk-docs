---
title: "监控拦截器"
linkTitle: "监控拦截器"
weight: 7
description: >
  启动监控拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-echo
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 Echo 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.name | Echo 服务名称 | string | N/A |
| echo.port | Echo 服务端口 | integer | nil, 服务不会启动 |
| echo.enabled | Echo 服务启动开关 | bool | false |
| echo.description | Echo 服务的描述 | string | "" |

## 监控选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.interceptors.metricsProm.enabled | 启动监控拦截器 | boolean | false |

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
| entryName | Echo entry 名字 |
| entryType | Echo entry 类型 |
| realm | 环境变量: REALM, eg: rk |
| region | 环境变量: REGION, eg: beijing |
| az | 环境变量: AZ, eg: beijing-1 |
| domain | 环境变量: DOMAIN, eg: prod |
| instance | 本地 Hostname |
| appVersion | 从 [AppInfoEntry](https://github.com/rookie-ninja/rk-entry#appinfoentry) 获取 |
| appName | 从 [AppInfoEntry](https://github.com/rookie-ninja/rk-entry#appinfoentry) 获取 |
| restMethod | eg: GET |
| restPath | eg: /rk/v1/healthy |
| resCode | 返回码, eg: 200 |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    prom:
      enabled : true         # Enable prometheus client in order to export metrics
    commonService:
      enabled: true          # Enable common service for testing
    interceptors:
      metricsProm:
        enabled: true        # Enable metrics interceptor/middleware
```

### 2.创建 main.go
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

### 3.验证
> 发送请求

```shell script
$ curl -X GET localhost:8080/rk/v1/healthy
{"healthy":true}
```

> Prometheus 客户端:
>
> http://localhost:8080/metrics

![prom-inter](/bootstrapper/user-guide/echo-golang/basic/echo-prom-inter.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
