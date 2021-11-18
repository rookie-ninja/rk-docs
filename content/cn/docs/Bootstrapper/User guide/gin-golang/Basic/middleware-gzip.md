---
title: "超时拦截器"
linkTitle: "超时拦截器"
weight: 13
description: >
  启动超时拦截器。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot
```

## 通用选项
> 启动器包含了如下通用选项，这些选项是启动 Gin 服务的必要选项。

| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.name | Gin 服务名称 | string | N/A |
| gin.port | Gin 服务端口 | integer | nil, 服务不会启动 |
| gin.enabled | Gin 服务启动开关 | bool | false |
| gin.description | Gin 服务的描述 | string | "" |

## Gzip 选项
| 名字 | 描述 | 类型 | 默认值 |
| ------ | ------ | ------ | ------ |
| gin.interceptors.gzip.enabled | 启动 Gzip 压缩/解压缩拦截器 | boolean | false |
| gin.interceptors.gzip.level | 压缩比例, 选项为 noCompression, bestSpeed, bestCompression, defaultCompression, huffmanOnly. | string | defaultCompression |

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
      gzip:
        enabled: true                 # Optional, default: false
        level: bestSpeed              # Optional, options: [noCompression, bestSpeed， bestCompression, defaultCompression, huffmanOnly]
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
	"fmt"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot"
	"io"
	"net/http"
	"strings"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
	boot.GetGinEntry("greeter").Router.POST("/v1/post", post)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

// PostResponse Response of Greeter.
type PostResponse struct {
	ReceivedMessage string
}

// post Handler.
func post(ctx *gin.Context) {
	buf := new(strings.Builder)
	io.Copy(buf, ctx.Request.Body)

	ctx.JSON(http.StatusOK, &PostResponse{
		ReceivedMessage: fmt.Sprintf("Received %s!", buf.String()),
	})
}
```

### 3.验证
> 发送请求

```shell script
$ echo 'this is message' | gzip | curl --compressed --data-binary @- -H "Content-Encoding: gzip" -H "Accept-Encoding: gzip" localhost:8080/v1/post
{"ReceivedMessage":"this is message\n"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)