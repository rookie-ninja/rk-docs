---
title: "Middleware jwt"
linkTitle: "Middleware jwt"
weight: 14
description: >
  Enable jwt interceptor/middleware for the server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
```

## General options
> These are general options to start an GoFrame server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.name | The name of GoFrame server | string | N/A |
| gf.port | The port of GoFrame server | integer | nil, server won't start |
| gf.enabled | Enable GoFrame entry | bool | false |
| gf.description | Description of GoFrame entry. | string | "" |

## JWT options
In order to make swagger UI and RK tv work under JWT without JWT token, we need to ignore prefixes of paths as bellow.

```yaml
jwt:
  ...
  ignorePrefix:
   - "/rk/v1/tv"
   - "/sw"
   - "/rk/v1/assets"
```

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.jwt.enabled | Enable JWT interceptor | boolean | false |
| gf.interceptors.jwt.signingKey | Required, Provide signing key. | string | "" |
| gf.interceptors.jwt.ignorePrefix | Provide ignoring path prefix. | []string | [] |
| gf.interceptors.jwt.signingKeys | Provide signing keys as scheme of <key>:<value>. | []string | [] |
| gf.interceptors.jwt.signingAlgo | Provide signing algorithm. | string | HS256 |
| gf.interceptors.jwt.tokenLookup | Provide token lookup scheme, please see bellow description. | string | "header:Authorization" |
| gf.interceptors.jwt.authScheme | Provide auth scheme. | string | Bearer |

The supported scheme of **tokenLookup** 

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

## Quick start
### 1.Create boot.yaml
```yaml
---
gf:
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
- with valid jwt token

```shell script
$ curl localhost:8080/rk/v1/healthy -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.EpM5XBzTJZ4J8AfoJEcJrjth8pfH28LWdjLo90sYb9g"
{"healthy":true}
```

- with invalid jwt token
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