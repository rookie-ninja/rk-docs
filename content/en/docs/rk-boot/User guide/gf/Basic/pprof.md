---
title: "PPROF"
linkTitle: "PPROF"
weight: 4
description: >
  Enable pprof UI。
---

## Install
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## Options
| options          | description          | type    | default |
|------------------|----------------------|---------|---------|
| gf.pprof.enabled | Enable pprof web UI  | boolean | false   |
| gf.pprof.path  | Path of pprof web UI | string  | pprof   |

## Quick start
### 1.Create boot.yaml

```yaml
---
gf:
  - name: greeter
    port: 8080
    enabled: true
    pprof:
      enabled: true
#      path: ""
```

### 2.Create main.go
```go
package main

import (
	"context"
	"fmt"
    "github.com/gogf/gf/v2/net/ghttp"
    "github.com/rookie-ninja/rk-boot/v2"
    "github.com/rookie-ninja/rk-gf/boot"
    "net/http"
)

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
    entry := rkgf.GetGfEntry("greeter")
    entry.Server.BindHandler("/v1/greeter", Greeter)

	// Bootstrap
	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *ghttp.Request) {
  ctx.Response.WriteHeader(http.StatusOK)
  ctx.Response.WriteJson(&GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.GetQuery("name").String()),
  })
}

type GreeterResponse struct {
	Message string
}
```

### 3.Validate
> **PPROF:** [http://localhost:8080/pprof](http://localhost:8080/pprof)

![](/rk-boot/user-guide/gin/basic/gin-pprof.png)

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)
