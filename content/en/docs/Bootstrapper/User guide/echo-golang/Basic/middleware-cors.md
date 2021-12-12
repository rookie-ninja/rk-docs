---
title: "Middleware cors"
linkTitle: "Middleware cors"
weight: 15
description: >
  Enable CORS interceptor/middleware for the server.
---

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-echo
```

## General options
> These are general options to start an echo server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| echo.name | The name of echo server | string | N/A |
| echo.port | The port of echo server | integer | nil, server won't start |
| echo.enabled | Enable echo entry | bool | false |
| echo.description | Description of echo entry. | string | "" |

## CORS options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| echo.interceptors.cors.enabled | Enable cors interceptor | boolean | false |
| echo.interceptors.cors.allowOrigins | Provide allowed origins with wildcard enabled. | []string | * |
| echo.interceptors.cors.allowMethods | Provide allowed methods returns as response header of OPTIONS request. | []string | All http methods |
| echo.interceptors.cors.allowHeaders | Provide allowed headers returns as response header of OPTIONS request. | []string | Headers from request |
| echo.interceptors.cors.allowCredentials | Returns as response header of OPTIONS request. | bool | false |
| echo.interceptors.cors.exposeHeaders | Provide exposed headers returns as response header of OPTIONS request. | []string | "" |
| echo.interceptors.cors.maxAge | Provide max age returns as response header of OPTIONS request. | int | 0 |

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
      cors:
        enabled: true                 # Optional, default: false
        # Accept all origins from localhost
        allowOrigins:
          - "http://localhost:*"      # Optional, default: *
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
	"github.com/rookie-ninja/rk-boot"
	_ "github.com/rookie-ninja/rk-echo/boot"
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

### 3.Create cors.html
```html
<!DOCTYPE html>
<html>
<body>

<h1>CORS Test</h1>

<p>Call http://localhost:8080/rk/v1/healthy</p>

<script type="text/javascript">
    window.onload = function() {
        var apiUrl = 'http://localhost:8080/rk/v1/healthy';
        fetch(apiUrl).then(response => response.json()).then(data => {
            document.getElementById("res").innerHTML = data["healthy"]
        }).catch(err => {
            document.getElementById("res").innerHTML = err
        });
    };
</script>

<h4>Response: </h4>
<p id="res"></p>

</body>
</html>
```

### 4.Directory hierarchy
```shell script
.
├── boot.yaml
├── cors.html
├── go.mod
├── go.sum
└── main.go

0 directories, 5 files
```

### 5.Validate
Open cors.html

![](/bootstrapper/user-guide/echo-golang/basic/cors-success.png)

### 6.With blocking CORS
Set echo.interceptors.cors.allowOrigins to http://localhost:8080 in order to block request.

```yaml
---
echo:
  - name: greeter                     # Required
    port: 8080                        # Required
    enabled: true                     # Required
    commonService:
      enabled: true                   # Optional, default: false
    interceptors:
      cors:
        enabled: true                 # Optional, default: false
        allowOrigins:
          - "http://localhost:8080"   # Optional, default: *
```

![](/bootstrapper/user-guide/echo-golang/basic/cors-fail.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)