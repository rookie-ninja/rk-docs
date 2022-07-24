Enable Prometheus middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

## Options
| options                     | description                        | type     | default |
|-----------------------------|------------------------------|----------|-------|
| gin.middleware.prom.enabled | Enable Prometheus middleware | boolean  | false |
| gin.middleware.prom.ignore  | Ignore by path               | []string | []    |

## Concept
Prometheus middleware will record bellow information

| Fields      | Type    | Description              |
|-------------|---------|--------------------------|
| elapsedNano | Summary | RPC elapsed time in nano |
| resCode     | Counter | Counter of response code |

Labels as bellow:

| Label      | Description                   |
|------------|-------------------------------|
| entryName  | Entry name                    |
| entryType  | Entry type                    |
| domain     | ENV value of DOMAIN, eg: prod |
| instance   | Hostname                      |
| restMethod | eg: GET                       |
| restPath   | eg: /rk/v1/alive              |
| resCode    | Response code, eg: 200        |

Example

```bash
rk_prom_elapsedNano{domain="*",entryName="greeter",entryType="GinEntry",instance="lark.local",resCode="200",restMethod="GET",restPath="/v1/greeter",quantile="0.5"} 88645
...
rk_prom_resCode{domain="*",entryName="greeter",entryType="GinEntry",instance="lark.local",resCode="200",restMethod="GET",restPath="/v1/greeter"} 1
```

## Quick start
### 1.Create boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    prom:
      enabled : true
    middleware:
      prom:
        enabled: true
#        ignore: [""]
```

### 2.Create main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/gin-gonic/gin"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-gin/v2/boot"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgin.GetGinEntry("greeter")
  entry.Router.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *gin.Context) {
  ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.Query("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.Validate
> Send request

```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev"
{"Message":"Hello rk-dev!"}
```

> Prometheus client:
>
> http://localhost:8080/metrics

![prom-inter](../../../img/user-guide/gin/basic/gin-prom-inter.png)

### _**Cheers**_
![](../../../img/user-guide/cheers.png)
