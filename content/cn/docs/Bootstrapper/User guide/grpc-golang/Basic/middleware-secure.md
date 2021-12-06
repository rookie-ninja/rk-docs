---
title: "安全拦截器"
linkTitle: "安全拦截器"
weight: 16
description: >
  启动安全拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 grpc 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.name | grpc 服务名称 | string | N/A |
| grpc.port | grpc 服务端口 | integer | nil, 服务不会启动 |
| grpc.enabled | grpc 服务启动开关 | bool | false |
| grpc.description | grpc 服务的描述 | string | "" |

## Secure 选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.secure.enabled | 启动安全拦截器 | boolean | false |
| grpc.interceptors.secure.xssProtection | X-XSS-Protection | string | "1; mode=block" |
| grpc.interceptors.secure.contentTypeNosniff | X-Content-Type-Options | string | nosniff |
| grpc.interceptors.secure.xFrameOptions | X-Frame-Options | string | SAMEORIGIN |
| grpc.interceptors.secure.hstsMaxAge | Strict-Transport-Security | int | 0 |
| grpc.interceptors.secure.hstsExcludeSubdomains | HSTS SubDomains | bool | false |
| grpc.interceptors.secure.hstsPreloadEnabled | 开启 HSTS 预加载。 | bool | false |
| grpc.interceptors.secure.contentSecurityPolicy | Content-Security-Policy | string | "" |
| grpc.interceptors.secure.cspReportOnly | Content-Security-Policy-Report-Only | bool | false |
| grpc.interceptors.secure.referrerPolicy | Referrer-Policy | string | "" |
| grpc.interceptors.secure.ignorePrefix | 忽略路径 | []string | [] |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
grpc:
  - name: greeter                     # Required
    port: 8080                        # Required
    enabled: true                     # Required
    commonService:
      enabled: true                   # Optional, default: false
    interceptors:
      secure:
        enabled: true                 # Optional, default: false
```

### 2.创建 main.go
```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package rkdemo

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
```shell script
$ curl -vs localhost:8080/rk/v1/healthy
  ...
  < X-Content-Type-Options: nosniff
  < X-Frame-Options: SAMEORIGIN
  < X-Xss-Protection: 1; mode=block
  < Date: Sun, 05 Dec 2021 18:32:24 GMT
  < Content-Length: 17
  <
  ...
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)