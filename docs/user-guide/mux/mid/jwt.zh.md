启动 JWT 中间件。

## 安装
```bash
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-mux
```

## 选项
| 名字                                     | 描述                             | 类型 | 默认值 |
|----------------------------------------|--------------------------------| ------ | --- |
| mux.middleware.jwt.enabled             | 启动 JWT 中间件                     | boolean | false |
| mux.middleware.jwt.ignore              | 局部选项，忽略 API 路径                 | []string | []             |
| mux.middleware.jwt.skipVerify           | 忽略 JWT 验证                                            | boolean  | false                  |
| mux.middleware.jwt.signerEntry         | SignerEntry 名称                 | string | "" |
| mux.middleware.jwt.symmetric.algorithm | 对称加密算法, 选项：HS256, HS384, HS512                         | string | "" |
| mux.middleware.jwt.symmetric.token     | 对称加密密钥                         | string | "" |
| mux.middleware.jwt.symmetric.tokenPath | 对称加密密钥本地路径                     | string | "" |
| mux.middleware.jwt.asymmetric.algorithm| 非对称加密算法, 选项：RS256, RS384, RS512, ES256, ES384, ES512                        | string | "" |
| mux.middleware.jwt.tokenLookup         | 寻找 JWT Token 的格式，参考下面的例子 | string | "header:Authorization" |
| mux.middleware.jwt.authScheme          | 提供 Auth Scheme                 | string | Bearer |

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
mux:
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
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.
package main

import (
  "context"
  _ "embed"
  "fmt"
  "github.com/rookie-ninja/rk-boot/v2"
  "github.com/rookie-ninja/rk-mux/boot"
  "github.com/rookie-ninja/rk-mux/middleware"
  "net/http"
)

// @title RK Swagger for Mux
// @version 1.0
// @description This is a greeter service with rk-boot.
func main() {
  // Create a new boot instance.
  boot := rkboot.NewBoot()

  // Get MuxEntry
  muxEntry := rkmux.GetMuxEntry("greeter")
  // Use *mux.Router adding handler.
  muxEntry.Router.NewRoute().Path("/v1/greeter").HandlerFunc(Greeter)

  // Bootstrap
  boot.Bootstrap(context.TODO())

  boot.WaitForShutdownSig(context.TODO())
}

// Greeter handler
// @Summary Greeter service
// @Id 1
// @version 1.0
// @produce application/json
// @Param name query string true "Input name"
// @Success 200 {object} GreeterResponse
// @Router /v1/greeter [get]
func Greeter(writer http.ResponseWriter, req *http.Request) {
  rkmuxmid.WriteJson(writer, http.StatusOK, &GreeterResponse{
    Message: fmt.Sprintf("Hello %s!", req.URL.Query().Get("name")),
  })
}

type GreeterResponse struct {
  Message string
}
```

### 3.验证
- 正确的 JWT Token

```bash
$ curl "localhost:8080/v1/greeter?name=rk-dev" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.EpM5XBzTJZ4J8AfoJEcJrjth8pfH28LWdjLo90sYb9g"
{"Message":"Hello rk-dev!"}
```

- 非法 JWT Token
```bash
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
![](../../../img/user-guide/cheers.png)