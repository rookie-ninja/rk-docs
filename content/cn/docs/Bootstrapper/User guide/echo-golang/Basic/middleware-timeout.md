---
title: "超时拦截器"
linkTitle: "超时拦截器"
weight: 11
description: >
  启动超时拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 Echo 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.name | Echo 服务名称 | string | N/A |
| echo.port | Echo 服务端口 | integer | nil, 服务不会启动 |
| echo.enabled | Echo 服务启动开关 | bool | false |
| echo.description | Echo 服务的描述 | string | "" |

## 超时选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| echo.interceptors.timeout.enabled | 启动超时拦截器 | boolean | false |
| echo.interceptors.timeout.timeoutMs | 超时时间，毫秒 | int | 5000 |
| echo.interceptors.timeout.paths.path | gRPC 方法路径 | string | "" |
| echo.interceptors.timeout.paths.timeoutMs | 基于访问路径的超时时间，毫秒 | int | 5000 |

## 快速开始
### 1.创建 boot.yaml
我们让 GC 的超时时间定位 1 毫秒，GC 一般会超过 1 毫秒。

```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true                                 # Enable common service for testing
    interceptors:
      timeout:
        enabled: true                               # Optional, default: false
        timeoutMs: 5000                             # Optional, default: 5000
        paths: 
          - path: "/rk/v1/gc"                       # Optional, default: ""
            timeoutMs: 1                            # Optional, default: 5000
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
$ curl -X GET localhost:8080/rk/v1/gc
{
    "error":{
        "code":408,
        "status":"Request Timeout",
        "message":"Request timed out!",
        "details":[]
    }
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)