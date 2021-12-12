---
title: "超时拦截器"
linkTitle: "超时拦截器"
weight: 12
description: >
  启动超时拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-grpc
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 gRPC 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 | 必要与否
| ------ | ------ | ------ | ------ | ------ |
| grpc.name | gRPC 服务名称 | string | "", 服务不会启动 | Required |
| grpc.port | gRPC 服务端口 | integer | 0, 服务不会启动 | Required |
| grpc.enabled | gRPC 服务启动开关 ｜ bool | false | Required |
| grpc.description | gRPC 服务的描述 | string | "" | Optional |
| grpc.enableReflection | 启动 gRPC 反射功能 | boolean | false | Optional |
| grpc.enableRkGwOption | 启动 RK 自定义 Gateway Option，此 option 会默认透传所有 Header | boolean | false | Optional |
| grpc.noRecvMsgSizeLimit | 从 gRPC 服务端取消 4MB 最大接收限制 | boolean | false | Optional |
| grpc.gwMappingFilePaths | gw_mapping.yaml 文件路径，用于 RK TV | []string | [] | Optional |

## 超时选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.timeout.enabled | 启动超时拦截器 | boolean | false |
| grpc.interceptors.timeout.timeoutMs | 超时时间，毫秒 | int | 5000 |
| grpc.interceptors.timeout.paths.path | gRPC 方法路径 | string | "" |
| grpc.interceptors.timeout.paths.timeoutMs | 基于 gRPC 方法路径的超时时间，毫秒 | int | 5000 |

## 快速开始
### 1.创建 boot.yaml
我们让 GC 的超时时间定位 1 毫秒，GC 一般会超过 1 毫秒。

```yaml
---
grpc:
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
          - path: "/rk.api.v1.RkCommonService/Gc"   # Optional, default: ""
            timeoutMs: 1                            # Optional, default: 5000
```

### 2.创建 main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
    _ "github.com/rookie-ninja/rk-grpc/boot"
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
$ grpcurl -plaintext localhost:8080 rk.api.v1.RkCommonService.Gc
ERROR:
  Code: Canceled
  Message: Request timed out!
  Details:
  1)	{"@type":"type.googleapis.com/rk.api.v1.ErrorDetail","code":1,"message":"[from-grpc] Request timed out!","status":"Canceled"}
```

```shell script
$ curl -X GET localhost:8080/rk/v1/gc
{
    "error":{
        "code":408,
        "status":"Request Timeout",
        "message":"Request timed out!",
        "details":[
            {
                "code":1,
                "status":"Canceled",
                "message":"[from-grpc] Request timed out!"
            }
        ]
    }
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)



