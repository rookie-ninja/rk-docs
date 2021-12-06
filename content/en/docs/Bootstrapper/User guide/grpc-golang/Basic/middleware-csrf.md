---
title: "Middleware CSRF"
linkTitle: "Middleware CSRF"
weight: 17
description: >
  Enable CSRF interceptor/middleware for the server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
```

## General options
> These are general options to start a gRPC server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.name | The name of grpc server | string | N/A |
| grpc.port | The port of grpc server | integer | nil, server won't start |
| grpc.enabled | Enable grpc entry | bool | false |
| grpc.description | Description of grpc entry. | string | "" |

## CSRF options
Interceptor for grpc-gateway.

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| grpc.interceptors.csrf.enabled | Enable csrf interceptor | boolean | false |
| grpc.interceptors.csrf.tokenLength | Provide the length of the generated token. | int | 32 |
| grpc.interceptors.csrf.tokenLookup | Provide csrf token lookup rules, please see code comments for details. | string | "header:X-CSRF-Token" |
| grpc.interceptors.csrf.cookieName | Provide name of the CSRF cookie. This cookie will store CSRF token. | string | _csrf |
| grpc.interceptors.csrf.cookieDomain | Domain of the CSRF cookie. | string | "" |
| grpc.interceptors.csrf.cookiePath | Path of the CSRF cookie. | string | "" |
| grpc.interceptors.csrf.cookieMaxAge | Provide max age (in seconds) of the CSRF cookie. | int | 86400 |
| grpc.interceptors.csrf.cookieHttpOnly | Indicates if CSRF cookie is HTTP only. | bool | false |
| grpc.interceptors.csrf.cookieSameSite | Indicates SameSite mode of the CSRF cookie. Options: lax, strict, none, default | string | default |
| grpc.interceptors.csrf.ignorePrefix | Ignoring path prefix. | []string | [] |

## Quick start
### 1.Create boot.yaml
```yaml
---
grpc:
  - name: greeter                     # Required
    port: 8080                        # Required
    enabled: true                     # Required
    interceptors:
      csrf:
        enabled: true                 # Optional, default: false
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
	"github.com/rookie-ninja/rk-boot"
	"net/http"
)

// Application entrance.
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Register handler in grpc-gateway
	entry := boot.GetGrpcEntry("greeter")
	entry.GwMux.HandlePath("GET", "/v1/greeter", func(w http.ResponseWriter, r *http.Request, pathParams map[string]string) {
		w.Write([]byte(fmt.Sprintf("Hello %s!", r.URL.Query().Get("name"))))
	})

	entry.GwMux.HandlePath("POST", "/v1/greeter", func(w http.ResponseWriter, r *http.Request, pathParams map[string]string) {
		w.Write([]byte(fmt.Sprintf("Hello %s!", r.URL.Query().Get("name"))))
	})

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```

### 3.Validate
- Send GET request, no csrf validation expected and a new cookie should be there.

```shell script
$ curl -X GET -vs localhost:8080/v1/greeter
  ...
  < HTTP/1.1 200 OK
  < Content-Type: application/json; charset=UTF-8
  < Set-Cookie: _csrf=WyOJLwzhfUGAMDHglkuIRucdpalxolWg; Expires=Mon, 06 Dec 2021 18:38:45 GMT
  < Vary: Cookie
  < Date: Sun, 05 Dec 2021 18:38:45 GMT
  < Content-Length: 22
  <
  {"Message":"Hello *!"}
```

- POST method with happy case

```shell script
$ curl -X POST -v --cookie "_csrf=my-test-csrf-token" -H "X-CSRF-Token:my-test-csrf-token" localhost:8080/v1/greeter
  ...
  > Cookie: _csrf=my-test-csrf-token
  > X-CSRF-Token:my-test-csrf-token
  >
  < HTTP/1.1 200 OK
  < Content-Type: application/json; charset=UTF-8
  < Set-Cookie: _csrf=my-test-csrf-token; Expires=Mon, 06 Dec 2021 18:40:13 GMT
  < Vary: Cookie
  < Date: Sun, 05 Dec 2021 18:40:13 GMT
  < Content-Length: 22
  <
  {"Message":"Hello *!"}
```

- POST method with invalid csrf

```shell script
$ curl -X POST -v -H "X-CSRF-Token:my-test-csrf-token" localhost:8080/v1/greeter
  ...
  > X-CSRF-Token:my-test-csrf-token
  >
  < HTTP/1.1 403 Forbidden
  < Content-Type: application/json; charset=UTF-8
  < Date: Sun, 05 Dec 2021 18:41:00 GMT
  < Content-Length: 92
  <
  {"error":{"code":403,"status":"Forbidden","message":"invalid csrf token","details":[null]}}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)