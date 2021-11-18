---
title: "Middleware gzip"
linkTitle: "Middleware gzip"
weight: 13
description: >
  Enable gzip interceptor/middleware for the server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
```

## General options
> These are general options to start a echo server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| echo.name | The name of echo server | string | N/A |
| echo.port | The port of echo server | integer | nil, server won't start |
| echo.enabled | Enable echo entry | bool | false |
| echo.description | Description of echo entry. | string | "" |

## Gzip options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| echo.interceptors.gzip.enabled | Enable gzip interceptor | boolean | false |
| echo.interceptors.gzip.level | Provide level of compression, options are noCompression, bestSpeed, bestCompression, defaultCompression, huffmanOnly. | string | defaultCompression |

## Quick start
### 1.Create boot.yaml
```yaml
---
echo:
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

### 2.Create main.go
```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
	"context"
	"fmt"
	"github.com/labstack/echo/v4"
	"github.com/rookie-ninja/rk-boot"
	"io"
	"net/http"
	"strings"
)

// @title RK Swagger for Echo
// @version 1.0
// @description This is a greeter service with rk-boot.

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
	boot.GetEchoEntry("greeter").Echo.POST("/v1/post", post)

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}

// PostResponse Response of Post.
type PostResponse struct {
	ReceivedMessage string
}

// post Handler.
func post(ctx echo.Context) error {
	buf := new(strings.Builder)
	io.Copy(buf, ctx.Request().Body)

	ctx.JSON(http.StatusOK, &PostResponse{
		ReceivedMessage: fmt.Sprintf("%s", buf.String()),
	})

	return nil
}
```

### 3.Validate
> Send request

```shell script
$ echo 'this is message' | gzip | curl --compressed --data-binary @- -H "Content-Encoding: gzip" -H "Accept-Encoding: gzip" localhost:8080/v1/post
{"ReceivedMessage":"this is message\n"}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)


