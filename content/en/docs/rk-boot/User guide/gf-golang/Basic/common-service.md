---
title: "Common Service"
linkTitle: "Common Service"
weight: 3
description: >
  Enable common service for server.
---

## Overview
Common service contains builtin API commonly used by user.

| Path | Description |
| ---- | ---- |
| /rk/v1/apis | List APIs in current EchoEntry. |
| /rk/v1/certs | List CertEntry. |
| /rk/v1/configs | List ConfigEntry. |
| /rk/v1/deps | List dependencies related application, entire contents of go.mod file would be returned. |
| /rk/v1/entries | List all Entries. |
| /rk/v1/gc | Trigger GC |
| /rk/v1/healthy | Get application healthy status. |
| /rk/v1/info | Get application and process info. |
| /rk/v1/license | Get license related application, entire contents of LICENSE file would be returned. |
| /rk/v1/logs | List logger related entries. |
| /rk/v1/git | Get git information. |
| /rk/v1/readme | Get contents of README file. |
| /rk/v1/req | List prometheus metrics of requests. |
| /rk/v1/sys | Get OS stat. |
| /rk/v1/tv | Get HTML page of /tv. |

## Installation
```shell script
go get github.com/rookie-ninja/rk-boot
go get github.com/rookie-ninja/rk-gf
```

## General options
> These are general options to start an GoFrame server with rk-boot

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.name | The name of GoFrame server | string | N/A |
| gf.port | The port of GoFrame server | integer | nil, server won't start |
| gf.enabled | Enable GoFrame entry | bool | false |
| gf.description | Description of GoFrame entry. | string | "" |

## CommonService options
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.commonService.enabled | Enable embedded common service | boolean | false |

## Quick start
### 1.Create boot.yaml
```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true     # Enable common service
```

### 2.Create main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
    _ "github.com/rookie-ninja/rk-gf/boot"
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
$ curl -X GET localhost:8080/rk/v1/healthy
{"healthy":true}
```

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

### 4.Enable swagger UI
> In order to visualise common service in swagger UI, we need to enable swagger in boot.yaml as bellow.

```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    commonService:
      enabled: true     # Enable common service
    sw:
      enabled: true     # Enable swagger UI
```

> Validate
>
> [http://localhost:8080/sw](http://localhost:8080/sw)

![sw-common](/bootstrapper/getting-started/gf-golang/gf-sw.png)

### _**Cheers**_
![](/bootstrapper/user-guide/cheers.png)

