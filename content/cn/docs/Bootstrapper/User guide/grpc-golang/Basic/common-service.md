---
title: "通用 API"
linkTitle: "通用 API"
weight: 4
description: >
  启动通用 API。
---

## 概述
通用 API 内嵌到了启动器里，不需要额外的实现。

| 路径 | 描述 |
| ---- | ---- |
| /rk/v1/apis | 返回 GinEntry 里的 API 列表。 |
| /rk/v1/certs | 返回 CertEntry 列表。 |
| /rk/v1/configs | 返回 ConfigEntry 列表。 |
| /rk/v1/deps | 返回 go.mod 文件内容。 |
| /rk/v1/entries | 返回所有 Entry 列表。 |
| /rk/v1/gc | 触发 GC。 |
| /rk/v1/healthy | 返回服务健康状态。 |
| /rk/v1/info | 返回服务和进程信息。|
| /rk/v1/license |返回 LICENSE 文件内容。 |
| /rk/v1/logs | 返回日志相关的 Entry 列表。 |
| /rk/v1/git | 返回 git 相关信息。 |
| /rk/v1/readme | 返回 README 文件内容。 |
| /rk/v1/req | 返回请求相关的 prometheus 监控信息。|
| /rk/v1/sys | 返回系统 CPU & 内存信息。|
| /rk/v1/tv | 返回 RK TV HTML 界面。 |

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
| grpc.enableReflection | 启动 GRPC 反射功能 | boolean | false |

## CommonService options
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.commonService.enabled | 启动通用 API | boolean | false |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter
    port: 8080
    enableReflection: true    # Enable reflection in order to use grpcurl for validation
    commonService:
      enabled: true           # Enable common service
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
> Install grpcurl from official site.
> https://github.com/fullstorydev/grpcurl

```shell script
# List grpc services at port 8080 without TLS
# Expect RkCommonService since we enabled common services.
$ grpcurl -plaintext localhost:8080 list                           
grpc.reflection.v1alpha.ServerReflection
rk.api.v1.RkCommonService

# List grpc methods in rk.api.v1.RkCommonService
$ grpcurl -plaintext localhost:8080 list rk.api.v1.RkCommonService            
rk.api.v1.RkCommonService.Apis
rk.api.v1.RkCommonService.Certs
rk.api.v1.RkCommonService.Configs
rk.api.v1.RkCommonService.Deps
rk.api.v1.RkCommonService.Entries
rk.api.v1.RkCommonService.Gc
rk.api.v1.RkCommonService.GwErrorMapping
rk.api.v1.RkCommonService.Healthy
rk.api.v1.RkCommonService.Info
rk.api.v1.RkCommonService.License
rk.api.v1.RkCommonService.Logs
rk.api.v1.RkCommonService.Readme
rk.api.v1.RkCommonService.Req
rk.api.v1.RkCommonService.Sys
rk.api.v1.RkCommonService.Git

# Send request to rk.api.v1.RkCommonService.Healthy
$ grpcurl -plaintext localhost:8080 rk.api.v1.RkCommonService.Healthy
{
    "healthy": true
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.启动 swagger 界面
> 如果想要为通用 API 启动 Swagger 界面，我们需要在 boot.yaml 中，启动 Swagger。

```yaml
---
grpc:
  - name: greeter
    port: 8080
    enableReflection: true    # Enable reflection in order to use grpcurl for validation
    commonService:
      enabled: true           # Enable common service
    sw:
      enabled: true
```

> 验证
>
> [http://localhost:8080/sw](http://localhost:8080/sw)

![sw-common](/bootstrapper/getting-started/grpc-golang/grpc-sw.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)
