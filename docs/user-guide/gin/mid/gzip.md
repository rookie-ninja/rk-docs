Enable gzip middleware

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

## gzip options
| options                     | description                        | type     | default |
|-----------------------------|--------------------------------------------------------------------------------------|----------|--------------------|
| gin.middleware.gzip.enabled | Enable gzip middleware                                                               | boolean  | false              |
| gin.middleware.gzip.ignore  | Ignore by path                                                                       | []string | []                 |
| gin.middleware.gzip.level   | options: noCompression, bestSpeed, bestCompression, defaultCompression, huffmanOnly. | string   | defaultCompression |

## Quick start
### 1.Create boot.yaml
```yaml
---
gin:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      gzip:
        enabled: true
        level: bestSpeed
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
  "io"
  "net/http"
  "strings"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  entry := rkgin.GetGinEntry("greeter")
  entry.Router.POST("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx *gin.Context) {
  buf := new(strings.Builder)
  io.Copy(buf, ctx.Request.Body)

  ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Received %s!", buf.String()),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.Validation
> Send request

```bash
$ echo 'this is message' | gzip | curl --compressed --data-binary @- -H "Content-Encoding: gzip" -H "Accept-Encoding: gzip" localhost:8080/v1/greeter
{"Message":"Received this is message\n!"}%
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)