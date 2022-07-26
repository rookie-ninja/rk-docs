Enable secure middleware.

## Install
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-zero
```

## Secure options
| options                     | description                        | type     | default |
|---------------------------------------------|------------------------------------|----------|-----------------|
| zero.middleware.secure.enabled               | Enable secure middleware           | boolean  | false           |
| zero.middleware.secure.ignore  | Ignore by path                     | []string | []    |
| zero.middleware.secure.xssProtection         | X-XSS-Protection                   | string   | "1; mode=block" |
| zero.middleware.secure.contentTypeNosniff    | X-Content-Type-Options             | string   | nosniff         |
| zero.middleware.secure.xFrameOptions         | X-Frame-Options                    | string   | SAMEORIGIN      |
| zero.middleware.secure.hstsMaxAge            | Strict-Transport-Security          | int      | 0               |
| zero.middleware.secure.hstsExcludeSubdomains | HSTS SubDomains                    | bool     | false           |
| zero.middleware.secure.hstsPreloadEnabled    | Enable HSTS preloading             | bool     | false           |
| zero.middleware.secure.contentSecurityPolicy | Content-Security-Policy            | string   | ""              |
| zero.middleware.secure.cspReportOnly         | Content-Security-Policy-Report-Only | bool     | false           |
| zero.middleware.secure.referrerPolicy        | Referrer-Policy                    | string   | ""              |

## Quick start
### 1.Create boot.yaml
```yaml
---
zero:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      secure:
        enabled: true
#        ignore: [""]
#        xssProtection: ""
#        contentTypeNosniff: ""
#        xFrameOptions: ""
#        hstsMaxAge: 0
#        hstsExcludeSubdomains: false
#        hstsPreloadEnabled: false
#        contentSecurityPolicy: ""
#        cspReportOnly: false
#        referrerPolicy: ""
```

### 2.Create main.go
```go
package main

import (
  "context"
  "encoding/json"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-zero/boot"
  "github.com/zeromicro/go-zero/rest"
  "net/http"
)

// @title Swagger Example API
// @version 1.0
// @description This is a sample rk-demo server.
// @termsOfService http://swagger.io/terms/

// @securityDefinitions.basic BasicAuth

// @contact.name API Support
// @contact.url http://www.swagger.io/support
// @contact.email support@swagger.io

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  zeroEntry := rkzero.GetZeroEntry("greeter")
  zeroEntry.Server.AddRoute(rest.Route{
    Method:  http.MethodGet,
    Path:    "/v1/greeter",
    Handler: Greeter,
  })

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter
// @Id 1
// @Tags Hello
// @version 1.0
// @Param name query string true "name"
// @produce application/json
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(writer http.ResponseWriter, request *http.Request) {
  writer.WriteHeader(http.StatusOK)
  resp := &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", request.URL.Query().Get("name")),
  }
  bytes, _ := json.Marshal(resp)
  writer.Write(bytes)
}

type GreeterResponse struct {
  Message string
}
```

### 3.Validate
```bash
$ curl -vs "localhost:8080/v1/greeter?name=rk-dev"
...
< Content-Type: application/json; charset=utf-8
< X-Content-Type-Options: nosniff
< X-Frame-Options: SAMEORIGIN
< X-Xss-Protection: 1; mode=block
< Date: Sat, 16 Apr 2022 12:35:05 GMT
< Content-Length: 27
...
```

### _**Cheers**_
![](../../../img/user-guide/cheers.png)