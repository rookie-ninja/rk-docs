---
title: "JWT 中间件"
linkTitle: "JWT 中间件"
weight: 14
description: >
  启动 JWT 中间件。
---

## 安装
```shell script
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-echo
```

## 选项
| 名字                                     | 描述                                                   | 类型 | 默认值 |
|----------------------------------------|------------------------------------------------------| ------ | --- |
| echo.middleware.jwt.enabled             | 启动 JWT 中间件                                           | boolean | false |
| echo.middleware.jwt.ignore              | 局部选项，忽略 API 路径                                       | []string | []             |
| echo.middleware.jwt.skipVerify           | 忽略 JWT 验证                                            | boolean  | false                  |
| echo.middleware.jwt.signerEntry         | SignerEntry 名称                                       | string | "" |
| echo.middleware.jwt.symmetric.algorithm | 对称加密算法, 选项：HS256, HS384, HS512                       | string | "" |
| echo.middleware.jwt.symmetric.token     | 对称加密密钥                                               | string | "" |
| echo.middleware.jwt.symmetric.tokenPath | 对称加密密钥本地路径                                           | string | "" |
| echo.middleware.jwt.asymmetric.algorithm| 非对称加密算法, 选项：RS256, RS384, RS512, ES256, ES384, ES512 | string | "" |
| echo.middleware.jwt.tokenLookup         | 寻找 JWT Token 的格式，参考下面的例子                             | string | "header:Authorization" |
| echo.middleware.jwt.authScheme          | 提供 Auth Scheme                                       | string | Bearer |

**tokenLookup** 格式

```
// Optional. Default value "header:Authorization".
// Possible values:
// - "header:<name>"
// - "query:<name>"
// Multiply sources example:
// - "header: Authorization,cookie: myowncookie"
```

## 快速开始
### 1.创建 boot.yaml
```yaml
---
echo:
  - name: greeter
    port: 8080
    enabled: true
    middleware:
      jwt:
        enabled: true
        symmetric:
          algorithm: HS256
          token: "my-secret"
#          tokenPath: ""
#        signerEntry: ""
#        ignore: [ "" ]
#        skipVerify: false
#        asymmetric:
#          algorithm: ""
#          privateKey: ""
#          privateKeyPath: ""
#          publicKey: ""
#          publicKeyPath: ""
#        tokenLookup: "header:<name>"
#        authScheme: "Bearer"
```

### 2.创建 main.go
```go
package main

import (
  "context"
  "fmt"
  "github.com/labstack/echo/v4"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-echo/boot"
  "github.com/rookie-ninja/rk-echo/middleware/context"
  "net/http"
)

func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Register handler
  echoEntry := rkecho.GetEchoEntry("greeter")
  echoEntry.Echo.GET("/v1/greeter", Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

func Greeter(ctx echo.Context) error {
  // Get JWT token
  rkechoctx.GetJwtToken(ctx)
  
  return ctx.JSON(http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", ctx.QueryParam("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.验证
- 正确的 JWT Token

```shell script
$ curl "localhost:8080/v1/greeter?name=rk-dev" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.EpM5XBzTJZ4J8AfoJEcJrjth8pfH28LWdjLo90sYb9g"
{"Message":"Hello rk-dev!"}
```

- 非法 JWT Token
```shell script
$ curl localhost:8080/v1/greeter -H "Authorization: Bearer invalid-jwt-token"
{
    "error":{
        "code":401,
        "status":"Unauthorized",
        "message":"Invalid or expired jwt",
        "details":[]
    }
}
```

### _**Cheers**_
![](/rk-boot/user-guide/cheers.png)