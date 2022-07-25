启动安全中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gf
```

## Secure 选项
| 名字                                          | 描述                                  | 类型       | 默认值             |
|---------------------------------------------|-------------------------------------|----------|-----------------|
| gf.middleware.secure.enabled               | 启动安全中间件                             | boolean  | false           |
| gf.middleware.secure.ignore                | 局部选项，忽略 API 路径                      | []string | []              |
| gf.middleware.secure.xssProtection         | X-XSS-Protection                    | string   | "1; mode=block" |
| gf.middleware.secure.contentTypeNosniff    | X-Content-Type-Options              | string   | nosniff         |
| gf.middleware.secure.xFrameOptions         | X-Frame-Options                     | string   | SAMEORIGIN      |
| gf.middleware.secure.hstsMaxAge            | Strict-Transport-Security           | int      | 0               |
| gf.middleware.secure.hstsExcludeSubdomains | HSTS SubDomains                     | bool     | false           |
| gf.middleware.secure.hstsPreloadEnabled    | 开启 HSTS 预加载。                        | bool     | false           |
| gf.middleware.secure.contentSecurityPolicy | Content-Security-Policy             | string   | ""              |
| gf.middleware.secure.cspReportOnly         | Content-Security-Policy-Report-Only | bool     | false           |
| gf.middleware.secure.referrerPolicy        | Referrer-Policy                     | string   | ""              |

## 快速开始
### 1.创建 boot.yaml
```yaml
---
gf:
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

### 2.创建 main.go
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

### 3.验证
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