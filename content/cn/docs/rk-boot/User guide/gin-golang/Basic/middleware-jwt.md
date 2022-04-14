---
title: "JWT 拦截器"
linkTitle: "JWT 拦截器"
weight: 14
description: >
  启动 JWT 拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/gin
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 Gin 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.name | Gin 服务名称 | string | N/A |
| gin.port | Gin 服务端口 | integer | nil, 服务不会启动 |
| gin.enabled | Gin 服务启动开关 | bool | false |
| gin.description | Gin 服务的描述 | string | "" |

## JWT 选项
如果要想让 RK TV 以及 Swagger UI 能够通过 JWT 验证，请以如下方式添加忽略路径。

```yaml
jwt:
  ...
  ignorePrefix:
   - "/rk/v1/tv"
   - "/sw"
   - "/rk/v1/assets"
```

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.interceptors.jwt.enabled | 启动 JWT 拦截器 | boolean | false |
| gin.interceptors.jwt.signingKey | 必要, signing key. | string | "" |
| gin.interceptors.jwt.ignorePrefix | 忽略路径。 | []string | [] |
| gin.interceptors.jwt.signingKeys | 以 <key>:<value> 的 Signing keys。 | []string | [] |
| gin.interceptors.jwt.signingAlgo | 签名算法。 | string | HS256 |
| gin.interceptors.jwt.tokenLookup | 拦截器寻找 CSRF Token 的方法，请参考下面的例子。 | string | "header:Authorization" |
| gin.interceptors.jwt.authScheme | 提供 Auth Scheme | string | Bearer |

**tokenLookup** 格式

```
// Optional. Default value "header:Authorization".
// Possible values:
// - "header:<name>"
// - "query:<name>"
// - "param:<name>"
// - "cookie:<name>"
// - "form:<name>"
// Multiply sources example:
// - "header: Authorization,cookie: myowncookie"
```

## 快速开始
### 1.创建 boot.yaml
```yaml
---
gin:
  - name: greeter                     # Required
    port: 8080                        # Required
    enabled: true                     # Required
    commonService:
      enabled: true                   # Optional, default: false
    interceptors:
      jwt:
        enabled: true                 # Optional, default: false
        signingKey: "my-secret"       # Required
```

### 2.创建 main.go
```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-boot/gin"
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
- 正确的 JWT Token

```shell script
$ curl localhost:8080/rk/v1/healthy -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.EpM5XBzTJZ4J8AfoJEcJrjth8pfH28LWdjLo90sYb9g"
{"healthy":true}
```

- 非法 JWT Token
```shell script
$ curl localhost:8080/rk/v1/healthy -H "Authorization: Bearer invalid-jwt-token"
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"invalid or expired jwt",
        "details":[
            "token contains an invalid number of segments"
        ]
    }
}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)