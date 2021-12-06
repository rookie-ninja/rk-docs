---
title: "Middleware secure"
linkTitle: "Middleware secure"
weight: 16
description: >
  Enable secure interceptor/middleware for the server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
```

## General options
> These are general options to start an echo server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| echo.name | The name of echo server | string | N/A |
| echo.port | The port of echo server | integer | nil, server won't start |
| echo.enabled | Enable echo entry | bool | false |
| echo.description | Description of echo entry. | string | "" |

## Secure options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| echo.interceptors.secure.enabled | Enable secure interceptor | boolean | false |
| echo.interceptors.secure.xssProtection | X-XSS-Protection header value. | string | "1; mode=block" |
| echo.interceptors.secure.contentTypeNosniff | X-Content-Type-Options header value. | string | nosniff |
| echo.interceptors.secure.xFrameOptions | X-Frame-Options header value. | string | SAMEORIGIN |
| echo.interceptors.secure.hstsMaxAge | Strict-Transport-Security header value. | int | 0 |
| echo.interceptors.secure.hstsExcludeSubdomains | Excluding subdomains of HSTS. | bool | false |
| echo.interceptors.secure.hstsPreloadEnabled | Enabling HSTS preload. | bool | false |
| echo.interceptors.secure.contentSecurityPolicy | Content-Security-Policy header value. | string | "" |
| echo.interceptors.secure.cspReportOnly | Content-Security-Policy-Report-Only header value. | bool | false |
| echo.interceptors.secure.referrerPolicy | Referrer-Policy header value. | string | "" |
| echo.interceptors.secure.ignorePrefix | Ignoring path prefix. | []string | [] |

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
      secure:
        enabled: true                 # Optional, default: false
```

### 2.Create main.go
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

### 3.Validate
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